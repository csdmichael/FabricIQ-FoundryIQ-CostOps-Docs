# ACR by Service — Monthly Azure Resource Consumption

[Back to main README](../../README.md#acr-by-service)

This is an illustrative monthly estimate in USD for the CostOps platform described in the [setup guide](../setup/README.md), not the enterprise workloads it monitors. ACR means **Azure Resource Consumption**, not Azure Container Registry or contractual Azure Consumed Revenue/MACC eligibility. Rates are planning assumptions, not current retail quotes; validate regional pricing and agreement eligibility before budgeting.

## Monthly estimate

Each environment has its own listed resources; Power BI licenses are shared and excluded from ACR. Workload estimates assume 100,000 / 250,000 / 1,000,000 events per day in Dev / QA / Prod, with 1 / 1 / 3 months of retention. Fixed resources are billed even when workloads are lower or Fabric is paused.

| Service | SKU / capacity basis (Dev / QA / Prod) | Dev | QA | Prod | Total |
| --- | --- | ---: | ---: | ---: | ---: |
| Microsoft Fabric capacity | F8, 160 h × $1.44/h / F8, 320 h × $1.44/h / F64, 730 h × $11.52/h | $230.40 | $460.80 | $8,409.60 | $9,100.80 |
| OneLake storage | 36 / 90 / 1,080 GB × $0.023/GB-month | $0.83 | $2.07 | $24.84 | $27.74 |
| API Management | 1 Standard v2 unit per environment | $700.00 | $700.00 | $700.00 | $2,100.00 |
| App Service — UI and API plans | 2 Linux P1v3 plans, 1 instance each | $300.00 | $300.00 | $300.00 | $900.00 |
| Functions — pricing refresh compute | Shares the API App Service plan | $0.00 | $0.00 | $0.00 | $0.00 |
| Function backing Storage account | 10 GB plus 10k / 25k / 100k storage operations | $0.25 | $0.33 | $0.70 | $1.28 |
| Cosmos DB — throughput and storage | 400 RU/s; 12 / 30 / 360 GB | $26.36 | $30.86 | $113.36 | $170.58 |
| Blob Storage — raw telemetry/snapshots | 6 / 15 / 180 GB, Hot LRS | $0.14 | $0.34 | $3.77 | $4.25 |
| Monitor / Log Analytics / Application Insights | 9 / 22.5 / 90 GB billable ingestion | $24.84 | $62.10 | $248.40 | $335.34 |
| Foundry / Azure OpenAI — platform chat | 1k / 2.5k / 10k conversations; 3 calls each | $36.00 | $90.00 | $360.00 | $486.00 |
| Key Vault | 10k / 25k / 100k secret operations | $0.03 | $0.08 | $0.30 | $0.41 |
| Private Link / private endpoints | 8 endpoints per environment; 10 / 25 / 100 GB processed | $58.50 | $58.65 | $59.40 | $176.55 |
| Private DNS | 8 zones per environment; 0.1 / 0.25 / 1M queries | $4.04 | $4.10 | $4.40 | $12.54 |
| Bandwidth — chargeable outbound/cross-region | 10 / 25 / 100 GB after applicable allowances | $0.87 | $2.18 | $8.70 | $11.75 |
| Entra ID / managed identities / base VNet | No premium features assumed | $0.00 | $0.00 | $0.00 | $0.00 |
| **Total ACR** | **Sum of Azure/Fabric resource costs** | **$1,382.26** | **$1,711.51** | **$10,233.47** | **$13,327.24** |
| Power BI Pro licenses — shared, outside ACR | 25 distinct users × $14/month, counted once | — | — | — | $350.00 |
| **Platform total including licenses** | **ACR plus shared licenses** |  |  |  | **$13,677.24** |

Fabric uses the assumed F8 rate of $1.44 per capacity-hour and a linearly scaled F64 rate of $11.52 per capacity-hour. Dev and QA hours are estimated at 160 and 320 per month; Prod is estimated at 730 hours. Actual billed cost depends on regional rates, capacity usage, and operating schedule.

The estimate excludes taxes, support, labor, disaster recovery, premium identity/governance/security, Container Registry, extra retention/backups, paid agent tools/hosted runtimes, AI Search/Foundry IQ retrieval, and monitored enterprise workloads. Shared resources should be charged once and allocated, not duplicated. Benchmark capacity and service usage, then reconcile against actual billing.

For current estimates, use the [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/) and validate meters with [Cost Management exports](https://learn.microsoft.com/azure/cost-management-billing/costs/tutorial-export-acm-data), [Fabric pricing](https://azure.microsoft.com/pricing/details/microsoft-fabric/), and the relevant [Azure service pricing pages](https://azure.microsoft.com/pricing/).
