# Architecture Guild --- From Fast APIs to Intelligent Data

## .NET 10 + AI-Powered Azure Cosmos DB

**Duration:** \~40 minutes\
**Audience:** Architects, Senior Developers, Tech Leads\
**Theme:** Improving enterprise API performance while making operational
data AI-ready

------------------------------------------------------------------------

# Slide 1 --- Today's Technology Radar

## Two connected architecture trends

### Part 1 --- .NET 10 for Modern APIs

-   Why .NET 10 matters
-   API performance and efficiency
-   ASP.NET Core improvements
-   JSON and OpenAPI improvements
-   Migration considerations

### Part 2 --- AI Capabilities with Azure Cosmos DB

-   Vector search
-   Embeddings with operational data
-   Semantic and hybrid search
-   RAG
-   AI agent memory
-   Practical enterprise use cases

### Speaker Notes

Today's session connects application performance and AI. We will look at
.NET 10 as a modernization opportunity for enterprise APIs, then explore
how Azure Cosmos DB vector capabilities can enable semantic search and
AI-powered experiences.

------------------------------------------------------------------------

# Slide 2 --- Why .NET 10 Matters for Enterprise APIs

.NET 10 is an **LTS release**.

Key questions: - What improves performance? - What simplifies API
development? - What improves observability? - What changes API contracts
and tooling? - What is worth adopting during migration from .NET 8?

``` text
.NET 8 API
    ↓
Evaluate .NET 10
    ↓
Benchmark
    ↓
Compatibility Testing
    ↓
Controlled Migration
```

### Speaker Notes

A new .NET version does not mean every application needs an immediate
upgrade. Architects should identify capabilities that provide measurable
value and evaluate support lifecycle, dependencies, performance,
compatibility, and operational risk.

------------------------------------------------------------------------

# Slide 3 --- API Performance Is More Than Framework Speed

``` text
Client → Network → API Gateway → Authentication → ASP.NET Core
       → Serialization → Business Logic → Database → External Services
```

Common bottlenecks: - Excessive allocations - Large payloads - Slow
serialization - Blocking I/O - Sequential external calls - Database
query inefficiency - Connection exhaustion - Missing caching - Excessive
logging

> Upgrading the runtime helps, but architecture determines end-to-end
> performance.

### Speaker Notes

If an API spends 500 milliseconds waiting for a database query,
framework improvements alone will not solve the main problem.
Performance work should begin with measurement.

------------------------------------------------------------------------

# Slide 4 --- .NET 10: Performance Areas to Evaluate

-   Runtime and JIT improvements
-   Reduced allocations
-   ASP.NET Core request processing
-   `System.Text.Json`
-   JSON Patch improvements
-   Minimal API scenarios
-   Native AOT suitability
-   HTTP client behavior
-   OpenAPI tooling

``` text
.NET 8 Baseline
      ↓
Same Workload on .NET 10
      ↓
Measure CPU | Memory | Latency | Throughput
      ↓
Architecture Decision
```

### Speaker Notes

Benchmark your own workloads. Compare p50, p95 and p99 latency,
throughput, CPU, memory and garbage collection behavior.

------------------------------------------------------------------------

# Slide 5 --- JSON Performance Matters

``` text
Request → JSON Deserialization → Business Logic → JSON Serialization → Response
```

Performance considerations: - Prefer `System.Text.Json` for standard
scenarios - Avoid unnecessarily large DTOs - Use pagination - Avoid
returning entire domain models - Consider source generation where
appropriate - Stream large responses where appropriate

``` csharp
[JsonSerializable(typeof(CustomerResponse))]
[JsonSerializable(typeof(List<CustomerResponse>))]
internal partial class AppJsonSerializerContext : JsonSerializerContext
{
}
```

### Speaker Notes

Serialization can become significant in high-throughput APIs and
large-payload scenarios. Measure before optimizing.

------------------------------------------------------------------------

# Slide 6 --- Practical API Performance Architecture

