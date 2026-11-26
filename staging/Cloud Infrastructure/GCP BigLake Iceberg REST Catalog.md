---
tags:
  - cloud-infrastructure/services/gcp
  - data-processing/storage/iceberg
aliases:
  - BigLake Iceberg REST Catalog
  - BigLake Metastore REST Catalog
source: "Google Cloud BigLake Iceberg REST Catalog Documentation"
source-type: documentation
vendor: "Google Cloud"
url: "https://cloud.google.com/biglake/docs/blms-rest-catalog"
created: 2026-11-25
---
**GCP BigLake Iceberg REST Catalog** (part of BigLake Metastore / Lakehouse runtime catalog) is Google Cloud's serverless, Spanner-backed implementation of the open [[Apache Iceberg REST Catalog]] protocol (`https://biglake.googleapis.com/iceberg/v1/restcatalog`). It lets **BigQuery, Apache Spark, Flink, Trino, PyIceberg, and custom Java Iceberg SDKs** share open [[Apache Iceberg]] tables on GCS with native **credential vending**—while preserving cloud-agnostic portability across GCP and AWS.

> [!IMPORTANT]
> In a clean platform architecture, **OpenTofu provisions the catalog resource (`google_biglake_iceberg_catalog`)**, while Iceberg clients (such as a Java `sdk-lakehouse` CLI or Spark) manage **namespaces and tables** through the BigLake REST endpoint. Choosing BigLake Iceberg REST Catalog over [[GCP BigLake Tables for Apache Iceberg]] keeps your Bronze layer decoupled from BigQuery-only DDL so porting to AWS (Glue Iceberg REST / S3 Tables) requires only swapping IaC and catalog config.

## Quick Comparison: The Three `catalog_type` Options

| `catalog_type` | Mental Model | Catalog `name` Constraint | Storage & `default_location` | Client `warehouse` URI | Best Fit |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`CATALOG_TYPE_BIGLAKE`** | **Logical multi-bucket catalog** (modern default) | Any logical ID (e.g., `bronze` across `dev`/`uat`/`prod`) | Multi-bucket capable; `default_location = "gs://<bucket>"` is **required** | `bl://projects/<project>/catalogs/<catalog>` | Greenfield GCP lakehouses, Data Mesh domains, multi-cloud parity with AWS Glue / Polaris. |
| **`CATALOG_TYPE_GCS_BUCKET`** | **1:1 bucket-locked catalog** (legacy / simple) | **Must equal the exact GCS bucket name** (`udp-dev-bronze-a1b2`) | Strictly one GCS bucket; `default_location` is output-only | `gs://<exact-bucket-name>` | Simple single-bucket setups or older BigLake integrations where bucket name = catalog name. |
| **`CATALOG_TYPE_FEDERATED`** | **Cross-cloud metadata mirror / proxy** | Logical federated catalog ID (e.g., `aws_glue_prod`) | Mirrors remote metadata in `primary_location`; data stays in AWS S3 / Databricks | Federated catalog URI (read via BigLake / BigQuery) | Querying external AWS Glue or Databricks Unity Catalog Iceberg tables from GCP without copying Parquet files. |

---

## 1. `CATALOG_TYPE_BIGLAKE` (Logical Multi-Bucket Catalog — Recommended)

`CATALOG_TYPE_BIGLAKE` decouples the **logical catalog name** from physical GCS bucket names. You name the catalog `bronze` in every environment (`dev`, `uat`, `prod`) while `default_location` points to that environment's randomized GCS bucket—and individual namespaces or domains can optionally override storage onto dedicated buckets.

