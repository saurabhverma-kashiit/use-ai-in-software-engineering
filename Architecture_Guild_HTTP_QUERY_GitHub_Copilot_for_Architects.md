# Architecture Guild | Technology Radar

## HTTP QUERY + GitHub Copilot for Architects

**Duration:** ~40 minutes  
**Audience:** Architects, Senior Developers, and Tech Leads  
**Theme:** Emerging technologies and practices shaping modern enterprise architecture

------------------------------------------------------------------------

# Slide 1 --- Today's Technology Radar

## Two trends worth watching

**A short framing slide:** one protocol shift and one AI shift.

### Part 1 --- HTTP `QUERY` Method

-   Why do we need another HTTP method?
-   GET vs POST vs QUERY
-   Architecture implications
-   Should we use it today?

### Part 2 --- GitHub Copilot for Architects

-   Moving beyond code completion
-   Architecture-aware code review
-   Repository custom instructions
-   Architecture governance with AI
-   Practical use cases

### Speaker Notes

Today I want to cover two developments that are worth watching closely.

The first is the HTTP QUERY method, which introduces a new way to express
complex read operations in a more semantically precise way. The second is
GitHub Copilot for Architects, which is moving beyond code completion into
repository understanding, code review, and architecture-aware assistance.

The goal is not to recommend immediate adoption of everything we discuss.
The goal is to understand these shifts early and identify where they may be
worth evaluating in our own environments.

------------------------------------------------------------------------

# Slide 2 --- Why QUERY Matters

## Simple queries work well with GET

``` http
GET /api/customers?country=US&status=Active
```

But enterprise applications often require more complex queries:

``` json
{
  "countries": ["US", "UK", "IN"],
  "status": ["Active", "Pending"],
  "createdAfter": "2025-01-01",
  "filters": {
    "riskScore": {
      "min": 50,
      "max": 90
    }
  }
}
```

### The question

**Where should this complex read operation live?**

### Speaker Notes

GET works well when query criteria can be represented cleanly in a URI.

But enterprise applications often require richer search patterns, including
nested filters, arrays, ranges, sorting, pagination, and large payloads.
In practice, teams often fall back to a POST endpoint such as
`/customers/search`.

That approach works technically, but it does not communicate the same safe
and idempotent semantics as a query operation. This is the gap the HTTP
QUERY method aims to address.

------------------------------------------------------------------------

# Slide 3 --- What QUERY Is

## HTTP `QUERY`

Published as **RFC 10008 --- June 2026**

Purpose:

> Send query content to a resource in a safe and idempotent manner.

Conceptually:

``` http
QUERY /api/customers
Content-Type: application/json
```

``` json
{
  "country": "US",
  "status": "Active",
  "minimumRevenue": 1000000
}
```

### Semantic intent

``` text
GET    → Retrieve a resource representation

POST   → Submit data or perform processing
         that may change state

QUERY  → Perform a query using request content
         without changing server state
```

### Key Characteristics

-   Safe
-   Idempotent
-   Supports request content
-   Explicitly communicates query intent

### Reference

RFC 10008 --- The HTTP QUERY Method\
https://www.rfc-editor.org/rfc/rfc10008.html

### Speaker Notes

The important point is not simply that QUERY can carry request content.

The architectural difference is its semantics.

QUERY is defined as a safe and idempotent method. This means the request
is intended to retrieve information without requesting a change to the
server's state.

That gives infrastructure and developers clearer information about the
nature of the operation compared with using POST for complex searches.

------------------------------------------------------------------------

# Slide 4 --- Why It Matters Architecturally

  Characteristic                 GET                    POST     QUERY
  ------------------------------ ---------------------- -------- -------
  Suitable for read operations   Yes                    Can be   Yes
  Safe                           Yes                    No       Yes
  Idempotent                     Yes                    No       Yes
  Request content for query      Generally unsuitable   Yes      Yes
  Complex query payload          Difficult              Yes      Yes
  Explicit query semantics       Limited                No       Yes

