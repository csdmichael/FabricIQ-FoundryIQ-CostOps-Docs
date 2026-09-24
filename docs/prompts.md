# Multi-Cloud AI Tokenomics & FinOps Platform Prompts

Use these prompts and commands to manage the Microsoft Fabric AI Tokenomics workspace, PySpark ML pipelines, Power BI reports and dashboards, and the CostOps web portal.

## Workspace Architecture & Medallion Data Plane

1. **Multi-Cloud Token Consumption Ingestion:**
   - Ingests raw invocation logs from AWS Bedrock, Google Cloud Vertex AI, OpenAI Direct, Anthropic Claude, and Azure OpenAI / Foundry.
   - Cleanses, deduplicates, and pseudonymizes user identifiers into Cosmos DB (`token-consumption`).
   - Stages data into Fabric Lakehouse Delta tables partitioned under source schemas: `aws`, `gcp`, `oai`, `cld`, and `msft`.

2. **Model Pricing Synchronization & Provenance:**
   - Polls published pricing APIs (AWS Bulk Price List, Google Cloud Billing API, Azure Retail Prices API, OpenAI pricing, Anthropic pricing).
   - Generates SHA-256 provenance hashes in `model_price_snapshot`.
   - Normalizes effective-dated pricing intervals in `model_price_history`.
   - Computes deterministic as-of cost enrichment in `ml.tokenomics_usage_fact`.

3. **12 Production PySpark ML Algorithms:**
   - 01 Anomaly Detection
   - 02 Cost Forecasting
   - 03 Cost Attribution and Chargeback
   - 04 User Segmentation (K-Means / DBSCAN)
   - 05 Token Efficiency Scoring
   - 06 Budget Overrun Prediction
   - 07 Model Optimization Recommendation Engine (40%-80% potential savings)
   - 08 Agent ROI Analytics
   - 09 Intelligent Quota Management (TPM/RPM recommendations)
   - 10 Prompt Quality Analytics
   - 11 Capacity Planning
   - 12 Executive AI Adoption Scorecard

4. **Power BI Analytics & Executive Dashboards:**
   - DirectQuery Semantic Model: `Tokenomics FinOps Model`
   - Consumption & Cost Report: `Tokenomics Consumption and Cost Analytics`
   - ML Insights Report: `Tokenomics ML Insights`
   - Executive Dashboard: `Tokenomics FinOps Executive Dashboard`
   - Operations Dashboard: `Tokenomics ML Operations Dashboard`

5. **Ionic / Angular CostOps Web UI & Fabric Data Agent:**
   - Responsive multi-device web portal.
   - Interactive Power BI report viewer.
   - 12 ML insight dashboards.
   - Conversational AI chat with Fabric Data Agent (`Tokenomics FinOps Analyst`).
   - Configurable saved prompt library.
