# Deep Dive Comparison: Azure Cosmos DB Classic Emulator vs. vNext Linux Emulator (GA)

This document provides a highly detailed, technical comparison between the legacy **Classic Windows Local Emulator (MSI)** and the newly generally available (GA) **vNext Linux-based Emulator (Container)**.

---

## 📊 High-Level Comparison Matrix

| Feature Feature Group | Classic Local Emulator (Windows MSI) | vNext Linux Emulator (GA Container) |
| :--- | :--- | :--- |
| **Host Operating System** | Windows Only | Cross-Platform (Windows, macOS, Linux) |
| **CPU Architecture** | Intel/AMD x64 Only | Native Multi-Arch (**x64 & ARM64**) |
| **Local AI & Vector Search**| ❌ Not Supported | ✅ Natively Supported (DiskANN, Vector Indexes) |
| **Pipeline Automation** | ❌ Complex external scripts / polling | ✅ Declarative initialization via `/init` directory |
| **Production Observability**| ❌ Proprietary Windows logs | ✅ **OpenTelemetry (OTLP)** metrics, traces, & logs |
| **CI/CD Integration** | ❌ Slower boot, log-scraping readiness | ✅ Lightweight, explicit HTTP `/ready` endpoints |
| **Query Engine Parity** | ⚠️ Partial (Legacy limitations on nested arrays) | ✅ High Fidelity (Supports Hierarchical PKs, complex JOINs) |

---

## 💡 Detailed Feature Breakdown with Examples

### 1. Local AI & Vector Search (RAG Applications)
The vNext Linux Emulator allows you to build and test Generative AI or Retrieval-Augmented Generation (RAG) applications directly on your local machine using vector embeddings. 

* **Classic Windows Emulator:** Fails with a `400 Bad Request` or unsupported syntax error when executing vector embeddings or indexing policies.
* **vNext Linux Emulator:** Fully supports the `vectorEmbeddingPolicy` and `indexingPolicy` for vector search models.

#### 📝 Code Example: Creating a Vector-Enabled Container (Node.js/Python SDK)
When deploying a container definition, you can pass vector definitions natively to your local vNext emulator endpoint:

```json
{
  "id": "VectorContainer",
  "partitionKey": { "paths": ["/id"], "kind": "Hash" },
  "vectorEmbeddingPolicy": {
    "vectorEmbeddings": [
      {
        "path": "/vector",
        "dataType": "float32",
        "distanceFunction": "cosine",
        "dimensions": 1536
      }
    ]
  },
  "indexingPolicy": {
    "vectorIndexes": [
      { "path": "/vector", "type": "diskANN" }
    ]
  }
}
```

---

### 2. Native Database Bootstrapping & Seeding (`/init`)
Populating seed data or setting up environments for automatic integration tests behaves completely differently across versions.

* **Classic Windows Emulator:** Requires you to write complex code routines in your application startup that wait, poll port `8081`, check if the endpoint is up, and then imperatively execute `.createDatabaseIfNotExists()` commands.
* **vNext Linux Emulator:** Uses a declarative directory structure. Any `.csh` script placed inside the container's `/init` folder executes automatically as soon as the service engine starts up.

#### 📝 Code Example: Declarative `init.csh` Script
```bash
# Inside /init/setup-db.csh
connect --endpoint https://localhost:8081 --key "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqOBD4b8mGGQ== "

create database --id CompanyDB
create container --id CompanyDB --container Employees --partition-key /department
```

---

### 3. OpenTelemetry (OTLP) Monitoring & Observability
vNext introduces cloud-native telemetry tracking to identify slow queries directly inside your local development terminal.

* **Classic Windows Emulator:** Relies on Windows Event Viewer or arbitrary text log files stored under local app-data directories.
* **vNext Linux Emulator:** Integrates natively with the OpenTelemetry collector specification, allowing you to feed telemetry data instantly into tools like Jaeger, Zipkin, or Prometheus.

#### 📝 Execution Example: Spin up with OTLP telemetry enabled
```bash
docker run --name cosmos-vnext \
  -p 8081:8081 -p 4317:4317 \
  -e "ENABLE_OTLP=true" \
  -e "OTLP_ENDPOINT=http://host.docker.internal:4317" \
  -d mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:vnext-latest
```

---

### 4. CI/CD Pipeline & Testcontainers Readiness Probes
The lightweight containerized design of vNext makes local environment or automated GitHub Actions workflows incredibly fast.

* **Classic Windows Emulator:** Booting up can take up to several minutes because it spins up heavy background Windows worker routines. Pipelines have to loop or rely on arbitrary delays.
* **vNext Linux Emulator:** Exposes specific lightweight, non-authenticated network routes (`/alive` and `/ready`) on port `8080` for immediate integration with orchestration layers and testing libraries.

#### 📝 Code Example: Testcontainers Integration (Java Example)
```java
public class CosmosTest {
    @Container
    private static final GenericContainer<?> cosmosContainer =
        new GenericContainer<>(DockerImageName.parse("mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:vnext-latest"))
            .withExposedPorts(8081, 8080)
            .waitingFor(Wait.forHttp("/ready").forPort(8080)); // Lightning-fast native readiness
}
```

---

## 🛠 Direct Connectivity String Configurations

Both emulators use identical default primary master keys, meaning connection string swaps are effortless.

```text
AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqOBD4b8mGGQ==;
```

* **Note:** For the vNext container architecture, you must operate your client application SDK explicitly in **Gateway Mode** (HTTP port `8081`), as direct TCP routing paths behave differently inside isolated container virtual networks.