### Key Point

The difference is not only syntax.

**The difference is semantic intent.**

### Speaker Notes

If I send a complex search through POST, the application understands the
intent because of the endpoint name and implementation.

But generic HTTP infrastructure sees only a POST request.

With QUERY, the protocol itself communicates that the operation is a query
and is safe and idempotent. That may eventually help infrastructure make
better decisions around retries, routing, and other HTTP behaviors,
although real support will depend on the ecosystem.

------------------------------------------------------------------------

# Slide 5 --- A Concrete Enterprise Example

Imagine a system needs to find assignments where:

``` text
Jurisdiction = US
Tax Year = 2025
Status = Pending
Risk Score > 70
Documents > 5
Created Date between X and Y
```

## Option 1 --- GET

``` http
GET /assignments?
    jurisdiction=US
    &year=2025
    &status=Pending
    &riskScoreMin=70
    &minimumDocuments=5
    &createdFrom=...
    &createdTo=...
```

### Potential challenges

-   Long URLs
-   Complex nested filters
-   URL-length constraints
-   Query information may appear in logs and browser history
-   Difficult representation of complex structures

------------------------------------------------------------------------

## Option 2 --- POST

``` http
POST /assignments/search
```

``` json
{
  "jurisdiction": "US",
  "taxYear": 2025,
  "status": "Pending",
  "riskScore": {
    "greaterThan": 70
  }
}
```

Technically effective, but:

``` text
POST ≠ inherently safe
POST ≠ inherently idempotent
```

------------------------------------------------------------------------

## Option 3 --- QUERY

``` http
QUERY /assignments
```

``` json
{
  "jurisdiction": "US",
  "taxYear": 2025,
  "status": "Pending",
  "riskScore": {
    "greaterThan": 70
  }
}
```

Semantic meaning:

> Query this resource using these criteria without requesting a state
> change.

### Speaker Notes

This is where QUERY becomes interesting for enterprise systems.

Many applications already use POST for search-style operations through
endpoints such as `/search`, `/query`, or `/filter`.

QUERY could eventually provide a standardized HTTP semantic for these
scenarios. However, that does not mean existing POST-based search APIs are
wrong or need to be rewritten immediately.

------------------------------------------------------------------------

# Slide 6 --- Architecture Implications of HTTP QUERY

Before adopting QUERY, architects need to evaluate the complete request
path.

## API Frameworks

-   ASP.NET Core
-   Node.js
-   Other backend frameworks
-   HTTP client libraries

## Frontend and Clients

-   Browser APIs
-   React applications
-   Mobile clients
-   SDKs

## Infrastructure

-   Azure API Management
-   Application Gateway
-   Web Application Firewall
-   Reverse proxies
-   Load balancers

## Observability

-   Application Insights
-   Logging platforms
-   Distributed tracing
-   OpenTelemetry

## Other Considerations

-   Caching behavior
-   Authentication and authorization
-   Request-body inspection
-   API documentation
-   OpenAPI tooling

### Speaker Notes

A method being standardized does not mean we should immediately use it
in production.

Our HTTP requests travel through multiple infrastructure layers.

For example:

Client → Front Door → WAF → API Management → App Service → ASP.NET Core
API.

Every component needs to correctly handle the HTTP method.

We also need to consider development tools, OpenAPI support, monitoring,
security policies, and client libraries.

Standardization and ecosystem adoption are two different things.

------------------------------------------------------------------------

# Slide 7 --- Should We Use HTTP QUERY Today?

## Suggested Technology Radar Position

### ASSESS

``` text
ADOPT    ❌

TRIAL    ?

ASSESS   ✅

HOLD
```

### Why Assess?

-   New standardized HTTP capability
-   Strong semantics for complex query operations
-   Potential enterprise API use cases
-   Ecosystem support requires validation
-   Existing GET and POST approaches remain practical