```mermaid
%%{init: {"theme": "base", "themeCSS": "svg { background-color: #09090B !important; border-radius: 8px; }", "themeVariables": {"darkMode": true, "background": "#09090B", "mainBkg": "#18181B", "textColor": "#F4F4F5", "primaryColor": "#18181B", "primaryTextColor": "#F4F4F5", "primaryBorderColor": "#7C3AED", "lineColor": "#A855F7", "secondaryColor": "#16141F", "tertiaryColor": "#0F0F14", "edgeLabelBackground": "#18181B", "clusterBkg": "#0F0F14", "clusterBorder": "#52525B", "titleColor": "#D8B4FE", "fontFamily": "Inter, ui-sans-serif, system-ui, -apple-system, sans-serif", "fontSize": "13px"}, "flowchart": {"curve": "basis", "nodeSpacing": 42, "rankSpacing": 56, "padding": 14}}}%%
flowchart TB
    subgraph Clients["1. Multi-Engine Clients · warehouse = bl://projects/proj/catalogs/bronze"]
        SDK["Java SDK / PyIceberg\nGoogleAuthManager\nX-Iceberg-Access-Delegation"]
        SPK["Dataproc Spark / Flink\nRESTCatalog + GCSFileIO"]
        BQ["BigQuery Engine\nCloud Resource Connection"]
    end

    subgraph ControlPlane["2. BigLake Logical Catalog · CATALOG_TYPE_BIGLAKE"]
        REST["Catalog: bronze\nbiglake.googleapis.com/iceberg/v1/restcatalog\ndefault_location = gs://udp-env-bronze-xxxx"]
        VEND["Credential Vending Gate\nbiglake_service_account\nTable-Scoped GCS Tokens"]
    end

    subgraph Storage["3. One or More GCS Buckets"]
        B1[("Default Warehouse Bucket\ngs://udp-env-bronze-xxxx\nnamespace: football")]
        B2[("Optional Domain Bucket\ngs://udp-env-finance-xxxx\nnamespace: finance")]
    end

    SDK -->|"HTTPS + OAuth2"| REST
    SPK -->|"HTTPS + OAuth2"| REST
    BQ -->|"BigLake Metadata"| REST
    REST -->|"Mint table-scoped token"| VEND
    SDK ==>|"Vended GCS Creds"| B1
    SPK ==>|"Vended GCS Creds"| B2
    BQ ==>|"Connection SA · objectViewer"| B1

    classDef core fill:#2E1065,stroke:#A855F7,stroke-width:2px,color:#F5F3FF
    classDef service fill:#18181B,stroke:#7C3AED,stroke-width:1.5px,color:#F4F4F5
    classDef store fill:#09090B,stroke:#C084FC,stroke-width:1.5px,color:#EDE9FE
    classDef guard fill:#3B0764,stroke:#E879F9,stroke-width:2px,stroke-dasharray:5 3,color:#FAF5FF

    class REST core
    class SDK,SPK,BQ service
    class VEND guard
    class B1,B2 store

    style Clients fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
    style ControlPlane fill:#16141F,stroke:#6D28D9,stroke-width:1.25px,stroke-dasharray:4 3,color:#C4B5FD
    style Storage fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
```

- **Environment Parity:** Because the logical name `bronze` stays constant across `dev`, `uat`, and `prod`, domain table YAML/SQL never hardcodes randomized GCS bucket names.
- **Credential Vending (`CREDENTIAL_MODE_VENDED_CREDENTIALS`):** Grant `biglake_service_account` `roles/storage.objectUser` on the bucket(s); clients sending `X-Iceberg-Access-Delegation: vended-credentials` receive ephemeral, table-scoped GCS tokens without holding direct bucket IAM.
- **Fine-Grained IAM:** Supports `google_biglake_iceberg_catalog_iam_member`, `google_biglake_iceberg_namespace_iam_member`, and `google_biglake_iceberg_table_iam_member` for Data Mesh isolation.

```hcl
resource "google_biglake_iceberg_catalog" "bronze" {
  project          = var.project_id
  name             = "bronze"
  catalog_type     = "CATALOG_TYPE_BIGLAKE"
  default_location = "gs://${module.bronze_bucket.name}"
  credential_mode  = "CREDENTIAL_MODE_VENDED_CREDENTIALS"
  deletion_policy  = "PREVENT"

  depends_on = [google_project_service.lakehouse_apis, module.bronze_bucket]
}

resource "google_storage_bucket_iam_member" "biglake_vended_creds" {
  bucket = module.bronze_bucket.name
  role   = "roles/storage.objectUser"
  member = "serviceAccount:${google_biglake_iceberg_catalog.bronze.biglake_service_account}"
}
```

```properties
# Java Iceberg 1.10+ / sdk-lakehouse (CATALOG_BACKEND=gcp_biglake)
type=rest
uri=https://biglake.googleapis.com/iceberg/v1/restcatalog
warehouse=bl://projects/udp-dev-xxxx/catalogs/bronze
io-impl=org.apache.iceberg.gcp.gcs.GCSFileIO
rest.auth.type=org.apache.iceberg.gcp.auth.GoogleAuthManager
header.x-goog-user-project=udp-dev-xxxx
header.X-Iceberg-Access-Delegation=vended-credentials
```

---

## 2. `CATALOG_TYPE_GCS_BUCKET` (Single-Bucket 1:1 Catalog)

`CATALOG_TYPE_GCS_BUCKET` binds a BigLake catalog **1:1 to a single pre-existing GCS bucket**. The catalog `name` is forced to match the exact GCS bucket name, `default_location` is read-only, and every namespace and table lives strictly inside that one bucket.

