---
tags:
  - data-processing/storage/iceberg
aliases:
  - Iceberg REST Catalog
  - RESTCatalog
  - Apache Polaris
source: "Apache Iceberg REST Catalog Specification & Apache Polaris Docs"
source-type: documentation
vendor: "Apache Software Foundation"
url: "https://iceberg.apache.org/concepts/catalog/"
created: 2026-11-25
---
The **Apache Iceberg REST Catalog (`RESTCatalog`)** is the open-standard **HTTP/JSON control plane** defined by an OpenAPI specification that decouples compute engines from underlying metastore storage, enabling cross-language interoperability, server-side commit resolution, and **storage credential vending**.

> [!IMPORTANT]
> Unlike legacy catalogs that require every client to bundle backend-specific JVM libraries (Hive Thrift, JDBC drivers, AWS SDKs), `RESTCatalog` pushes catalog implementation behind a standard HTTP API. Open-source servers implementing this spec include **Apache Polaris**, **Lakekeeper**, and **Project Nessie (REST mode)**; on Google Cloud, it maps directly to [[GCP BigLake Iceberg REST Catalog]].

## Protocol & Credential Vending Architecture

```mermaid
%%{init: {"theme": "base", "themeCSS": "svg { background-color: #09090B !important; border-radius: 8px; }", "themeVariables": {"darkMode": true, "background": "#09090B", "mainBkg": "#18181B", "textColor": "#F4F4F5", "primaryColor": "#18181B", "primaryTextColor": "#F4F4F5", "primaryBorderColor": "#7C3AED", "lineColor": "#A855F7", "secondaryColor": "#16141F", "tertiaryColor": "#0F0F14", "edgeLabelBackground": "#18181B", "clusterBkg": "#0F0F14", "clusterBorder": "#52525B", "titleColor": "#D8B4FE", "fontFamily": "Inter, ui-sans-serif, system-ui, -apple-system, sans-serif", "fontSize": "13px"}, "flowchart": {"curve": "basis", "nodeSpacing": 42, "rankSpacing": 56, "padding": 14}}}%%
flowchart TB
    subgraph Clients["1. Polyglot Compute Clients (Zero DB / Thrift Deps)"]
        SPK["Apache Spark / Flink\nIceberg REST Client\nZero Bucket IAM"]
        PY["PyIceberg / DuckDB\nPython / C++ Client\nZero JVM Overhead"]
        SDK["Java Lakehouse SDK\nRESTCatalog + FileIO\nVended Credentials"]
    end

    subgraph ControlPlane["2. Iceberg REST Catalog Server"]
        API["OpenAPI REST Endpoint\n/v1/config · /v1/namespaces · /v1/tables\nServer-Side Commit Assertions"]
        AUTH["Auth & RBAC Policy Gate\nOAuth2 / OIDC · Table-Scoped Grants"]
        VEND["Credential Vending\nShort-Lived Prefix-Scoped Storage Tokens"]
        META[("Backend Metadata Store\nPostgreSQL / Spanner / FoundationDB\nAtomic CAS Commits")]
    end

    subgraph DataPlane["3. Object Storage (GCS / S3 / ADLS)"]
        OBJ[("Iceberg Warehouse\nmetadata.json · manifests · Parquet")]
    end

    SPK -->|"HTTPS + OAuth2 Bearer"| API
    PY -->|"HTTPS + OAuth2 Bearer"| API
    SDK -->|"HTTPS + X-Iceberg-Access-Delegation"| API
    API -->|"Validate principal & RBAC"| AUTH
    AUTH -->|"Atomic CAS metadata update"| META
    AUTH -.->|"Mint table-scoped token"| VEND
    VEND -.->|"Return vended creds in loadTable"| API
    SPK ==>|"Direct read/write (vended creds)"| OBJ
    PY ==>|"Direct read/write (vended creds)"| OBJ
    SDK ==>|"Direct read/write (vended creds)"| OBJ

    classDef core fill:#2E1065,stroke:#A855F7,stroke-width:2px,color:#F5F3FF
    classDef service fill:#18181B,stroke:#7C3AED,stroke-width:1.5px,color:#F4F4F5
    classDef store fill:#09090B,stroke:#C084FC,stroke-width:1.5px,color:#EDE9FE
    classDef guard fill:#3B0764,stroke:#E879F9,stroke-width:2px,stroke-dasharray:5 3,color:#FAF5FF

    class API core
    class SPK,PY,SDK,VEND service
    class AUTH guard
    class META,OBJ store
    style Clients fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
    style ControlPlane fill:#16141F,stroke:#6D28D9,stroke-width:1.25px,stroke-dasharray:4 3,color:#C4B5FD
    style DataPlane fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
```

