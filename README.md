# 🏦 Enterprise BI Report Prototype for Mal
A single-file, working prototype of an enterprise BI report for Mal - a newly launched Islamic digital bank in the UAE — built solo, from scratch, as the first BI hire. It covers an executive dashboard, a Credit & Lending domain dashboard, a metric conflict resolution view, a KPI dictionary, and a self-service portal access steps.
Status: Prototype/discovery artifact. All data is synthetic. Not wired to any production system.
________________________________________
# Getting started
There is nothing to install and nothing to build. The file is fully self-contained and works offline.
Standing up the production stack (roadmap)
To have the file for production data the following steps need to be done:
1.	Set up Unity catalog for Mal Bank with bronze, silver, and gold schemas; register the Azure UAE storage location;  grant catalog/schema access per domain.
2.	Ingest the bronze layer by landing raw exports from Core all the source systems
3.	Conform the clean silver layer enforcing data-quality expectations
4.	Curate the gold layer by building star schema around customer and aggregated tables that the KPI Dictionary documents.
5.	Build a Power BI semantic layer by connecting to the gold schema, apply Certified endorsement, publish to the workspace, configure RLS via Entra ID groups.
6.	Rebuild the five report pages from this mockup as Power BI report pages against that semantic model.
________________________________________
# Stack choice
Production target: Databricks (Lakehouse Platform, Unity Catalog) for data and governance, Power BI for the semantic layer and reporting is being proposed. 

**Why Databricks**
-	Provides a single Lakehouse platform for ETL, warehouse, and ML
-	Has an in-built Unity Catalog which is the enforcement mechanism for single governed namespace, fine-grained access control and full lineage .Thus, issues like multiple KPIs for ‘Active Customers’ cannot occur
-	Regulatory and CBUAE audit requests routinely ask "what did this number look like as of date X." Delta's transaction log and time travel answer that natively, without a custom snapshotting pipeline.
-	Deployable in an Azure UAE region, which matters for CBUAE data localization expectations on customer financial data.
-	Medallion Architecture with Bronze (raw, as-landed from source systems) → Silver (cleaned, conformed, deduplicated by CIF) → Gold (the star schema and certified Metric Views this report's KPIs are built on) is a well-worn pattern for exactly this kind of multi-source, multi-domain consolidation.

**Why Power BI**
-	Power BI's dataset Certified and Promoted endorsement labels are the exact governance concept this report's KPI Dictionary is emulating by hand. In production, "Certified" on a metric isn't a column value in a table — it's a real workspace-level badge, restricted to whoever Data Governance grants Certification rights to.
-	Power BI's Data Hub (dataset/report catalog with descriptions, ownership, and endorsement filters) is the production equivalent of this repo's Self-Service Portal page — it's where a domain analyst actually finds and requests access to a dataset, rather than reading a static table.
-	Power BI can query Delta tables directly — no import refresh lag, no DirectQuery latency penalty. Gold-layer tables built once in Databricks are queried live, not copied.
-	Domain-scoped access (e.g., a Credit analyst sees the Credit workspace, not GL detail) maps onto the bank's existing Microsoft 365 / Entra ID identities — no separate identity system to maintain.
-	The workspaces within Power BI help segregate reports for different verticals and ensure that proper access control is maintained
-	Widely used and already licensed in most UAE enterprises running Microsoft 365, which lowers the adoption barrier for domain analysts who've used it before
________________________________________
# Data model
As Power BI works well with a start schema, a similar data model is proposed with customer being the centre and joined with each of the table using customer_id. A logical data model for the semantic layer is as below:
 
<img width="697" height="332" alt="image" src="https://github.com/user-attachments/assets/28cfa0c3-5f8b-4407-aa58-40def9d71d74" />


The KPI Dictionary page in the report is the human-readable version of this model: each metric row states which Gold table or Metric View it's sourced from, its formula, its owner, and its refresh cadence, so the mapping from "number on a dashboard" back to "table and column" is never a mystery — and in production, that mapping is enforced by Unity Catalog rather than just documented.
________________________________________
# How to interpret the Metric Conflict Resolution view
This page exists because of a real, common failure mode in BI: three teams built a number called "Active Customers," all three were defensible, and none of them agreed. The page is not a novelty — it's the template for how this org resolves (and re-resolves) metric disputes going forward.
What's on the page
The page contains the list of all the disputed KPIs as drop down and their resolution by the Metric Council. 
How to read it
The page has a KPI drop down option for all the disputed KPIs. On having chosen the KPI of interest it mentions:
-	**Metric Name:** The different names for the chosen KPI
-	**Vertical:** The verticals who are using that particular metric name for their analysis
-	**Source:** The source of the metric for each vertical
-	**Definition:** The definition or the logic they have implemented to populate the metric
-	**Result:** Total count for that metric obtained by each of the vertical
-	**Decision:** Final decision of the Metric Council as to how that particular KPI needs to be handled
  
Along with the above fields, a detailed Metric Council Verdict is also mentioned which led to the final decision and is also supported by the Governance Principle if applicable
