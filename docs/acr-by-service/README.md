# ACR by Service — Monthly Azure Resource Consumption

[Back to main README](../../README.md#acr-by-service)

## Summary

**Illustrative monthly platform estimate: $3,225.07 USD**, comprising **$2,875.07 of Azure/Fabric resource consumption** and **$350.00 of Power BI user licenses**. With a 15% planning contingency, budget **$3,708.83/month**.

| Summary | Monthly USD |
| --- | ---: |
| Azure/Fabric resource consumption — subtotal of resource rows below | $2,875.07 |
| Power BI Pro — 25 users, shown separately from resource consumption | $350.00 |
| **Total platform operating estimate** | **$3,225.07** |
| Contingency — total × 15% | $483.76 |
| **Planning budget including contingency** | **$3,708.83** |

Here **ACR means Azure Resource Consumption**, not Azure Container Registry. This is a resource-cost planning model, **not a contractual Azure Consumed Revenue or MACC-eligibility calculation**. Confirm commercial eligibility with your Microsoft agreement.

**Rate basis: illustrative planning inputs, documented October 2026; not verified current retail quotes.** Every dollar rate below is an editable assumption, including model-token rates and monthly SKU allowances. Obtain an approved regional quote before budgeting or deploying; the precision of the arithmetic does not imply pricing accuracy. See [pricing references and recalibration](#pricing-references-and-recalibration).

The scope follows the [architecture](../../README.md#architecture--data-flow) and [setup guide](../setup/README.md): telemetry ingestion, pricing refresh, Fabric analytics and 12 ML use cases, Power BI, the CostOps UI/API, Foundry chat, identity, secrets, monitoring, and private networking. This is a **small single-environment planning baseline**, not the enterprise scenario in the [business case](../business-case/business-case.md#4-value-model-micron-scale-enterprise), whose illustrative $1.9M/year run cost assumes a different scale.

## Workload and deployment assumptions

| Important factor | Baseline assumption | Effect on consumption |
| --- | --- | --- |
| Calendar, currency, regions | 30-day workload month; 730 billed hours for always-on infrastructure; USD; West US 2 for Fabric/hosting and West US for APIM/Foundry | 730 is an average billing-month convention, not 30 × 24. Replace with actual active hours and regional prices. |
| Environments and availability | One environment, one region per service, no standby/DR deployment, no reservation or negotiated discount | Additional environments, replicas, availability requirements, and region pairs add costs. This is not a validated production sizing recommendation. |
| Telemetry | 1,000,000 events/day × 30 days; average 2 KB/event using decimal KB/GB | 30M events and 60 GB of new logical telemetry/month; token volume alone does not determine telemetry size. |
| Retention and copies | 3 months of raw and normalized data at steady state; Cosmos storage/index multiplier 2; three physical medallion layers with an aggregate overhead multiplier 2 | Blob = 180 GB; Cosmos = 360 GB; OneLake = 1,080 GB. Multipliers are placeholders for compression, indexing, Delta versions, prices, ML outputs, and model artifacts; replace with measured average billed storage. |
| Blob ingestion | 1,000 events per batch; 30,000 writes and 60,000 reads/month | Estimate assumes batching, not one storage transaction per event. Other operation classes must be added if used. |
| Fabric | One F8 capacity (8 CUs), active 730 hours; daily ingestion/scoring; 12 training notebooks weekly and 12 inference notebooks daily | All jobs, SQL/BI queries, Data Agent work, and optional ontology share this capacity. F8 is a sizing hypothesis, not proof that the workload fits. |
| Hosting and price refresh | Two dedicated App Service plans: UI and API; pricing Function shares the API plan; 24 refreshes/day × 30 = 720 runs | Charge plans/instances, not each app. Function runtime has no separate consumption-plan execution charge in this deployment; its storage remains billable. |
| Cosmos DB | Single-region provisioned throughput, 400 RU/s, no autoscale or multi-region replication | 400 RU/s is a starting hypothesis only. Benchmark ingestion, reads, item size, partition distribution, and indexes before choosing throughput. |
| Portal and BI | 25 distinct licensed users, including report authors and viewers; user-owns-data embedding; no existing licenses assumed | Below F64, viewers generally need Pro (or qualifying PPU) licenses; embedding does not remove user licensing requirements. |
| Chat | 10,000 conversations/month; 3 model calls/conversation; 4,000 input and 800 output tokens/call; 25% of input billed as cache reads | Count every agent turn, tool follow-up, system prompt, history, and retry. The cache share is assumed, not guaranteed. |
| Observability | Analytics logs: 3 GB/day of billable ingestion, 90 GB/month; 31-day included retention | Includes workspace-based Application Insights telemetry once; excludes full prompt/response logging. Extra retention, export, and query tiers add charges. |
| Private access and traffic | 8 private endpoints, 8 DNS zones, 1M DNS queries, 100 GB of chargeable Private Link processing, and 100 GB of chargeable outbound transfer/month | Illustrative endpoint inventory: UI, API, Function, Blob, Cosmos, Key Vault, Foundry, APIM. Validate required storage subresources and any Fabric endpoints; adjust counts. |

## Monthly estimate by service

**All unit prices in this table are illustrative inputs, not offers for the named SKUs.** GB represents average stored GB for storage rows, ingested GB for Monitor, and transferred GB for networking. `M` means one million. Costs are rounded to cents per row; the subtotal sums these rounded rows.

| Azure technology / service | Monthly quantity and chosen meter | Assumed unit price (USD) | Calculation | Monthly USD |
| --- | --- | --- | --- | ---: |
| **Microsoft Fabric capacity** | 1 F8 × 730 active hours | $1.44 / capacity-hour | `1 × 730 × 1.44` | **$1,051.20** |
| **OneLake storage** | 1,080 GB average stored | $0.023 / GB-month | `1,080 × 0.023` | $24.84 |
| **Azure API Management** | 1 Standard v2 instance/unit, full month | $700 / unit-month allowance | `1 × 700` | $700.00 |
| **Azure App Service** | 2 Linux P1v3 plans × 1 instance each, full month | $150 / instance-month allowance | `2 × 1 × 150` | $300.00 |
| **Azure Functions — pricing refresh compute** | 720 timer executions on the API's dedicated plan | Included in the App Service row | `0` additional compute charge; size shared plan for both workloads | $0.00 |
| **Azure Functions backing Storage account** | 10 GB; 100,000 illustrative billable operations | $0.02 / GB-month; $0.05 / 10,000 operations (blended allowance) | `10 × 0.02 + (100,000 / 10,000) × 0.05` | $0.70 |
| **Azure Cosmos DB for NoSQL** | 400 provisioned RU/s × 730 hours; 360 GB storage | $0.008 / 100 RU/s-hour; $0.25 / GB-month | `(400 / 100) × 730 × 0.008 + 360 × 0.25` | $113.36 |
| **Azure Blob Storage — raw telemetry and snapshots** | Hot LRS: 180 GB; 30,000 writes; 60,000 reads | $0.02 / GB-month; $0.05 / 10,000 writes; $0.004 / 10,000 reads | `180 × 0.02 + 3 × 0.05 + 6 × 0.004` | $3.77 |
| **Azure Monitor / Log Analytics / Application Insights** | 90 GB billable ingestion; no extra retention | $2.76 / GB ingested | `90 × 2.76` | $248.40 |
| **Microsoft Foundry / Azure OpenAI — platform agent inference** | 90M uncached input, 30M cached input, 24M output tokens | $1.25 / M uncached input; $0.25 / M cache read; $10 / M output | `90 × 1.25 + 30 × 0.25 + 24 × 10` | $360.00 |
| **Azure Key Vault — standard secret operations** | 100,000 secret operations; no premium keys/HSM | $0.03 / 10,000 operations | `(100,000 / 10,000) × 0.03` | $0.30 |
| **Azure Private Link / private endpoints** | 8 × 730 endpoint-hours; 100 GB processing | $0.01 / endpoint-hour; $0.01 / processed GB | `8 × 730 × 0.01 + 100 × 0.01` | $59.40 |
| **Azure Private DNS** | 8 zones; 1M queries | $0.50 / zone-month; $0.40 / M queries | `8 × 0.50 + 1 × 0.40` | $4.40 |
| **Azure bandwidth — outbound/cross-region** | 100 GB chargeable transfer after applicable allowances | $0.087 / GB blended allowance | `100 × 0.087` | $8.70 |
| **Microsoft Entra ID / managed identities** | App registrations, OBO token flow, baseline identity | No incremental premium identity licenses assumed | `0`; add tenant security licenses if required | $0.00 |
| **Azure VNet / subnets / App Service VNet integration** | Base networking only, no paid gateway/appliance | No additional base-resource meter assumed | `0`; private endpoints, DNS, and transfer are priced above | $0.00 |
| **Azure/Fabric resource subtotal** | Sum of resource rows | | | **$2,875.07** |
| **Power BI Pro licenses (separate)** | 25 users | $14 / user-month allowance | `25 × 14` | **$350.00** |
| **Total** | Resource subtotal + licenses | | `2,875.07 + 350.00` | **$3,225.07** |

### Service coverage and avoiding double counting

- **Fabric Dataflow Gen2, pipelines, Lakehouse, Spark/PySpark training and inference, SQL analytics endpoint, Power BI semantic-model/report compute, and Fabric Data Agent** consume the shared Fabric capacity. Data Agent AI tokens and its generated queries both consume capacity; OneLake operations also consume capacity rather than separate Blob-style transaction fees. Do not add a second standalone Spark cluster, SQL database, or per-notebook compute charge. Capacity throttling or billable overage settings can change this model; measure the actual CU consumption and add applicable charges.
- **Fabric IQ ontology/GraphModel** is optional/preview in the setup guide. No separate ontology allowance is included; verify its current availability and billing before enabling it. Its workload must fit the capacity budget.
- **Microsoft Foundry project and classic agent orchestration** have no separate allowance here; underlying model calls are counted in the inference row. Fabric Data Agent's capacity consumption is separate from those Foundry tokens. New hosted-agent runtimes or paid tools need their own meters.
- **Pricing APIs, saved prompts, SHA-256 pseudonymization, model-price history, and ML artifacts** have no extra standalone Azure service line. Their hosting, storage, logs, and Fabric processing are covered above; public catalog access is not model inference.
- **Ionic/Angular, Node, and FastAPI** are application technologies, not additional Azure billing meters. The UI has its own plan; API and pricing Function share the second. Function storage is separate from raw Blob storage.
- **APIM Standard v2** is an illustrative private-access choice, not the Consumption or Developer tier. Validate inbound private-endpoint support, outbound VNet integration, region availability, policy support, and SLA against the actual deployment. Full VNet injection, more units, or a different tier changes the quote.
- **Power BI** uses the same Fabric capacity, not a separate Embedded capacity in this user-owns-data baseline. Report viewer licenses are additional; existing eligible licenses can reduce incremental spend. F64+ can change viewer licensing, but authors still require appropriate licenses and capacity compute increases.
- **Container hosting/Registry, Azure AI Search/Foundry IQ retrieval, embeddings, agent file search/code interpreter, Cosmos mirroring, NAT Gateway, Firewall, VPN/ExpressRoute, Front Door, paid Purview governance, premium Entra security, Defender, backups beyond stated retention, and geo-replication** are not additional deployed services in this baseline. If your implementation adds any, add a row using its actual runtime, provisioned units, storage, transactions, or user licenses. Do not interpret an excluded service as free.

## Formulas and sizing checks

### Storage and ingestion

```text
monthly_events = events_per_day × workload_days
monthly_logical_GB = monthly_events × average_event_KB / 1,000,000
average_raw_GB = monthly_logical_GB × retained_months
average_cosmos_GB = average_raw_GB × storage_and_index_multiplier
average_onelake_GB = monthly_logical_GB × retained_months × layer_count × overhead_multiplier

baseline: 1,000,000 × 30 = 30,000,000 events
          30,000,000 × 2 / 1,000,000 = 60 GB/month
          raw = 60 × 3 = 180 GB
          Cosmos = 180 × 2 = 360 GB
          OneLake = 60 × 3 × 3 × 2 = 1,080 GB

storage_cost = average_billed_GB × price_per_GB_month
operation_cost = SUM(operation_count_by_type / billing_unit × unit_price_by_type)
```

These are steady-state estimates, not first-month end-of-month storage. Use provider-metered units where they differ from decimal GB, and replace the multipliers with actual billed footprints. Add model artifacts or snapshots separately if they exceed the assumed overhead.

```text
required_RU_per_second =
  (writes_per_second × measured_RU_per_write
   + reads_per_second × measured_RU_per_read
   + query_RU_per_second) × peak_and_headroom_factor

Cosmos_provisioned_cost =
  (provisioned_RU_per_second / 100) × billed_hours × rate_per_100_RU_hour
  + average_storage_GB × storage_rate
```

For orientation, 1M events/day averages about 11.57 writes/second; at an assumed 10 RU/write that is about 116 RU/s **before** reads, indexing effects, and bursts. With a 3× headroom factor it is about 347 RU/s before reads; 400 RU/s may therefore need increasing. Autoscale and serverless have different billing formulas; do not apply the provisioned formula to them.

### Fabric and hosting

```text
Fabric_cost = SUM(capacity_count_by_SKU × active_hours_by_SKU × hourly_SKU_price)
App_Service_cost = SUM(plan_instance_count × billed_instance_hours × hourly_instance_price)
```

The table uses full-month hosting allowances instead of hourly quotes. Fabric's internal CU consumption determines **required capacity**, not an extra per-job charge on top of this fixed F8 baseline. Size with the Fabric Capacity Metrics app: ingestion/refresh overlap, the 12 training and scoring jobs, interactive SQL/DirectQuery, ontology, and Data Agent concurrency all matter. Do not assume pausing preserves chat/report availability or eliminates storage charges; pausing can also settle outstanding smoothed usage.

### Foundry model calls and platform boundary

```text
model_calls = conversations × calls_per_conversation
input_tokens = model_calls × input_tokens_per_call
cached_input_tokens = input_tokens × billed_cache_read_fraction
uncached_input_tokens = input_tokens - cached_input_tokens
output_tokens = model_calls × output_tokens_per_call

inference_cost = SUM(
  uncached_input_tokens / 1,000,000 × input_rate
  + cached_input_tokens / 1,000,000 × cache_read_rate
  + output_tokens / 1,000,000 × output_rate
) by model, deployment type, and effective price interval

baseline: 10,000 × 3 = 30,000 calls
          input = 30,000 × 4,000 = 120M (90M uncached + 30M cached)
          output = 30,000 × 800 = 24M
          cost = 90 × 1.25 + 30 × 0.25 + 24 × 10 = $360
```

The token prices are **hypothetical**, not a quote for the README's `gpt-5.2` deployment or any other named model. Replace them with the actual model/version, region, deployment mode, and negotiated rate. Use zero cache share if caching is unavailable; add cache-write meters, reasoning/output tokens, paid tools, and retry/agent-loop usage where applicable. Provisioned-throughput deployments require their capacity/commitment pricing rather than this pay-per-token formula.

**Do not confuse governed enterprise AI spend with the cost of running CostOps.** The baseline includes only the platform's own Foundry assistant inference, not the Azure OpenAI workloads it observes and not AWS Bedrock, Vertex AI, OpenAI Direct, or Anthropic bills. For an expanded Azure budget:

```text
expanded_Azure_resource_total =
  $2,875.07 + monitored_Azure_model_spend_not_already_counted + added_Azure_services
```

Estimate monitored Azure inference separately using the same effective-dated token formula and `prices.token_price_history`; reconcile billed actuals to Azure Cost Management. Exclude platform-agent calls from that added workload subtotal if already counted above. Non-Azure model spend belongs in a separate multi-cloud total, not Azure resource consumption.

### Sensitivity examples

| Change, all other inputs held constant | Monthly impact |
| --- | ---: |
| Double platform conversations to 20,000, same call/token/cache mix | +$360.00 inference; infrastructure may also need resizing |
| No billed input cache hits instead of 25% | +$30.00 (`30M × (1.25 − 0.25)`) |
| Add 1 GB/day of billable logs | +$82.80 (`30 × 2.76`) |
| Add one private endpoint for the full month | +$7.30 before processing/DNS (`730 × 0.01`) |
| Add 10 Pro users at the assumed allowance | +$140.00 |
| Increase retention from 3 to 6 months at steady state | +$118.44 storage only: Blob $3.60 + Cosmos $90.00 + OneLake $24.84 |

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

1. Record the quote date, region, currency, service/SKU, meter and billing unit, OS, redundancy, model/version, deployment mode, and discount/commitment terms alongside every replaced rate. Use licensing quotes for Power BI; do not assume every meter is available from the Retail Prices API.
2. Measure actual event sizes and billed storage, RU charges and bursts, Fabric CU utilization, App Service instance count, APIM tier/traffic allowances, model tokens/cache behavior, log ingestion/retention, and chargeable network traffic.
3. Recalculate each service row, resource subtotal, separate licenses, and contingency. Add any incremental units, overage meters, tools, retention, recovery, or connectivity services required by the real design.
4. Reconcile monthly to Azure/Fabric billing and license invoices, preserving market, negotiated, and billed costs separately. Apply tags/cost-center ownership; do not allocate the full cost of an already shared capacity or plan to CostOps twice.

Taxes, support, implementation labor, migration, training, and non-Azure provider charges are excluded. Free grants, trials, existing licenses, reservations, and negotiated discounts are not credited in this baseline. Contingency is a planning reserve, not a billed service or measured ACR.