```mermaid
%%{init: {"theme": "base", "themeCSS": "svg { background-color: #09090B !important; border-radius: 8px; }", "themeVariables": {"darkMode": true, "background": "#09090B", "mainBkg": "#18181B", "textColor": "#F4F4F5", "primaryColor": "#18181B", "primaryTextColor": "#F4F4F5", "primaryBorderColor": "#7C3AED", "lineColor": "#A855F7", "secondaryColor": "#16141F", "tertiaryColor": "#0F0F14", "edgeLabelBackground": "#18181B", "clusterBkg": "#0F0F14", "clusterBorder": "#52525B", "titleColor": "#D8B4FE", "fontFamily": "Inter, ui-sans-serif, system-ui, -apple-system, sans-serif", "fontSize": "13px"}, "flowchart": {"curve": "basis", "nodeSpacing": 42, "rankSpacing": 56, "padding": 14}}}%%
flowchart LR
    subgraph Client["1. Iceberg Client"]
        CLI["Spark / PyIceberg / SDK\nwarehouse = gs://udp-dev-bronze-a1b2"]
    end

    subgraph Catalog["2. Bucket-Coupled Catalog · CATALOG_TYPE_GCS_BUCKET"]
        CAT["Catalog ID: udp-dev-bronze-a1b2\nName MUST equal GCS bucket name\ndefault_location is output-only"]
    end

    subgraph Bucket["3. Single GCS Bucket"]
        GCS[("gs://udp-dev-bronze-a1b2\nAll namespaces & tables\nlocked to this bucket")]
    end

    CLI -->|"REST API · warehouse=gs://..."| CAT
    CAT -->|"1:1 Strict Binding"| GCS
    CLI ==>|"Direct or Vended GCS Access"| GCS

    classDef core fill:#2E1065,stroke:#A855F7,stroke-width:2px,color:#F5F3FF
    classDef service fill:#18181B,stroke:#7C3AED,stroke-width:1.5px,color:#F4F4F5
    classDef store fill:#09090B,stroke:#C084FC,stroke-width:1.5px,color:#EDE9FE

    class CAT core
    class CLI service
    class GCS store

    style Client fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
    style Catalog fill:#16141F,stroke:#6D28D9,stroke-width:1.25px,stroke-dasharray:4 3,color:#C4B5FD
    style Bucket fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
```

- **Why `CATALOG_TYPE_BIGLAKE` usually wins:** GCS bucket names are globally unique across all of Google Cloud and typically include random suffixes (`-a1b2`). Coupling catalog identity to bucket name leaks environment-specific names into catalog identifiers and prevents placing separate domain buckets under one catalog.

```hcl
resource "google_biglake_iceberg_catalog" "single_bucket" {
  project         = var.project_id
  name            = module.bronze_bucket.name # MUST match GCS bucket name verbatim
  catalog_type    = "CATALOG_TYPE_GCS_BUCKET"
  credential_mode = "CREDENTIAL_MODE_VENDED_CREDENTIALS"
  deletion_policy = "PREVENT"

  depends_on = [google_project_service.lakehouse_apis, module.bronze_bucket]
}
```

---

## 3. `CATALOG_TYPE_FEDERATED` (Cross-Cloud External Catalog Proxy)

`CATALOG_TYPE_FEDERATED` turns BigLake into a **read/discovery mirror** over an external Iceberg REST catalog—specifically **AWS Glue / S3 Table Buckets** (`glue_catalog_info`) or **Databricks Unity Catalog** (`unity_catalog_info`). BigLake periodically refreshes remote catalog metadata into `primary_location` on a schedule (`refresh_interval >= 300s`) so BigQuery and GCP engines query remote tables without duplicating data into GCS.