### Recommendation

**Understand it now.**

**Experiment in a controlled POC.**

**Do not redesign existing APIs solely because QUERY exists.**

### Speaker Notes

My recommendation would be to place QUERY in the Assess category.

As architects, we should understand the standard and identify potential
use cases.

A small proof of concept could test support across our actual technology
stack.

For example:

React → Azure Front Door → APIM → App Service → ASP.NET Core.

If every layer works correctly, we can then evaluate whether it provides
enough value for future APIs.

------------------------------------------------------------------------

# Slide 8 --- Evidence and Benchmarks for HTTP QUERY

## What evidence exists today?

-   There is not yet a widely published production benchmark showing that
    HTTP QUERY is faster than GET or POST for real-world workloads.
-   The strongest evidence today is semantic and architectural rather than
    performance-based.
-   RFC 10008 defines QUERY as a safe and idempotent method, which makes
    the intent of the request clearer for clients, gateways, and APIs.

## Practical benefit framing

Instead of claiming that QUERY is "faster", the better message is:

-   Better API intent and semantics
-   Fewer custom POST-based search endpoints
-   Cleaner long-term contract design for complex read operations
-   Improved consistency for retries, caching discussions, and observability

## Suggested wording for the talk

> There is no mature public benchmark yet proving a performance win for
> QUERY over POST. The value proposition is currently stronger in
> semantics, contract clarity, and API design than in raw speed.

### Reference

-   RFC 10008 --- The HTTP QUERY Method\
    https://www.rfc-editor.org/rfc/rfc10008.html
-   RFC 9110 --- HTTP Semantics\
    https://www.rfc-editor.org/rfc/rfc9110.html

### Speaker Notes

This is an important point to make clearly.

The HTTP QUERY method is still emerging, and there is not yet a large body
of public benchmark data proving measurable performance gains.

That is why the strongest value case today is architectural and semantic.

For architects, the question is not "Is QUERY faster?" but rather "Does it
help us express the intent of complex read operations more clearly and more
consistently across our platform?"

------------------------------------------------------------------------

# Slide 8 --- GitHub Copilot: From Code Assistant to Architecture Assistant

## The Evolution

``` text
Autocomplete
     ↓
Chat
     ↓
Code Generation
     ↓
Repository Understanding
     ↓
Agentic Workflows
     ↓
Code Review
     ↓
Architecture-Aware Assistance
```

## The Architecture Question

> Can AI help us enforce architecture --- not just write code?

### Speaker Notes

Most developers originally experienced GitHub Copilot as an autocomplete
tool.

You start typing code and Copilot suggests the next few lines.

But AI-assisted development is moving beyond that model.

Modern Copilot capabilities can work with broader repository context,
assist with code review, follow repository-specific instructions, and
participate in agentic workflows.

This creates an interesting question for architects.

Can the same technology that helps developers write code also help teams
follow architectural standards?

------------------------------------------------------------------------

# Slide 9 --- Use Case 1: Understand a Large Codebase

An architect joining an existing system might ask:

``` text
Analyze this solution and identify:

1. Service boundaries
2. Dependencies between services
3. External integrations
4. Data access patterns
5. Messaging patterns
6. Potential tight coupling
```

Instead of manually exploring:

``` text
100 Projects
      ↓
5,000 Files
      ↓
Multiple Microservices
      ↓
Azure Functions
      ↓
Service Bus
      ↓
Cosmos DB
```

AI can assist with initial exploration.

## Important Principle

``` text
AI Analysis
      ↓
Architect Validation
      ↓
Architecture Decision
```

Not:

``` text
AI Analysis
      ↓
Automatic Architecture Decision
```

### Speaker Notes

One practical use case for architects is codebase discovery.

When an architect joins a mature project, understanding the architecture
can take significant time.

Copilot can help identify dependencies, integration points, data access
patterns, and service boundaries.