- **Single Endpoint, Any Engine:** Non-JVM engines (PyIceberg, DuckDB, Polars, Rust) get first-class catalog access without running a JVM or Thrift bridge.
- **Server-Side Commit Optimization:** Instead of clients downloading/rewriting `metadata.json` blindly on contention, clients send granular **metadata updates + requirements** (`POST /v1/namespaces/{ns}/tables/{table}`), allowing the server to validate assertions cleanly.
- **Credential Vending (`X-Iceberg-Access-Delegation: vended-credentials`):** The catalog checks RBAC per namespace/table and returns ephemeral GCS/S3 credentials scoped to that table's prefix—eliminating broad bucket IAM grants on every worker node.

## Notable Open-Source REST Catalog Servers

| Server | Backing Store | Key Differentiator | Senior Note |
| :--- | :--- | :--- | :--- |
| **Apache Polaris** | PostgreSQL / EclipseLink / In-Memory | Native RBAC (`principal -> role -> catalog role`), credential vending, Snowflake-contributed ASF project | Strongest open-source default for self-hosted multi-engine governance. |
| **Lakekeeper** | PostgreSQL | Lightweight Rust binary, OpenFGA fine-grained authZ, low memory footprint | Fast to deploy on Kubernetes/Cloud Run with minimal ops overhead. |
| **Project Nessie (REST)** | RocksDB / PostgreSQL / DynamoDB | Exposes Iceberg REST API backed by Git-like branching & tagging | Combines standard `RESTCatalog` clients with [[Apache Iceberg Nessie Catalog]] WAP workflows. |
| **Iceberg REST Fixture** | SQLite / In-Memory (`tabulario/iceberg-rest`) | Zero-setup Docker container for local integration tests | Strictly for CI/local testing (never production). |

## Configuration Examples

### PyIceberg (OSS Polaris / Lakekeeper)
```python
from pyiceberg.catalog import load_catalog

catalog = load_catalog(
    "bronze",
    **{
        "type": "rest",
        "uri": "https://polaris.internal/api/catalog",
        "warehouse": "bronze",
        "credential": "<client_id>:<client_secret>",
        "scope": "PRINCIPAL_ROLE:ALL",
        "header.X-Iceberg-Access-Delegation": "vended-credentials",
    },
)
```

### Apache Spark
```bash
spark-sql \
  --conf spark.sql.catalog.bronze=org.apache.iceberg.spark.SparkCatalog \
  --conf spark.sql.catalog.bronze.type=rest \
  --conf spark.sql.catalog.bronze.uri=https://polaris.internal/api/catalog \
  --conf spark.sql.catalog.bronze.warehouse=bronze \
  --conf spark.sql.catalog.bronze.header.X-Iceberg-Access-Delegation=vended-credentials
```

## Related Notes
- **Concepts:** [[Apache Iceberg]] · [[Apache Iceberg Catalogs]]
- **GCP Implementation:** [[GCP BigLake Iceberg REST Catalog]]
- **Alternative OSS Catalogs:** [[Apache Iceberg Hive Metastore Catalog]] · [[Apache Iceberg JDBC Catalog]] · [[Apache Iceberg Nessie Catalog]] · [[Apache Iceberg Hadoop Catalog]]

## When Not to Use Self-Hosted OSS REST Servers
- When your cloud provider already supplies a serverless, IAM-integrated Iceberg REST endpoint (e.g., [[GCP BigLake Iceberg REST Catalog]] or AWS Glue Iceberg REST) and you do not need custom self-hosted multi-cloud RBAC.

## Documentation & References
- [Apache Iceberg REST Catalog OpenAPI Spec](https://github.com/apache/iceberg/blob/main/open-api/rest-catalog-open-api.yaml)
- [Apache Polaris Documentation](https://polaris.apache.org/)
- [Lakekeeper Documentation](https://docs.lakekeeper.io/)