```mermaid
%%{init: {"theme": "base", "themeCSS": "svg { background-color: #09090B !important; border-radius: 8px; }", "themeVariables": {"darkMode": true, "background": "#09090B", "mainBkg": "#18181B", "textColor": "#F4F4F5", "primaryColor": "#18181B", "primaryTextColor": "#F4F4F5", "primaryBorderColor": "#7C3AED", "lineColor": "#A855F7", "secondaryColor": "#16141F", "tertiaryColor": "#0F0F14", "edgeLabelBackground": "#18181B", "clusterBkg": "#0F0F14", "clusterBorder": "#52525B", "titleColor": "#D8B4FE", "fontFamily": "Inter, ui-sans-serif, system-ui, -apple-system, sans-serif", "fontSize": "13px"}, "flowchart": {"curve": "basis", "nodeSpacing": 42, "rankSpacing": 56, "padding": 14}}}%%
flowchart TB
    subgraph GCPZone["1. Google Cloud Platform"]
        BQ["BigQuery / Dataproc Spark\nQueries federated catalog"]
        FED["BigLake Federated Catalog\nCATALOG_TYPE_FEDERATED\nprimary_location · refresh >= 300s"]
        SEC["Secret Manager / OIDC\nCross-Cloud Auth Credentials"]
    end

    subgraph ExtCloud["2. External Cloud · AWS / Databricks"]
        EXT["Remote Iceberg REST Catalog\nAWS Glue or Unity Catalog\nnamespace_filters"]
        S3[("Remote Object Storage\nAmazon S3 / S3 Tables\nParquet stays in-place")]
    end

    BQ -->|"Query remote tables"| FED
    FED -.->|"Fetch auth / assume role"| SEC
    FED <-->|"Scheduled metadata sync"| EXT
    BQ ==>|"Zero-copy cross-cloud read"| S3

    classDef core fill:#2E1065,stroke:#A855F7,stroke-width:2px,color:#F5F3FF
    classDef service fill:#18181B,stroke:#7C3AED,stroke-width:1.5px,color:#F4F4F5
    classDef store fill:#09090B,stroke:#C084FC,stroke-width:1.5px,color:#EDE9FE
    classDef guard fill:#3B0764,stroke:#E879F9,stroke-width:2px,stroke-dasharray:5 3,color:#FAF5FF
    classDef external fill:#27272A,stroke:#71717A,stroke-width:1.5px,color:#E4E4E7

    class FED core
    class BQ service
    class SEC guard
    class EXT external
    class S3 store

    style GCPZone fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
    style ExtCloud fill:#18181B,stroke:#52525B,stroke-width:1.25px,stroke-dasharray:5 4,color:#A1A1AA
```

```hcl
resource "google_biglake_iceberg_catalog" "aws_glue_federated" {
  project          = var.project_id
  name             = "aws_glue_prod"
  catalog_type     = "CATALOG_TYPE_FEDERATED"
  primary_location = "us-central1"

  federated_catalog_options {
    glue_catalog_info {
      aws_region   = "us-east-1"
      aws_role_arn = "arn:aws:iam::123456789012:role/BigLakeGlueReadRole"
      warehouse    = "123456789012"
    }

    refresh_options {
      refresh_schedule {
        refresh_interval = "300s"
      }
      refresh_scope {
        namespace_filters = ["football", "finance"]
      }
    }
  }
}
```

---

## Multi-Cloud Platform Parity (GCP vs. AWS)

Using `CATALOG_TYPE_BIGLAKE` over the open [[Apache Iceberg REST Catalog]] protocol keeps your application and schema code identical across clouds—only the OpenTofu module and SPI plugin properties change:

| Layer | GCP (`platform_unified_data_gcp`) | AWS (`platform_unified_data_aws`) | Shared Unchanged |
| :--- | :--- | :--- | :--- |
| **Object Storage** | GCS Bucket (`cloud-foundation-fabric//modules/gcs`) | Amazon S3 Bucket / S3 Table Bucket | Iceberg v2 Parquet layout |
| **REST Catalog** | `google_biglake_iceberg_catalog` (`CATALOG_TYPE_BIGLAKE`) | AWS Glue Iceberg REST / S3 Tables / Apache Polaris | Standard Iceberg REST API |
| **DDL / Schema Runner** | `sdk-lakehouse` (`CATALOG_BACKEND=gcp_biglake`) | `sdk-lakehouse` (`CATALOG_BACKEND=aws_glue` / `rest`) | Same domain `table.yaml` / `table.proto` |
| **Auth / Storage IO** | `GoogleAuthManager` + `GCSFileIO` + vended creds | SigV4 / STS + `S3FileIO` + vended creds | Zero business-logic changes |

## Related Notes
- **OSS Counterpart & Concepts:** [[Apache Iceberg REST Catalog]] · [[Apache Iceberg Catalogs]] · [[Apache Iceberg]]
- **Sibling GCP Iceberg Options:** [[GCP BigLake Tables for Apache Iceberg]] (BigQuery-managed DDL/DML + auto-compaction) · [[GCP Dataproc Metastore and BigLake Hive Catalog]] (managed Thrift HMS)

## When Not to Use
- When **BigQuery is the sole transactional writer** and you want zero manual Iceberg maintenance/compaction jobs (evaluate [[GCP BigLake Tables for Apache Iceberg]]).
- When you require **multi-table Git-like catalog branching** across commits (evaluate self-hosted [[Apache Iceberg Nessie Catalog]]).

## Documentation & References
- [Google Cloud BigLake Iceberg REST Catalog Docs](https://cloud.google.com/biglake/docs/blms-rest-catalog)
- [Terraform/OpenTofu `google_biglake_iceberg_catalog` Resource](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/biglake_iceberg_catalog)
- [Apache Iceberg `GoogleAuthManager` (Iceberg 1.10+)](https://iceberg.apache.org/docs/latest/gcp/)