However, AI-generated analysis should be treated as an accelerator.

The architect still needs to validate the findings and make the final
architectural decisions.

------------------------------------------------------------------------

# Slide 10 --- Use Case 2: Architecture Review

Imagine giving Copilot architecture rules:

``` text
Architecture Rules

- API layer must not access Cosmos DB directly.
- Domain layer must not depend on Infrastructure.
- Cross-service asynchronous communication should use Service Bus.
- Secrets must come from Key Vault.
- External calls must implement resilience policies.
- APIs must propagate Correlation IDs.
```

Then ask:

``` text
Review this implementation against
our architecture standards.
```

## Potential Checks

Can AI help identify:

-   Layer violations?
-   Tight coupling?
-   Missing resilience?
-   Incorrect dependency direction?
-   Security concerns?
-   Missing observability?
-   Missing tests?

### Speaker Notes

This is where Copilot becomes particularly interesting from an
architecture perspective.

Most organizations already have architecture principles and engineering
standards.

The challenge is that these standards often live in documents.

Developers need to find them, read them, remember them, and apply them.

AI-assisted review creates the possibility of bringing some of these
guidelines closer to the development workflow.

------------------------------------------------------------------------

# Slide 11 --- Repository Custom Instructions

One mechanism for providing repository-level context:

``` text
.github/
    copilot-instructions.md
```

Example:

``` markdown
# Architecture Guidelines

- Follow Clean Architecture.
- Domain projects must not reference Infrastructure.
- Use Azure Service Bus for asynchronous integration.
- Use Managed Identity for Azure resources.
- Do not store secrets in configuration files.
- Use structured logging.
- Propagate correlation IDs across services.
```

Other mechanisms can include:

``` text
AGENTS.md

.github/instructions/
    *.instructions.md
```

## Architecture Opportunity

Turn human-readable architecture standards into instructions available
within AI-assisted development workflows.

### Reference

GitHub Copilot documentation\
https://docs.github.com/en/copilot

### Speaker Notes

This is one of the areas I find most relevant for architects.

Instead of architecture standards existing only in Confluence or a Wiki,
some development guidance can be placed closer to the source code.

Repository instructions can provide Copilot with context about the
project's standards and conventions.

This does not replace architecture documentation, ADRs, or governance.

But it can make selected engineering rules more accessible during
development and review.

------------------------------------------------------------------------

# Slide 12 --- Architecture as Code?

## Traditional Model

``` text
Architecture Guidelines
        ↓
Confluence / Wiki
        ↓
Developers Read Them
        ↓
Implementation
        ↓
Code Review
        ↓
Architect Finds Violations
```

## Emerging Model

``` text
Architecture Guidelines
        ↓
Repository Instructions
        ↓
AI-Assisted Development
        ↓
AI-Assisted PR Review
        ↓
Automated Checks
        ↓
Human Review
```

## Discussion

> Could architecture guidelines become part of the development
> toolchain?

### Concepts

-   Architecture governance
-   Shift-left architecture
-   Automated guardrails
-   Architecture fitness functions
-   AI-assisted reviews
-   Human accountability

### Speaker Notes

Architecture as Code is not necessarily about asking AI to design the
entire system.

A more practical interpretation is making architectural rules testable
and accessible throughout the engineering lifecycle.

Some rules can already be enforced using static analysis, dependency
tests, policy-as-code, and CI/CD checks.

AI can potentially complement these deterministic controls by reviewing
areas that are harder to express as simple rules.

------------------------------------------------------------------------

# Slide 13 --- Architecture Diagrams as Code

## A practical pattern

Copilot can help generate architecture diagrams from repository context,
including:

-   Service boundaries
-   Data stores and queues
-   External APIs and integrations
-   Security and trust boundaries
-   Deployment topology

## Example prompt