``` csharp
var customerTask = customerService.GetCustomerAsync(id);
var ordersTask = orderService.GetOrdersAsync(id);
await Task.WhenAll(customerTask, ordersTask);
```

Important patterns: - Async I/O - Connection pooling - Pagination -
Response compression - Output caching - Rate limiting - Request
timeouts - Resilience policies - Efficient database queries

### Speaker Notes

Architecture-level improvements often produce larger gains than
micro-optimizations. Independent downstream calls, for example, may be
executed concurrently where safe.

------------------------------------------------------------------------

# Slide 7 --- OpenAPI and API Governance

Architecture opportunities: - Standard API contracts - Automated
documentation - Client generation - Contract validation -
Breaking-change detection - API governance

``` text
API Implementation → OpenAPI Contract → Validation → Client Generation → Consumers
```

> Can API contracts become enforceable assets in our CI/CD pipeline?

### Speaker Notes

OpenAPI should be treated as more than Swagger UI. Specifications can
support governance and breaking-change detection.

------------------------------------------------------------------------

# Slide 8 --- Moving from Fast APIs to Intelligent APIs

Traditional:

``` text
User → React → .NET API → Cosmos DB → Exact Data Match
```

AI-enabled:

``` text
User Intent → .NET API → Embedding Model → Vector + Structured Search
            → Cosmos DB → Semantically Relevant Results
```

From: \> Find records where field X equals value Y.

To: \> Find records that mean something similar to this request.

------------------------------------------------------------------------

# Slide 9 --- What Is Vector Search?

Traditional search:

``` text
Status = "Pending"
TaxYear = 2025
Country = "US"
```

Vector search compares meaning.

``` text
Text
 ↓
Embedding Model
 ↓
[0.12, -0.42, 0.87, ...]
 ↓
Vector Search
 ↓
Semantically Similar Records
```

### Speaker Notes

Embeddings are numerical representations of content. Vector search
retrieves records based on semantic similarity rather than only exact
keywords.

------------------------------------------------------------------------

# Slide 10 --- Azure Cosmos DB as an AI-Ready Data Store

``` text
Application
    ↓
Cosmos DB
    ├── Operational Data
    └── Vector Embeddings
```

Potential benefits: - Keep vectors near operational data - Reduce
separate synchronization - Apply metadata filters - Combine structured
and semantic search - Support RAG scenarios - Support AI application
memory patterns

### Speaker Notes

Integrated vector capabilities may reduce the need for a separate vector
store in some architectures, although the right choice depends on
requirements.

------------------------------------------------------------------------

# Slide 11 --- Vector Index Choices

Common concepts: - Flat - Quantized Flat - DiskANN

Architecture considerations: - Dataset size - Query latency - Recall
requirements - RU consumption - Partitioning - Vector dimensions -
Update frequency

### Speaker Notes

There is no universally best index. Test with representative data and
workload.

------------------------------------------------------------------------

# Slide 12 --- Practical Use Case: Semantic Assignment Search

Traditional:

``` text
TaxYear = 2025 AND Jurisdiction = US AND Status = Pending
```

Semantic:

> Find assignments similar to cases where customers had missing tax
> documents and delayed filing.

``` text
React UI
   ↓
.NET 10 API
   ↓
Embedding Model
   ↓
Query Vector
   ↓
Cosmos DB Vector Search
   ↓
Relevant Assignments
```

------------------------------------------------------------------------

# Slide 13 --- Hybrid Search: Best of Both Worlds

``` text
Structured Filters + Vector Similarity → Hybrid Search
```

Example:

> Find US assignments from tax year 2025 that are semantically similar
> to cases involving missing foreign-income documentation.

Structured:

``` text
Jurisdiction = US
TaxYear = 2025
```

Semantic:

``` text
Similar to "Missing foreign-income documentation"
```

### Speaker Notes

Enterprise search often needs structured filtering and semantic
similarity together.

------------------------------------------------------------------------

