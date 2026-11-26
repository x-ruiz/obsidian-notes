---
tags:
  - data-processing/storage/iceberg
aliases:
  - JdbcCatalog
  - Iceberg JDBC Catalog
source: "Apache Iceberg JDBC Catalog Documentation"
source-type: documentation
vendor: "Apache Software Foundation"
url: "https://iceberg.apache.org/docs/latest/jdbc/"
created: 2026-11-25
---
The **Apache Iceberg JDBC Catalog (`JdbcCatalog`)** uses any JDBC-compliant relational database (PostgreSQL, MySQL, MariaDB, SQLite) directly as the catalog store, executing **ACID [[SQL]] conditional updates** on a single metadata table (`iceberg_tables`) to perform atomic Compare-and-Swap (CAS) commits without deploying a separate metastore daemon.

## Architecture & SQL CAS Mechanics

```mermaid
%%{init: {"theme": "base", "themeCSS": "svg { background-color: #09090B !important; border-radius: 8px; }", "themeVariables": {"darkMode": true, "background": "#09090B", "mainBkg": "#18181B", "textColor": "#F4F4F5", "primaryColor": "#18181B", "primaryTextColor": "#F4F4F5", "primaryBorderColor": "#7C3AED", "lineColor": "#A855F7", "secondaryColor": "#16141F", "tertiaryColor": "#0F0F14", "edgeLabelBackground": "#18181B", "clusterBkg": "#0F0F14", "clusterBorder": "#52525B", "titleColor": "#D8B4FE", "fontFamily": "Inter, ui-sans-serif, system-ui, -apple-system, sans-serif", "fontSize": "13px"}, "flowchart": {"curve": "basis", "nodeSpacing": 42, "rankSpacing": 56, "padding": 14}}}%%
flowchart TB
    subgraph Clients["1. Direct Compute Clients (No Catalog Daemon)"]
        SPK["Apache Spark / Flink\nJdbcCatalog + JDBC Driver\nDirect DB Creds + Bucket IAM"]
        PY["PyIceberg\nSqlCatalog · SQLAlchemy / psycopg\nDirect DB Creds + Bucket IAM"]
    end

    subgraph DBPlane["2. Relational Database (SQL CAS Store)"]
        PG[("PostgreSQL / MySQL\niceberg_tables · namespace_properties\n0 Rows Affected -> CommitFailedException")]
    end

    subgraph Storage["3. Object Storage (GCS / S3)"]
        OBJ[("Iceberg Warehouse\nImmutable Object Store\nvN.metadata.json · Parquet")]
    end

    SPK -->|"JDBC ACID CAS\nUPDATE ... WHERE loc = v1"| PG
    PY -->|"SQL Lookup\nSELECT metadata_location"| PG
    SPK ==>|"Direct Storage IAM\nStage Parquet + v2 JSON"| OBJ
    PY ==>|"Direct Storage IAM\nRead / Write Parquet"| OBJ

    classDef core fill:#2E1065,stroke:#A855F7,stroke-width:2px,color:#F5F3FF
    classDef service fill:#18181B,stroke:#7C3AED,stroke-width:1.5px,color:#F4F4F5
    classDef store fill:#09090B,stroke:#C084FC,stroke-width:1.5px,color:#EDE9FE

    class PG core
    class SPK,PY service
    class OBJ store
    style Clients fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
    style DBPlane fill:#16141F,stroke:#6D28D9,stroke-width:1.25px,stroke-dasharray:4 3,color:#C4B5FD
    style Storage fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
```

- **Two-Table Schema:** On first connect (`jdbc.schema-version=V1`), Iceberg bootstraps `iceberg_tables` (`catalog_name`, `table_namespace`, `table_name`, `metadata_location`, `previous_metadata_location`, `iceberg_type`) and `iceberg_namespace_properties`.
- **Atomic CAS via SQL:** Commits run an optimistic `UPDATE iceberg_tables SET metadata_location = ?, previous_metadata_location = ? WHERE catalog_name = ? AND table_namespace = ? AND table_name = ? AND metadata_location = ?`. If `rows_affected == 0`, the client raises `CommitFailedException` and retries.
- **No Credential Vending or Central AuthZ:** Every engine connects directly to the RDBMS (requiring DB credentials and VPC connectivity) AND directly to object storage.

## Configuration Examples

### Apache Spark (PostgreSQL)
```bash
spark-sql \
  --conf spark.sql.catalog.bronze=org.apache.iceberg.spark.SparkCatalog \
  --conf spark.sql.catalog.bronze.catalog-impl=org.apache.iceberg.jdbc.JdbcCatalog \
  --conf spark.sql.catalog.bronze.uri=jdbc:postgresql://pg.internal:5432/iceberg \
  --conf spark.sql.catalog.bronze.jdbc.user=iceberg_writer \
  --conf spark.sql.catalog.bronze.jdbc.password="${ICEBERG_DB_PASSWORD}" \
  --conf spark.sql.catalog.bronze.jdbc.schema-version=V1 \
  --conf spark.sql.catalog.bronze.warehouse=gs://my-lakehouse-bronze/
```

### PyIceberg (`SqlCatalog` Equivalent)
```python
from pyiceberg.catalog.sql import SqlCatalog

catalog = SqlCatalog(
    "bronze",
    **{
        "uri": "postgresql+psycopg2://iceberg_writer:secret@pg.internal:5432/iceberg",
        "warehouse": "gs://my-lakehouse-bronze/",
    },
)
```

## Trade-Offs

| Dimension | Behavior | Senior Note |
| :--- | :--- | :--- |
| **Simplicity** | Zero extra catalog microservices to operate | Great for single-team prototypes or small self-contained platforms already running PostgreSQL. |
| **Security Posture** | Every compute worker needs RDBMS credentials + bucket IAM | Difficult to enforce table-level RBAC or audit access across multiple teams. |
| **Connection Scaling** | Direct DB connections per driver/client | Large fleets of serverless jobs can exhaust RDBMS connection pools without PgBouncer or an [[Apache Iceberg REST Catalog]] facade. |

## Related Notes
- **Concepts:** [[Apache Iceberg]] · [[Apache Iceberg Catalogs]] · [[SQL]]
- **Alternative OSS Catalogs:** [[Apache Iceberg REST Catalog]] · [[Apache Iceberg Hive Metastore Catalog]] · [[Apache Iceberg Nessie Catalog]] · [[Apache Iceberg Hadoop Catalog]]
- **GCP Serverless Alternative:** [[GCP BigLake Iceberg REST Catalog]] (eliminates Cloud SQL credential distribution and connection pooling)

## When Not to Use
- Multi-team / Data Mesh environments requiring fine-grained RBAC and credential vending (put [[Apache Iceberg REST Catalog]] such as Polaris or [[GCP BigLake Iceberg REST Catalog]] in front of the metadata store instead).

## Documentation & References
- [Apache Iceberg JDBC Catalog Documentation](https://iceberg.apache.org/docs/latest/jdbc/)
- [PyIceberg SQL Catalog Configuration](https://py.iceberg.apache.org/configuration/#sql-catalog)