``` text
Review this repository and generate a Mermaid component diagram for the
current architecture. Include services, data stores, messaging, and
external integrations. Keep it suitable for documentation and PR review.
```

## Mermaid example

``` mermaid
flowchart LR
    Client[Web / Mobile] --> API[API Layer]
    API --> Auth[Identity / Auth]
    API --> DB[(Database)]
    API --> Queue[Azure Service Bus]
    Queue --> Worker[Background Worker]
    API --> Cache[(Cache)]
```

## Why this is valuable

-   Works with GitHub Markdown and VS Code preview
-   Does not require a paid extension for basic diagram rendering
-   Can be stored as versioned repository artifacts
-   Supports Architecture as Code practices when diagrams live alongside
    source and ADRs

## Recommended workflow

1.  Ask Copilot to analyze the repository.
2.  Generate Mermaid output into a file such as
    `docs/architecture/system.md` or `docs/architecture/system.mmd`.
3.  Review and refine the diagram with architects and developers.
4.  Treat the diagram as a living architectural artifact that changes with
    the code.

### Reference

-   GitHub Mermaid documentation\
    https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams
-   GitHub Copilot documentation\
    https://docs.github.com/en/copilot

### Speaker Notes

This is one of the most practical ways to connect Copilot to architecture
work.

Instead of only generating code, Copilot can help produce architecture
artifacts that are reviewable, diffable, and stored with the repository.

That makes the architecture easier to evolve and easier to validate in
pull requests.

------------------------------------------------------------------------

# Slide 14 --- Use Case 3: PR Architecture Review

## Potential Workflow

``` text
Developer Creates PR
        ↓
Automated Tests
        ↓
Static Analysis
        ↓
Copilot Review
        ↓
Architecture Instructions
        ↓
Repository Context
        ↓
Potential Violations Identified
        ↓
Human Review
```

## Key Principle

Copilot should be an **additional reviewer**, not the final architecture
authority.

### Potential Benefits

-   Earlier feedback
-   Reduced repetitive review effort
-   Consistent reminders of engineering standards
-   More architect time for complex design decisions

### Speaker Notes

Imagine that before an architect reviews a pull request, the code has
already been checked for common architectural concerns.

The AI reviewer might flag a direct infrastructure dependency, missing
resilience handling, or a deviation from repository guidelines.

The architect can then spend more time on the difficult questions that
require business and system context.

The goal is not replacing architects.

The goal is reducing repetitive review work.

------------------------------------------------------------------------

# Slide 15 --- Use Case 4: Specialized AI Agents

Potential specialized agents:

``` text
Solution Architect Agent
        ↓
Architecture standards

Security Agent
        ↓
Security practices

API Governance Agent
        ↓
REST / OpenAPI standards

Cloud Architect Agent
        ↓
Azure best practices

Testing Agent
        ↓
Testing standards and quality
```

## Example Responsibilities

### Solution Architect Agent

-   Dependency boundaries
-   Integration patterns
-   Architecture standards

### Security Agent

-   Secrets
-   Authentication
-   Authorization
-   Common vulnerabilities

### Cloud Architect Agent

-   Azure service usage
-   Resilience
-   Scalability
-   Cost awareness

### Speaker Notes

Another emerging direction is specialized agents.

Instead of having one generic AI assistant, teams can define agents with
specialized responsibilities.

For example, a Solution Architect Agent could focus on architectural
boundaries while a Security Agent focuses on security concerns.

These agents should be viewed as engineering assistants and automated
guardrails.

Accountability still remains with the engineering team.

------------------------------------------------------------------------

# Slide 16 --- Advanced-Level Use Cases

## Advanced use cases for architects and platform teams

### 1. Architecture drift detection

Ask Copilot to compare the current implementation with the target
architecture and highlight drift in services, dependencies, or data flow.

### 2. Change impact analysis

Before a major change, ask Copilot to identify which services, APIs, and
security boundaries would be affected by a proposed modification.