# Slide 14 --- RAG with Operational Data

``` text
User Question
     ↓
Create Embedding
     ↓
Search Cosmos DB
     ↓
Retrieve Relevant Context
     ↓
LLM
     ↓
Grounded Response
```

Potential use cases: - Case summarization - Assignment assistance -
Knowledge retrieval - Similar-case discovery - Document-based Q&A

> Retrieval provides context. The LLM generates the response.

------------------------------------------------------------------------

# Slide 15 --- AI Agent Memory with Cosmos DB

Potential roles: - Conversation history - Agent state - User/session
context - Embeddings - Retrieval context

Architecture considerations: - Tenant isolation - Data retention - PII
handling - Authorization - Token cost - Memory summarization

### Speaker Notes

Agent memory requires governance. Architects must decide what is
retained, for how long, and who can access it.

------------------------------------------------------------------------

# Slide 16 --- Proposed POC: AI-Powered Assignment Discovery

``` text
Existing Data → Generate Embeddings → Store in Cosmos DB
```

Then:

``` text
Natural Language Query → Embedding → Vector Search → Matching Assignments
```

Then:

``` text
Metadata Filters + Vector Search → Hybrid Search
```

Success metrics: - Search relevance - Query latency - RU consumption -
Index size - User usefulness - Security correctness

------------------------------------------------------------------------

# Slide 17 --- Technology Radar

  Capability                     Position         Why
  ------------------------------ ---------------- --------------------------------------
  .NET 10                        ASSESS / TRIAL   Next LTS modernization path
  API Performance Benchmarking   ADOPT            Evidence-based optimization
  OpenAPI Governance             ADOPT / TRIAL    API lifecycle value
  Cosmos DB Vector Search        TRIAL            AI-search potential
  Hybrid Search                  ASSESS / TRIAL   Enterprise search pattern
  RAG over Operational Data      ASSESS           Requires governance
  AI Agent Memory                ASSESS           Privacy and retention considerations

------------------------------------------------------------------------

# Slide 18 --- Final Takeaway

``` text
.NET 10 → Faster and Modern APIs
Cosmos DB → Operational + Vector Data
AI → Semantic Understanding
Combined → Fast + Intelligent Applications
```

> The goal is not to add AI to every application. The goal is to
> identify where semantic understanding solves problems that
> deterministic queries cannot solve efficiently.

------------------------------------------------------------------------

# Suggested 40-Minute Timing

  Section                            Duration
  -------------------------------- ----------
  Introduction                          2 min
  .NET 10 and API Performance          12 min
  API Architecture and OpenAPI          4 min
  Cosmos DB AI and Vector Search       10 min
  Semantic / Hybrid Search              5 min
  RAG and Agent Memory                  3 min
  POC, Radar and Closing                4 min

------------------------------------------------------------------------

# Suggested Demo

1.  Prepare sample assignments or support cases.
2.  Generate embeddings for a description field.
3.  Store embeddings with records in Cosmos DB.
4.  Search using natural language.
5.  Show semantically similar records.
6.  Add a structured filter such as year or jurisdiction.
7.  Compare keyword search with semantic search.

------------------------------------------------------------------------

# Useful References

-   .NET 10:
    https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview
-   ASP.NET Core 10:
    https://learn.microsoft.com/en-us/aspnet/core/release-notes/aspnetcore-10.0
-   Cosmos DB Vector Search:
    https://learn.microsoft.com/en-us/azure/cosmos-db/vector-search
-   Cosmos DB Generative AI:
    https://learn.microsoft.com/en-us/azure/cosmos-db/gen-ai/

------------------------------------------------------------------------

# Optional Discussion Questions

1.  Which APIs would benefit most from .NET 10 benchmarking?
2.  Where do we have complex search experiences today?
3.  Could semantic search improve assignment or case discovery?
4.  Should vectors live with operational data or separately?
5.  How would tenant authorization work during vector retrieval?
6.  What metrics would determine whether the POC succeeds?
