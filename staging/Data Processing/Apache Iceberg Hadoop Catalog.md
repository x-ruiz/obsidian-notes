---
tags:
  - data-processing/storage/iceberg
aliases:
  - HadoopCatalog
  - Filesystem Iceberg Catalog
source: "Apache Iceberg Hadoop Catalog Documentation"
source-type: documentation
vendor: "Apache Software Foundation"
url: "https://iceberg.apache.org/docs/latest/java-api-quickstart/#creating-a-table"
created: 2026-11-25
---
The **Apache Iceberg Hadoop Catalog (`HadoopCatalog`)** is a server-less filesystem catalog that discovers tables by directory path and tracks the current metadata version in a `version-hint.text` file using filesystem **atomic rename** (`FileSystem.rename`).

> [!WARNING]
> **Never use `HadoopCatalog` in production on object storage (GCS, S3, ADLS).** Object stores do not implement atomic file rename-without-overwrite; concurrent writers will silently overwrite each other's commits and corrupt table metadata. Use `HadoopCatalog` strictly for **local unit tests** or legacy **HDFS** clusters.

## Why It Works on HDFS but Breaks on Object Storage

```mermaid
%%{init: {"theme": "base", "themeCSS": "svg { background-color: #09090B !important; border-radius: 8px; }", "themeVariables": {"darkMode": true, "background": "#09090B", "mainBkg": "#18181B", "textColor": "#F4F4F5", "primaryColor": "#18181B", "primaryTextColor": "#F4F4F5", "primaryBorderColor": "#7C3AED", "lineColor": "#A855F7", "secondaryColor": "#16141F", "tertiaryColor": "#0F0F14", "edgeLabelBackground": "#18181B", "clusterBkg": "#0F0F14", "clusterBorder": "#52525B", "titleColor": "#D8B4FE", "fontFamily": "Inter, ui-sans-serif, system-ui, -apple-system, sans-serif", "fontSize": "13px"}, "flowchart": {"curve": "basis", "nodeSpacing": 42, "rankSpacing": 56, "padding": 14}}}%%
flowchart TB
    subgraph Writers["1. Concurrent Writers (Both Read v1)"]
        W1["Writer A\nHadoopCatalog Client\nStages v2.metadata.json.tmp"]
        W2["Writer B\nHadoopCatalog Client\nStages v2.metadata.json.tmp"]
    end

    subgraph HDFS["2. POSIX / HDFS (Safe Atomic Rename)"]
        HDFS_OP{"FileSystem.rename\nAtomic · Fail if Target Exists"}
        HDFS_OK["Single Winner Commits v2\nUpdates version-hint.text\nLoser Catches Exception & Retries"]
    end

    subgraph ObjStore["3. GCS / S3 Object Storage (UNSAFE)"]
        OBJ_OP{"Non-Atomic Copy + Delete\nOverwrites v2.metadata.json"}
        OBJ_BAD["Last Writer Clobbers v2\nSilent Lost Commit & Corruption\nZero ACID Isolation"]
    end

    W1 -->|"HDFS rename"| HDFS_OP
    W2 -->|"HDFS rename"| HDFS_OP
    HDFS_OP ==>|"Atomic CAS"| HDFS_OK

    W1 -.->|"gs:// or s3a://"| OBJ_OP
    W2 -.->|"gs:// or s3a://"| OBJ_OP
    OBJ_OP -.->|"Race condition"| OBJ_BAD

    classDef core fill:#2E1065,stroke:#A855F7,stroke-width:2px,color:#F5F3FF
    classDef service fill:#18181B,stroke:#7C3AED,stroke-width:1.5px,color:#F4F4F5
    classDef guard fill:#3B0764,stroke:#E879F9,stroke-width:2px,stroke-dasharray:5 3,color:#FAF5FF

    class HDFS_OK core
    class W1,W2,HDFS_OP service
    class OBJ_OP,OBJ_BAD guard
    style Writers fill:#0F0F14,stroke:#52525B,stroke-width:1.5px,color:#D8B4FE
    style HDFS fill:#16141F,stroke:#6D28D9,stroke-width:1.25px,stroke-dasharray:4 3,color:#C4B5FD
    style ObjStore fill:#18181B,stroke:#52525B,stroke-width:1.25px,stroke-dasharray:5 4,color:#A1A1AA
```

- **Storage Layout:** Under `<warehouse>/<namespace>/<table>/metadata/`, `HadoopCatalog` writes sequentially numbered `v1.metadata.json`, `v2.metadata.json`, and updates `version-hint.text` with the latest integer version.
- **No Central Discovery or Governance:** Listing namespaces/tables requires slow filesystem `LIST` operations, and there is no RBAC, audit log, or credential vending.
- **Deprecation Trajectory:** The Apache Iceberg community actively recommends replacing `HadoopCatalog` with [[Apache Iceberg REST Catalog]] (or `InMemoryCatalog` / `JdbcCatalog` backed by SQLite in unit tests).

## Usage Examples (Safe vs. Unsafe)

```java
// GOOD: Local JUnit integration test against a temporary directory
Path tempWarehouse = Files.createTempDirectory("iceberg-test-warehouse");
HadoopCatalog testCatalog = new HadoopCatalog(new Configuration(), tempWarehouse.toUri().toString());
testCatalog.createTable(TableIdentifier.of("football", "matches"), schema, spec);

// BAD: Production HadoopCatalog pointed at GCS or S3 (causes silent lost updates!)
// HadoopCatalog prodCatalog = new HadoopCatalog(conf, "gs://udp-prod-bronze/");
```

## Related Notes
- **Concepts:** [[Apache Iceberg]] · [[Apache Iceberg Catalogs]]
- **Production OSS Replacements:** [[Apache Iceberg REST Catalog]] · [[Apache Iceberg JDBC Catalog]] · [[Apache Iceberg Hive Metastore Catalog]] · [[Apache Iceberg Nessie Catalog]]
- **GCP Production Replacement:** [[GCP BigLake Iceberg REST Catalog]]

## When Not to Use
- Any cloud object store (GCS, S3, ADLS), any multi-writer workload, or any production environment requiring governance and safe ACID concurrency.

## Documentation & References
- [Apache Iceberg Catalogs Overview (HadoopCatalog Warning)](https://iceberg.apache.org/concepts/catalog/)
- [Apache Iceberg Java API Quickstart](https://iceberg.apache.org/docs/latest/java-api-quickstart/)