### 3. Migration planning

Use Copilot to support modernization conversations such as:

-   Monolith to microservices
-   Legacy API to Azure-native services
-   On-premises to cloud architecture
-   Event-driven redesign

### 4. Security and compliance review

Ask Copilot to review the repository for:

-   Secret handling
-   Identity and access patterns
-   Logging and auditability
-   Compliance-sensitive architecture decisions

### 5. Incident reconstruction and runbook support

Feed Copilot logs, traces, and architecture context to help reconstruct
service interactions, identify bottlenecks, or draft operational follow-up
steps.

## Example prompt

``` text
Analyze this system for a potential migration to Azure Container Apps.
Identify the services, dependencies, network considerations, and
operational risks. Generate a target-state architecture summary and a
Mermaid diagram.
```

### Speaker Notes

These use cases move beyond simple code help and become more strategic.

The value is strongest when Copilot is used with repository context,
architecture documentation, and clear human review.

For architects, the opportunity is not to replace judgment but to make
architecture analysis faster, more consistent, and more reusable.

------------------------------------------------------------------------

# Slide 17 --- Live Demo: Copilot as an Architecture Assistant

## Demo 1 --- Understand the Solution

``` text
Analyze this .NET solution.

Identify:
- Architecture pattern
- Project dependencies
- Data access layer
- External integrations
- Potential architecture violations

Do not modify any code.
```

## Demo 2 --- Clean Architecture Review

``` text
Review this implementation against
Clean Architecture principles.

Identify violations and explain
their architectural impact.
```

## Demo 3 --- Azure Architecture Review

``` text
Review this code from the perspective
of an Azure Solution Architect.

Focus on:
- Scalability
- Resilience
- Security
- Observability
- Cost
```

### Demo Tip

Use a prepared repository so the demo does not depend on generating a
large project during the meeting.

### Speaker Notes

For the demo, I would use an existing small .NET solution.

The first prompt demonstrates repository understanding.

The second prompt focuses on architectural principles.

The third prompt changes the perspective to Azure architecture.

The important thing to observe is not whether every answer is perfect.

The question is whether the analysis saves enough architect time to be
useful as a first-pass review.

------------------------------------------------------------------------

# Slide 18 --- Where Architects Should Use Copilot

## Good Candidates

-   Codebase exploration
-   Architecture documentation assistance
-   PR review assistance
-   Dependency analysis
-   Technical debt discovery
-   Architecture guideline checking
-   Migration planning
-   Test strategy review
-   Initial modernization assessment

## Use With Caution

-   Final architecture decisions
-   Security approval
-   Compliance decisions
-   Production changes
-   Cost estimates without validation
-   Decisions requiring undocumented business context

## Key Message

> AI assists architectural reasoning.\
> It does not own architectural accountability.

### Speaker Notes

There is an important boundary.

Copilot can accelerate analysis, but it does not have full
organizational context.

It may not understand regulatory requirements, historical decisions,
commercial constraints, or undocumented business requirements.

Architects remain responsible for validating recommendations and making
decisions.

------------------------------------------------------------------------

# Slide 19 --- Technology Radar

  -----------------------------------------------------------------------
  Technology / Capability Suggested Position      Reason
  ----------------------- ----------------------- -----------------------
  HTTP QUERY              **ASSESS**              New standardized HTTP
                                                  capability with
                                                  interesting
                                                  complex-query semantics

  Copilot Code Review     **TRIAL**               Practical opportunity
                                                  for engineering
                                                  productivity

  Repository Custom       **TRIAL**               Potential to bring
  Instructions                                    engineering standards
                                                  closer to development

  Custom Copilot Agents   **ASSESS / TRIAL**      Promising specialized
                                                  engineering workflows

  AI-Assisted             **ASSESS**              Useful potential, but
  Architecture Review                             requires human
                                                  validation
  -----------------------------------------------------------------------

### Speaker Notes

