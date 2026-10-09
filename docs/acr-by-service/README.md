# ACR by Service — Monthly Azure Resource Consumption

[Back to main README](../../README.md#acr-by-service)

## Summary

**Total estimated ACR across Dev, Test, and Prod: $5,565.64/month USD.**

| Monthly summary | Dev | Test | Prod | Total |
| --- | ---: | ---: | ---: | ---: |
| **Azure/Fabric resource consumption (ACR)** | **$1,209.46** | **$1,481.11** | **$2,875.07** | **$5,565.64** |
| Power BI Pro licenses — shared across environments, outside ACR | — | — | — | $350.00 |
| **Platform total including licenses** | | | | **$5,915.64** |

ACR here means **Azure Resource Consumption**, not Container Registry or contractual Azure Consumed Revenue/MACC eligibility. All rates are **illustrative planning assumptions, not current retail quotes**; validate regional pricing and agreement eligibility before budgeting.

This estimates the CostOps platform described in the [setup guide](../setup/README.md), not the enterprise AI workloads it monitors or the larger [business-case scenario](../business-case/business-case.md#4-value-model-micron-scale-enterprise).

## Workload and deployment assumptions

Each environment has **its own resources**, including APIM, hosting, storage, and private networking. Only Power BI user licenses are shared. Fixed costs are not reduced just because Dev/Test traffic is lower.

| Factor | Dev | Test | Prod |
| --- | --- | --- | --- |
| Variable workload factor (`s`) | 0.10 | 0.25 | 1.00 |
| Events/day; 30-day workload month, 2 KB/event | 100,000 | 250,000 | 1,000,000 |
| Raw/normalized retention (`r`, months) | 1 | 1 | 3 |
| Fabric SKU; active hours/month | F2; 160 | F4; 320 | F8; 730 |
| Assumed Fabric capacity-hour rate | $0.36 | $0.72 | $1.44 |
| Platform chat conversations/month | 1,000 | 2,500 | 10,000 |
| Billable log ingestion GB/month | 9 | 22.5 | 90 |
| Chargeable Private Link / outbound GB/month, each | 10 | 25 | 100 |

- **Always-on resources in each environment:** one APIM Standard v2 unit, two Linux P1v3 App Service plans (one instance each), 400 provisioned Cosmos RU/s, eight private endpoints and eight private DNS zones. These use a full-month allowance or 730 billed hours, even when Fabric is paused.
- **Storage:** steady-state raw GB = `60 × s × r`; Cosmos GB = raw × 2; OneLake GB = raw × 6 (three medallion layers × 2 overhead). Function storage is 10 GB per environment.
- **Variable usage:** Prod has 30,000 Blob writes, 60,000 Blob reads, 100,000 Function storage operations, 100,000 Key Vault operations, and 1M DNS queries/month; Dev/Test multiply those counts by `s`.
- **Chat:** three model calls/conversation, 4,000 input and 800 output tokens/call, 25% billed input-cache reads. Token rates below are hypothetical, not quotes for a named model.
- **Availability:** Dev/Test Fabric is unavailable outside its active hours. Storage and provisioned resources remain billable. Single-region resources, no DR, no reservations or discounts; sizes are hypotheses requiring load validation.

## Monthly estimate by service

USD/month, rounded half-up to cents per service/environment. Totals sum the rounded cells.

| Azure service | Dev | Test | Prod | Total ACR |
| --- | ---: | ---: | ---: | ---: |
| Microsoft Fabric capacity | $57.60 | $230.40 | $1,051.20 | $1,339.20 |
| OneLake storage | $0.83 | $2.07 | $24.84 | $27.74 |
| API Management | $700.00 | $700.00 | $700.00 | $2,100.00 |
| App Service — UI and API plans | $300.00 | $300.00 | $300.00 | $900.00 |
| Functions — pricing refresh compute, included in API plan | $0.00 | $0.00 | $0.00 | $0.00 |
| Function backing Storage account | $0.25 | $0.33 | $0.70 | $1.28 |
| Cosmos DB — throughput and storage | $26.36 | $30.86 | $113.36 | $170.58 |
| Blob Storage — raw telemetry/snapshots | $0.14 | $0.34 | $3.77 | $4.25 |
| Monitor / Log Analytics / Application Insights | $24.84 | $62.10 | $248.40 | $335.34 |
| Foundry / Azure OpenAI — platform chat | $36.00 | $90.00 | $360.00 | $486.00 |
| Key Vault | $0.03 | $0.08 | $0.30 | $0.41 |
| Private Link / private endpoints | $58.50 | $58.65 | $59.40 | $176.55 |
| Private DNS | $4.04 | $4.10 | $4.40 | $12.54 |
| Bandwidth — chargeable outbound/cross-region | $0.87 | $2.18 | $8.70 | $11.75 |
| Entra ID / managed identities / base VNet — no premium features | $0.00 | $0.00 | $0.00 | $0.00 |
| **Total ACR** | **$1,209.46** | **$1,481.11** | **$2,875.07** | **$5,565.64** |

## Calculation formulas

Use `s` and `r` from the environment assumptions. GB is decimal; storage uses average billed GB at steady state, not first-month end-of-month storage. `M` means one million.

| Service | Formula per environment using illustrative USD rates |
| --- | --- |
| Fabric | `active hours × SKU hourly rate` |
| OneLake | `(60 × s × r × 6) GB × $0.023/GB-month` |
| APIM | `1 Standard v2 unit × $700/month` |
| App Service / Functions compute | `2 P1v3 plans × 1 instance × $150/month`; Function shares API plan |
| Function storage | `10 GB × $0.02 + (100,000 × s / 10,000) operations × $0.05` (blended operation allowance) |
| Cosmos DB | `(400 RU/s / 100) × 730 hours × $0.008 + (60 × s × r × 2) GB × $0.25` |
| Blob Storage, Hot LRS | `(60 × s × r) GB × $0.02 + (30,000 × s / 10,000) writes × $0.05 + (60,000 × s / 10,000) reads × $0.004` |
| Monitor | `90 × s GB ingested × $2.76/GB`; 31-day included Analytics retention |
| Foundry chat | `s × (90M uncached input × $1.25/M + 30M cached input × $0.25/M + 24M output × $10/M)` |
| Key Vault | `(100,000 × s / 10,000) secret operations × $0.03` |
| Private Link | `8 endpoints × 730 hours × $0.01 + 100 × s GB processed × $0.01` |
| Private DNS | `8 zones × $0.50 + 1 × s M queries × $0.40/M` |
| Bandwidth | `100 × s chargeable GB × $0.087/GB`, after applicable allowances |
| Power BI, outside ACR | `25 distinct users across all environments × $14/month = $350/month`, counted once |

For example, Prod chat has `10,000 × 3 = 30,000` calls: `30,000 × 4,000 = 120M` input tokens (90M uncached + 30M cached) and `30,000 × 800 = 24M` output tokens, costing `$360`. Dev/Test scale this usage, **not** fixed infrastructure charges.

**Total ACR = Dev + Test + Prod = $5,565.64/month.** Platform total = ACR + shared licenses = **$5,915.64/month**. An optional 15% reserve on that platform total is **$887.35**, giving a **$6,802.99/month** planning budget; the reserve is not ACR.

## Scope and important checks

- **No double counting:** Dataflows, pipelines, Lakehouse, 12 PySpark ML use cases, SQL/BI queries, Fabric Data Agent AI/query work, and OneLake operations share each environment's Fabric capacity. Optional Fabric IQ ontology must fit that budget; verify preview billing. Function compute shares the API plan, but its storage is separate.
- **Validate sizing and networking:** benchmark Fabric CU usage, Cosmos RU peaks/indexes, and App Service concurrency. Confirm APIM Standard v2 private endpoints/outbound VNet integration and regional availability. Dev/Test workloads must fit their reduced Fabric operating windows; pausing may settle outstanding smoothed usage.
- **Licensing:** 25 distinct Power BI users are assumed across all environments, not 25 per environment. User-owns-data embedding below F64 generally requires Pro/qualifying PPU for viewers; authors need appropriate licenses. Existing eligible licenses can reduce incremental cost.
- **Excluded:** monitored enterprise Azure/non-Azure model spend, taxes, support, labor, DR, extra backups/retention, paid agent tools/hosted runtimes, AI Search/Foundry IQ retrieval, Container Registry, premium identity/governance/security, and paid gateways/firewalls. Add meters if deployed; excluded does not mean free. Count workspace-based Application Insights logs once and include agent retries/history in actual token usage.

## Pricing references and recalibration

Official sources for replacing the **illustrative** rates and validating meters:

| Service | Pricing / billing source |
| --- | --- |
| Fabric capacity and OneLake | [Microsoft Fabric pricing](https://azure.microsoft.com/pricing/details/microsoft-fabric/), [OneLake billing](https://learn.microsoft.com/fabric/onelake/onelake-capacity-consumption), and [Data Agent consumption](https://learn.microsoft.com/fabric/fundamentals/data-agent-consumption) |
| Power BI licensing | [Power BI pricing](https://www.microsoft.com/power-platform/products/power-bi/pricing) and [Fabric licenses](https://learn.microsoft.com/fabric/enterprise/licenses) |
| API Management | [APIM pricing](https://azure.microsoft.com/pricing/details/api-management/) and [virtual network concepts](https://learn.microsoft.com/azure/api-management/virtual-network-concepts) |
| App Service / Functions | [App Service Linux pricing](https://azure.microsoft.com/pricing/details/app-service/linux/), [Functions pricing](https://azure.microsoft.com/pricing/details/functions/), and [dedicated hosting](https://learn.microsoft.com/azure/azure-functions/dedicated-plan) |
| Cosmos DB | [Cosmos DB pricing](https://azure.microsoft.com/pricing/details/cosmos-db/) |
| Blob and Function storage | [Blob Storage pricing](https://azure.microsoft.com/pricing/details/storage/blobs/) |
| Monitoring | [Azure Monitor pricing](https://azure.microsoft.com/pricing/details/monitor/) |
| Foundry / Azure OpenAI | [Foundry Agent Service pricing](https://azure.microsoft.com/pricing/details/foundry-agent-service/) and [Azure OpenAI pricing](https://azure.microsoft.com/pricing/details/cognitive-services/openai-service/) |
| Secrets and identity | [Key Vault pricing](https://azure.microsoft.com/pricing/details/key-vault/) and [Microsoft Entra pricing](https://www.microsoft.com/security/business/microsoft-entra-pricing) |
| Networking | [Private Link pricing](https://azure.microsoft.com/pricing/details/private-link/), [DNS pricing](https://azure.microsoft.com/pricing/details/dns/), and [bandwidth pricing](https://azure.microsoft.com/pricing/details/bandwidth/) |
| Regional quotes and billed actuals | [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/), [Azure Retail Prices API](https://learn.microsoft.com/rest/api/cost-management/retail-prices/azure-retail-prices), and [Cost Management exports](https://learn.microsoft.com/azure/cost-management-billing/costs/tutorial-export-acm-data) |

Replace assumed rates with dated regional SKU/model quotes, measure actual usage, and recalculate each environment. Tag resources by environment and reconcile monthly against Azure/Fabric billing and license invoices. If resources are shared instead of independently provisioned, charge them once and allocate their cost rather than duplicating it.
