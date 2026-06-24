# Azure Cosmos DB Setup & Configuration Lab

## Overview

This project documents the hands-on setup and configuration of an **Azure Cosmos DB** account using the NoSQL (Core/SQL) API. It covers global distribution, database and container creation, CRUD operations, throughput management, security controls, and performance monitoring.

---

## Configuration Choices

### 1. API Type
**Azure Cosmos DB for NoSQL (Core/SQL API)**

The NoSQL/SQL API was selected because it supports JSON document storage with a familiar SQL-like query syntax. It is the most widely used Cosmos DB API and best suited for general-purpose cloud applications requiring flexible schema design and low-latency access.

### 2. Consistency Level
**Session Consistency** (Azure Default)

Session consistency was used for this lab. It guarantees that within a single client session, all reads reflect the most recent writes — providing a strong balance between consistency and performance. This is the recommended default for most application workloads as it avoids stale reads without the latency cost of Strong consistency.

| Consistency Level | Trade-off |
|---|---|
| Strong | Highest consistency, highest latency |
| Bounded Staleness | Configurable lag, good for global apps |
| **Session** ✅ | Per-session consistency, best for most apps |
| Consistent Prefix | No out-of-order reads, lower latency |
| Eventual | Lowest latency, possible stale reads |

### 3. Partition Key
**`/category`**

The `/category` field was chosen as the partition key because:
- It has **multiple distinct values** (electronics, clothing, books) enabling even data distribution across physical partitions
- It is **commonly included in queries**, making it an efficient filter for targeted reads
- It supports **horizontal scalability** as the dataset grows
- It avoids hot partitions since data is spread across multiple category values

---

## Deployed Resources

| Resource | Value |
|---|---|
| **Account Name** | cosmoslab-ade |
| **API** | Azure Cosmos DB for NoSQL |
| **Resource Group** | rg-cosmoslab |
| **Primary Region** | Sweden Central |
| **Read Region** | UK South |
| **Capacity Mode** | Provisioned Throughput |
| **Free Tier** | Opted In (1,000 RU/s + 25 GB free) |
| **Consistency Level** | Session |

---

## Tasks Completed

### Task 1 — Account Creation
Created an Azure Cosmos DB account using the NoSQL API in Sweden Central with Provisioned Throughput mode and the Free Tier discount applied.

📸 `cosmos-account-overview.png`

---

### Task 2 — Global Distribution
Configured multi-region replication by adding **UK South** as a read replica to the primary **Sweden Central** write region. Multi-region writes were intentionally left disabled to avoid multiplying RU costs across regions.

**CAP Theorem Trade-off:**
Adding a read region improves availability and reduces read latency for geographically distributed users. However, it increases replication cost and introduces slight write latency due to cross-region synchronization. This is the core Consistency vs. Availability vs. Cost trade-off in distributed database design.

📸 `replication-map.png`

---

### Task 3 — Database and Container Setup
Created a database and container with the following configuration:

| Setting | Value |
|---|---|
| **Database ID** | LabDatabase |
| **Container ID** | LabContainer |
| **Partition Key** | `/category` |
| **Throughput** | 400 RU/s (Manual) |

📸 `database-container.png`

---

### Task 4 — Data Operations (CRUD)
Performed all four CRUD operations via the Azure Portal Data Explorer:

**Create** — Inserted 3 JSON documents:
```json
{ "id": "1", "category": "electronics", "name": "Laptop", "price": 999.99, "inStock": true }
{ "id": "2", "category": "clothing", "name": "Running Shoes", "price": 89.99, "inStock": true }
{ "id": "3", "category": "books", "name": "Cloud Architecture Patterns", "price": 49.99, "inStock": false }
```

**Read** — Queried documents using SQL:
```sql
SELECT * FROM c WHERE c.category = 'electronics'
```

**Update** — Modified Document 1 price from `999.99` to `899.99`

**Delete** — Removed Document 3 (books)

📸 `crud-documents.png`

---

### Task 5 — Throughput Management
Explored both throughput modes available in Azure Cosmos DB:

| Mode | Setting | Use Case |
|---|---|---|
| **Manual** | 400 RU/s (fixed) | Predictable, low-traffic workloads. Best for development. |
| **Autoscale** | 40–400 RU/s (dynamic) | Variable traffic. Scales automatically based on demand. |

The account was returned to **Manual at 400 RU/s** after testing to maintain zero cost under the free tier.

📸 `throughput-scale.png`

---

### Task 6 — Security and Networking

**Encryption:** All data is encrypted at rest and in transit by default using Microsoft-managed keys. No additional configuration was required.

**Access Control:** Both Key-based authentication and Role-Based Access Control (RBAC) are available. Keys were reviewed under Settings → Keys. For production systems, RBAC with managed identities is preferred over sharing primary/secondary keys.

**Networking:** The account is accessible over all networks. Private endpoint configuration was reviewed — in production, routing traffic through a Virtual Network (VNet) via a private endpoint eliminates public internet exposure and is a security best practice.

---

### Task 7 — Monitoring
Configured Azure Metrics to monitor database performance using two key metrics:

| Metric | Observed Value | What It Means |
|---|---|---|
| **Total Requests** | 14 requests | All API operations performed during the lab |
| **Server Side Latency Gateway** | 11.29ms | Server processing time — well within the <10ms target for point reads |

📸 `metrics-dashboard.png`

---

## Screenshots

| # | Screenshot | Description |
|---|---|---|
| 1 | `cosmos-account-overview.png` | Cosmos DB account overview showing Online status |
| 2 | `replication-map.png` | Global replication showing Sweden Central + UK South |
| 3 | `database-container.png` | Data Explorer with LabDatabase and LabContainer |
| 4 | `crud-documents.png` | Items view showing inserted JSON documents |
| 5 | `throughput-scale.png` | Scale pane showing Manual 400 RU/s configuration |
| 6 | `metrics-dashboard.png` | Metrics dashboard with Total Requests and Latency graphs |

---

## Repository Structure

```
azure-cosmosdb-lab-ade/
│
├── README.md
├── cosmos-account-overview.png
├── replication-map.png
├── database-container.png
├── crud-documents.png
├── throughput-scale.png
└── metrics-dashboard.png
```

---

## Key Learnings

- **NoSQL vs Relational:** Cosmos DB's schema-flexible JSON model allows rapid iteration without migrations, unlike traditional RDBMS systems.
- **CAP Theorem in Practice:** Adding a second region demonstrates the real-world trade-off between consistency, availability, and partition tolerance.
- **Request Units (RUs):** RUs abstract compute, memory, and I/O into a single billing unit — understanding RU consumption is critical for cost-efficient database design.
- **Partition Keys:** Choosing an effective partition key is the most important design decision in Cosmos DB — it directly impacts scalability, performance, and cost.