If I put today's topics on a Technology Radar, I would place HTTP QUERY
in Assess.

For Copilot Code Review and repository instructions, Trial may be
appropriate for teams that already have Copilot access.

Custom agents and broader AI-assisted architecture reviews are areas I
would Assess or Trial through controlled experiments.

The key is to evaluate measurable value rather than adopting AI simply
because it is available.

------------------------------------------------------------------------

# Slide 20 --- Final Takeaway

## Two Changes Worth Watching

``` text
HTTP QUERY
     ↓
Evolution of API Semantics
```

``` text
GitHub Copilot
     ↓
Evolution from Coding Assistant
to Engineering Assistant
```

## Closing Thought

> The architect's role is not to adopt every new technology.

> It is to understand changes early, evaluate them in the context of the
> organization's architecture, and identify where they can genuinely
> improve engineering outcomes.

### Speaker Notes

My takeaway from these two topics is that changes are happening at two
different layers of our technology stack.

At the protocol level, HTTP is evolving with QUERY to better represent
complex, safe query operations.

At the engineering layer, AI tools such as GitHub Copilot are moving
beyond code generation toward repository-aware and agentic workflows.

For us as architects, the opportunity is not to immediately adopt every
new capability.

It is to understand these changes early, experiment where appropriate,
and decide where they can genuinely improve our architecture and
engineering practices.

------------------------------------------------------------------------

# Suggested 40-Minute Timing

  Section                                           Duration
  ----------------------------------------- ----------------
  Introduction                                     2 minutes
  HTTP QUERY --- Problem and Introduction          5 minutes
  GET vs POST vs QUERY                             4 minutes
  Enterprise Example                               3 minutes
  Architecture Implications and Radar              4 minutes
  GitHub Copilot Evolution                         3 minutes
  Architect Use Cases                              9 minutes
  Architecture as Code and PR Review               4 minutes
  Live Demo                                        4 minutes
  Technology Radar and Closing                     2 minutes
  **Total**                                   **40 minutes**

------------------------------------------------------------------------

# Useful References

## HTTP QUERY

-   RFC 10008 --- The HTTP QUERY Method\
    https://www.rfc-editor.org/rfc/rfc10008.html

-   RFC 10008 Information Page\
    https://www.rfc-editor.org/info/rfc10008/

## GitHub Copilot

-   GitHub Copilot Documentation\
    https://docs.github.com/en/copilot

-   GitHub Copilot Code Review\
    https://docs.github.com/en/copilot/how-tos/copilot-on-github/use-copilot-agents/copilot-code-review

-   GitHub Copilot Custom Agents\
    https://docs.github.com/en/copilot/concepts/agents/copilot-cli/about-custom-agents

------------------------------------------------------------------------

# Presenter Preparation Checklist

-   Prepare a small .NET repository for the Copilot demo.
-   Test all Copilot prompts before the meeting.
-   Keep screenshots as backup in case the live demo fails.
-   Verify HTTP QUERY support in your chosen demo tools before
    attempting a live QUERY demo.
-   Keep RFC 10008 open in a browser tab for reference.
-   Prepare one example of a complex enterprise search API.
-   Prepare one example of an architecture violation for the Copilot
    demo.
-   Keep the Technology Radar slide open for the closing discussion.

------------------------------------------------------------------------

# Optional Discussion Questions

If the audience becomes interactive, use these questions:

1.  Do we currently have POST-based search APIs that could conceptually
    benefit from QUERY semantics?
2.  What parts of our Azure infrastructure would we need to validate
    before experimenting with QUERY?
3.  Which architecture rules in our projects could be expressed as
    repository-level Copilot instructions?
4.  Could AI-assisted PR review reduce repetitive work for architects
    and senior developers?
5.  Which architecture decisions should always require explicit human
    review?
6.  Would an Architecture Review Agent be useful as a proof of concept
    for one of our repositories?
