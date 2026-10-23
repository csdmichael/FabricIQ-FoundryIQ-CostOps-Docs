# Multi-Cloud AI Tokenomics & FinOps — Quick Setup

Deploy the whole platform with **one script**: `scripts/deploy-all.ps1` in the
[application repository](https://github.com/csdmichael/FabricIQ-FoundryIQ-CostOps). It deploys every
component in dependency order, creates the Fabric capacity and App Service plans when you say they
do not exist yet, and records the IDs it creates in `config/deployment.json` for you.

For the detailed, resource-by-resource guide, see the [full setup guide](../README.md).

## Table of Contents

- [1. Before You Start](#1-before-you-start)
- [2. Edit the Configuration](#2-edit-the-configuration)
- [3. Run the Script](#3-run-the-script)
- [4. What the Script Deploys](#4-what-the-script-deploys)
- [5. Manual Steps That Remain](#5-manual-steps-that-remain)
- [6. Verify](#6-verify)
- [7. Troubleshooting](#7-troubleshooting)

## 1. Before You Start

| Need | Details |
| --- | --- |
| Tools | PowerShell 7, Python 3.13, Node.js 22 LTS, Git with Git LFS, Azure CLI (Bicep installs automatically). |
| Sign-in | `az login --tenant <tenant-id>` and `az account set --subscription <subscription-id>`. |
| Permissions | Contributor on the resource group; Fabric admin rights to create a capacity and workspace; Entra rights to register applications on the first run. |
| Already present | An **API Management** service with a system-assigned identity, its VNet and subnets, a **Cosmos DB** account, a **storage account** for raw telemetry, and a **Log Analytics** workspace. The script checks these and stops with the missing name. |
| Optional | A Foundry resource (only if you enable `deployFoundry`), and a jump VM named `caldova-jump` for the synthetic seed (`seed-tokenomics-data.ps1 -RunnerVmName`). |
| Source | A clone of the application repository with the video assets pulled: `git lfs pull`. |

The script can create the resource group, the App Service plans, and the Fabric capacity. Everything else
in the table above must already exist.

## 2. Edit the Configuration

Open `config/deployment.json` and replace the sample values in these blocks with your own. You do **not**
need to look up Fabric IDs, the Fabric capacity GUID, or the Resource API client ID; the script records
them after it creates or identifies those resources.

| Block | Values to set |
| --- | --- |
| `azure` | `tenantId`, `subscriptionId`, `resourceGroup`, `location` |
| `apim` | `tenantId`, `subscriptionId`, `resourceGroup`, `serviceName`, `gatewayUrl`, `location`, `publisherEmail`, `publisherName` |
| `identity` | `resourceTenantId`, `callerTenantId`, `allowedUserPrincipalName`, `allowedUserObjectIds` (the Entra object ID of each user allowed to sign in) |
| `network` | `apimVnetResourceId`, `apimSubnetName`, `apimSubnetPrefix`, `apimPrivateEndpointSubnetResourceId` |
| `fabric` | `capacityName` (new or existing), `capacityRegion`, `workspaceName`, `lakehouseName`, `dataAgentName` |
| `tokenomicsPlatform` | `cosmos.accountName`, `rawStorage.accountName`, and the matching `fabric` names |
| `logAnalytics` | `workspaceResourceId` |
| `ui` | `appName`, `existingPlanName`, `productionUrl`, `allowedOrigin`, `redirectUris` |
| `dashboardApi` | `appName`, `existingPlanName`, `planSku`, `publicUrl`, `integrationSubnetResourceId` |
| `pricingFunction` | `appName`, `existingPlanName`, `existingPlanResourceGroup`, `storageAccountName`, `integrationSubnetResourceId` |
| `tags` | Cost center and owner tags |

The script records these values automatically, and only when it creates the resource or finds a different ID:

- `fabric.capacityId` and `tokenomicsPlatform.fabric.capacityId`: the Fabric capacity GUID.
- `fabric.workspaceId`, `lakehouseId`, `sqlEndpointId`, and `sqlEndpointHost`: the Fabric workspace values.
- `dashboardApi.resourceApiClientId`: the Entra application the API uses as its token audience.
- `powerBi` report, semantic model, and dashboard IDs.

Review the diff in `config/deployment.json` after the run and commit it to your own repository.

## 3. Run the Script

Choose the command that matches your environment. Each `*AppServicePlanExists` and `FabricCapacityExists`
parameter takes `$true` or `$false`:

- `$true` means the resource **must already exist**. The script stops if it is missing, so a typo cannot
  create billable resources by accident.
- `$false` means **create it if it is missing**, and reuse it if it already exists.

**New environment** (create the Fabric capacity and all three App Service plans):

```powershell
pwsh scripts/deploy-all.ps1 `
  -FabricCapacityExists:$false `
  -UiAppServicePlanExists:$false `
  -ApiAppServicePlanExists:$false `
  -PricingFunctionAppServicePlanExists:$false
```

**Existing environment** (the capacity and plans already exist; these are the defaults):

```powershell
pwsh scripts/deploy-all.ps1
```

**Mixed environment** (for example, your Fabric capacity is new but the plans already exist):

```powershell
pwsh scripts/deploy-all.ps1 -FabricCapacityExists:$false
```

| Parameter | Default | Set to `$false` when | Applies to |
| --- | --- | --- | --- |
| `-FabricCapacityExists` | `$true` | The Fabric capacity does not exist yet. It is created with `-FabricCapacitySku` (default `F2`; billing starts when it is active). | Fabric capacity, workspace, ontology |
| `-UiAppServicePlanExists` | `$true` | The UI App Service plan does not exist yet. It is created as Windows `B1`. | UI |
| `-ApiAppServicePlanExists` | `$true` | The API App Service plan does not exist yet. It is created as Linux with `dashboardApi.planSku`. | API |
| `-PricingFunctionAppServicePlanExists` | `$true` | The pricing function's plan does not exist yet. It is created as Linux with `dashboardApi.planSku`. | Pricing function |

Other options:

- `-PlanOnly` prints the ordered plan and each component's decision without changing Azure.
- `-SkipComponent seed,sync,powerbi` skips named components (names are listed in section 4).
- `-RepositoryCommit <sha>` seeds from a specific commit. The commit must already be on `origin/main`.
- The run stops at the first failed component and writes `.generated/deployment/deploy-all-summary.json`.
  Fix the cause, then run the command again. Add `-SkipComponent` for the components the summary shows as `succeeded`.

The script refuses to run if these are enabled, because this repository no longer deploys them:
`deployNetworking`, `deployBroker`, `createCopilotStudioAgents`, or `deployWorkspacePrivateLink`.

## 4. What the Script Deploys

Components run in this order. Each one is a script in `scripts/`.

| # | Component (`-SkipComponent` name) | What it does | Script |
| --- | --- | --- | --- |
| 1 | `preflight` | Checks tools, sign-in, the resource group (creates it if missing), and the prerequisites in section 1. | `deploy-all.ps1` |
| 2 | `fabric-capacity` | Creates the Fabric capacity if `-FabricCapacityExists:$false`, resumes it if paused, and records its GUID. | `provision-fabric-capacity.ps1` |
| 3 | `identity` | Creates the Entra identity metadata when `.generated/identity.json` is missing, and records the API client ID. | `provision-identity.ps1` |
| 4 | `fabric-workspace` | Creates the workspace, Lakehouse, and dataflows on the capacity, and records the Fabric IDs. | `provision-tokenomics-fabric.ps1` |
| 5 | `fabric-ontology` | Deploys the Fabric IQ ontology. | `deploy-fabric-ontology.py` |
| 6 | `api` | Creates the plan if needed, creates the API App Service (Linux, Python 3.12), applies settings, deploys the code, and checks `/live`. | `deploy-api.ps1` |
| 7 | `pricing-function` | Creates the plan if needed, deploys the pricing Azure Function and its code. | `deploy-pricing-function.ps1` |
| 8 | `data-platform` | Creates the Cosmos DB containers, Azure Maps account, and raw telemetry container, and points the API at them. | `deploy-data-platform.ps1` |
| 9 | `apim` | Deploys the APIM APIs, products, and policies. | `deploy-apim.ps1` |
| 10 | `ui` | Creates the plan if needed, deploys the UI App Service, publishes the package, and checks `/health` and the showcase videos. | `deploy-ui.ps1` |
| 11 | `seed` | Seeds synthetic multi-cloud telemetry through the runner VM. | `seed-tokenomics-data.ps1` |
| 12 | `sync` | Materializes the Delta tables in OneLake. | `sync-tokenomics-to-fabric.ps1` |
| 13 | `powerbi` | Publishes the semantic model, reports, and dashboards, and redeploys the API if the IDs changed. | `publish-tokenomics-powerbi.ps1` |
| 14 | `foundry` | Provisions the Foundry project and analyst agent. Off by default (`deployFoundry`). | `provision-tokenomics-foundry.py` |

## 5. Manual Steps That Remain

These steps need a portal or an in-network host, so the script cannot finish them:

1. **Create the Data Agent.** Create the `Tokenomics FinOps Analyst` Data Agent over the Lakehouse and ML tables in the Fabric portal, then set its ID in `fabric.dataAgentId`.
2. **Bind dataflow connections.** Complete the actions listed in `portalActionsRequired` in `.generated/tokenomics/fabric.json` (for example, the Cosmos DB and Log Analytics connections).
3. **Pin dashboard tiles** if `.generated/tokenomics/powerbi/published.json` reports a `portalActionRequired` for a dashboard.
4. **Foundry** (only if `deployFoundry` is `true`): run the step from a host that can reach the private Foundry endpoint, after `pip install -r scripts/requirements-foundry.txt`.

## 6. Verify

| Check | How | Expected |
| --- | --- | --- |
| Run summary | Open `.generated/deployment/deploy-all-summary.json` | Every component `succeeded` |
| UI host | `GET https://<ui.appName>.azurewebsites.net/health` | `{ "status": "ok" }` |
| API liveness | `GET https://<dashboardApi.appName>.azurewebsites.net/live` | `{ "status": "ok" }` |
| Config diff | `git diff config/deployment.json` | Only the IDs listed in section 2 changed |

Then sign in to the UI with an account listed in `identity.allowedUserObjectIds`, open the Overview screen, and check the showcase **Try it** tab.

## 7. Troubleshooting

| Message | Fix |
| --- | --- |
| `Fabric capacity '<name>' was not found` | Pass `-FabricCapacityExists:$false` to create it, or correct `fabric.capacityName`. |
| `App Service plan '<name>' is Linux; Windows was requested` (or the reverse) | Plans cannot change operating system. Point the config at a plan with the matching OS, or use a new plan name. |
| `Prerequisite '<name>' was not found` | Create the resource, or correct its name or ID in `config/deployment.json`. |
| `Identity metadata was generated for an earlier config/deployment.json` | A warning, not an error. The script records IDs after identity is created, so the stored fingerprint differs. Re-run `provision-identity.ps1` only if identity settings changed. |
| `RepositoryCommit must already be reachable from origin/main` | Push your commit, or pass `-RepositoryCommit` with a commit that is on `origin/main`. |
| `Showcase video verification failed` or a Git LFS pointer | Run `git lfs pull`, then run the UI step again. |
| `API liveness check failed` | Open the API App Service log stream (`az webapp log tail --name <dashboardApi.appName> --resource-group <rg>`) and check the startup script output. |

If something else fails, the run summary shows the failing component and its message. Fix the cause and run the same command again.
