# Architecture Guild | Candidate Topic

## Web Locks API for Architects: Coordinating Shared Work in the Browser

**Duration:** ~30-40 minutes  
**Audience:** Architects, Senior Developers, Tech Leads, and Engineering Managers  
**Theme:** A deeper look at the Web Locks API as a browser-native coordination primitive for multi-context web applications

---

# Slide 1 --- Why This Topic Matters

## The browser is no longer just a single-threaded rendering surface; it is a multi-context execution environment.

### Architectural problem

Modern web applications increasingly run across:
- multiple tabs
- service workers
- shared workers
- background sync processes
- offline-first workflows

In these environments, the same logical work can be triggered concurrently by different contexts. Without coordination, teams risk:
- duplicate execution
- race conditions
- conflicting state transitions
- wasted network traffic
- inconsistent user experience

### Framing question

**How should architects coordinate shared work when the browser runtime itself becomes a distributed execution boundary?**

### Speaker Notes

The architectural relevance of the Web Locks API is that it introduces a browser-native concurrency control mechanism.
It is not a general distributed systems tool, but it solves a very real class of problems in modern web apps where multiple contexts may contend for the same task.

---

# Slide 2 --- What the Web Locks API Actually Is

## The Web Locks API is a coordination primitive for the browser platform.

### What it provides

It allows code running in different browsing contexts to coordinate access to named resources.
The API is designed around the idea of acquiring a lock for a resource name while work is in progress.

### Key capabilities

- name a resource or task that needs coordination
- request exclusive or shared access
- hold the lock for the duration of a critical section
- release it automatically when the task completes

### Example

```js
await navigator.locks.request('sync-job', async () => {
  await doSyncWork();
});
```

### Important nuance

This is not a database locking system and not a durable transaction mechanism.
It is a browser-scoped coordination tool for coordinating work within the same origin environment.

### Speaker Notes

At a technical level, the API is deceptively simple: you name a resource, request a lock, run work, and let the platform manage contention.
The power is in the fact that it works across browsing contexts without requiring a custom queue or handshake protocol.

---

# Slide 3 --- Execution Model and Semantics

## The lock model is simple, but the architectural implications are important.

### Core semantics

- the lock is identified by a string name
- code can request exclusive or shared access
- the browser coordinates requests across contexts for the same origin
- the lock is held for the duration of the callback execution

### Execution flow

1. A context requests a lock for a named resource.
2. If no conflicting lock exists, the request succeeds.
3. If another context holds a conflicting lock, the request waits or is queued depending on the mode and runtime conditions.
4. When the callback completes, the lock is released.

### Architectural significance

This model is useful for protecting critical sections without building a bespoke client-side coordination layer.

### Speaker Notes

The real insight is that the browser now exposes a concurrency primitive that is comparable in spirit to a lightweight mutex or critical section, but at the platform level.
That matters for design because it changes how we think about cross-context coordination in the client.

---

# Slide 4 --- A More Technical Example

## A practical implementation pattern looks like this.

```js
async function runIfSingleSync() {
  await navigator.locks.request('background-sync', { mode: 'exclusive' }, async () => {
    await refreshLocalCache();
    await pushPendingChanges();
  });
}
```

### Why this pattern is valuable

- only one context performs the work at a time
- duplicate refreshes are avoided
- the code remains declarative and localized
- the coordination logic is embedded in the platform rather than ad hoc

### Variations

You can also use shared locks for read-style coordination, where multiple readers can proceed but writers still need exclusivity.

### Speaker Notes

This is the sort of code pattern that can remove a lot of brittle custom logic.
Instead of each tab independently deciding whether it should sync, the browser coordinates the work for you.

---

# Slide 5 --- Typical Use Cases in Modern Web Architecture

## The best use cases are those where the browser is an active coordination boundary.

### 1. Deduplicating background sync

Two tabs may both trigger refresh logic. A lock ensures only one execution path performs the work.

### 2. Coordinating service worker and page work

A page and a service worker may both attempt to refresh or reconcile local state. The lock avoids both acting at once.

### 3. Serializing access to local shared state

A queue, cache, or index update can be protected so that multiple contexts do not write concurrently.

### 4. Avoiding duplicate offline reconciliation

When connectivity returns, several contexts may try to reconcile pending operations. A lock makes that work single-flight.

### 5. Protecting UI-driven workflow steps

A multi-tab workflow can use locks to ensure one context owns a specific business operation at a time.

### Speaker Notes

The important architectural pattern is single-flight coordination.
The API is most helpful when multiple contexts might otherwise attempt the same expensive or stateful operation at once.

---

# Slide 6 --- Where It Fits in the Architecture Stack

## The Web Locks API is most useful when it sits alongside other browser capabilities rather than replacing them.

### Related platform capabilities

- Service Workers for background and offline execution
- Cache API and IndexedDB for local persistence
- BroadcastChannel for messaging between contexts
- Shared Workers for shared execution contexts
- Background Sync for deferred network work

### Architecture view

These capabilities form a richer browser-side operating model for:
- offline-first experiences
- local persistence
- state reconciliation
- coordination across contexts

### Architectural takeaway

The Web Locks API is best seen as part of a broader browser platform strategy rather than as an isolated feature.

### Speaker Notes

This is where the architectural framing becomes stronger.
You are not just evaluating a new API; you are evaluating how the browser platform is maturing into a more capable runtime for multi-context applications.

---

# Slide 7 --- How It Compares to Other Coordination Options

## It is useful to compare the Web Locks API to other common approaches.

### Compared to custom client-side queues

Pros:
- less custom orchestration code
- standardized coordination behavior
- fewer race-condition bugs in the browser layer

Cons:
- less visibility into queue policies
- not a full workflow engine
- still needs careful design around lock duration

### Compared to BroadcastChannel

- BroadcastChannel is primarily for messaging
- Web Locks is for coordination and mutual exclusion
- they solve different problems and can complement each other

### Compared to server-side locking

- server-side locking is durable and globally coordinated
- Web Locks is browser-local and ephemeral
- it is not a replacement for enterprise coordination, distributed transactions, or cross-device consistency

### Speaker Notes

This distinction is critical for architects.
The Web Locks API is not trying to replace server-side concurrency control; it is filling a browser-local coordination gap.

---

# Slide 8 --- Design Trade-offs and Failure Modes

## The API is simple, but there are important design constraints.

### Key trade-offs

- lock lifetime must be short and well-understood
- long-running work can keep a lock for too long, reducing throughput
- lock contention can introduce waiting behavior and perceived latency
- the feature is not durable across browser restarts or cross-device contexts

### Failure modes to plan for

- callback throws an error and the lock is released
- the page or worker is terminated while holding a lock
- the work is too slow and blocks other contexts longer than intended
- unsupported environments require fallback behavior

### Architectural implication

This is best used for short-lived, local coordination, not for long-running business-critical state management.

### Speaker Notes

One of the most important architecture lessons is that locks are only as good as the scope and duration of the work they protect.
If the critical section is too broad, the coordination mechanism itself becomes a bottleneck.

---

# Slide 9 --- Why It Matters for Reliability Engineering

## From a reliability perspective, the key benefit is reducing duplicate and conflicting work.

### Reliability benefits

- fewer duplicate background jobs
- less chance of state corruption from concurrent updates
- more predictable behavior during multi-tab interactions
- better single-flight semantics for expensive operations

### What this enables

Teams can move from “hope the right context wins” logic to a more explicit coordination model.
That improves operational predictability and reduces edge-case bugs.

### Speaker Notes

This is especially relevant in systems with offline or intermittent connectivity.
When multiple contexts attempt recovery or reconciliation at once, coordination becomes a reliability feature rather than a convenience feature.

---

# Slide 10 --- Implementation Pattern for Production Systems

## A robust production design usually combines the lock with fallback behavior.

### Suggested pattern

1. Attempt to acquire a lock for the critical operation.
2. If successful, perform the work and release the lock.
3. If the environment does not support the API, fall back to a simpler strategy.
4. Keep the protected region as small as possible.
5. Log or instrument contention to understand whether the coordination is too coarse.

### Example structure

```js
async function guardedSync() {
  if (!('locks' in navigator)) {
    return fallbackSync();
  }

  return navigator.locks.request('sync-job', async () => {
    await doSyncWork();
  });
}
```

### Architecture takeaway

Use the API as an enhancement to the app’s coordination model, not as a substitute for backend validation and business rules.

### Speaker Notes

The production guidance is straightforward: treat this as a platform capability to improve coordination, but never as the only guardrail.
Server-side validation and business rules still matter.

---

# Slide 11 --- Security and Isolation Considerations

## The Web Locks API is scoped to the browser environment, which has important implications.

### Security implications

- the API is bounded by the origin model
- it does not provide cross-user or cross-device coordination
- it should not be treated as a security boundary
- it is not a substitute for authentication or authorization

### Why this matters architecturally

The lock is helpful for coordination, but it does not solve trust, identity, or shared-state integrity across users or systems.

### Speaker Notes

This is an easy place to overestimate the capability.
The API helps coordinate execution inside the browser, but it does not give you distributed trust semantics.

---

# Slide 12 --- Recommended Position for the Technology Radar

## Suggested assessment position

### ASSESS

Why:
- it addresses a real concurrency problem in modern web applications
- it is relevant to multi-tab, service worker, and offline-first scenarios
- it reduces the need for custom client-side coordination code
- it fits naturally with modern browser platform evolution

### Recommendation

Treat the Web Locks API as a valuable capability to evaluate where browser-side coordination is important, especially in multi-context applications.

### Speaker Notes

For an Architecture Guild discussion, this is an excellent Assess topic because it combines practical value with architectural nuance.
It is not a universal solution, but it does address a genuine gap in the web platform.

---

# Slide 13 --- What to Watch

## The deeper architectural signal is not the API alone; it is the emergence of richer browser-native coordination patterns.

### Watch for these signals

- whether teams are shifting from custom queues to platform-supported coordination
- whether multi-context applications are becoming more common in enterprise web solutions
- whether browser platforms continue to evolve toward more robust offline and background execution models
- whether the API is used as part of a broader coordination strategy rather than a one-off workaround

### Final takeaway

If your application has shared work across tabs, workers, or background tasks, the Web Locks API is worth understanding as a lightweight, browser-native coordination mechanism.

### Speaker Notes

The strongest architectural message is that this API is a good example of how the web platform is evolving from a document renderer into a more capable execution environment.

---

# Slide 14 --- Closing Thought

## The browser is increasingly becoming a runtime that must coordinate work, not just render UI.

### Closing message

The Web Locks API is important because it gives architects a native way to manage contention, deduplicate work, and make multi-context applications more predictable.

> The right architecture is not just about building features; it is about choosing the right coordination primitive for the runtime environment.

---

# Suggested References

- MDN Web Docs: Web Locks API: https://developer.mozilla.org/en-US/docs/Web/API/Web_Locks_API
- MDN Web Docs: Service Workers: https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
- MDN Web Docs: BroadcastChannel: https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel
- MDN Web Docs: IndexedDB: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
- MDN Web Docs: Background Sync: https://developer.mozilla.org/en-US/docs/Web/API/Background_Sync_API

---

# Presenter Notes Summary

To make the talk more compelling, use one concrete example from your environment:
- a multi-tab workflow that currently causes duplicate sync work
- a service worker that competes with page-level refresh logic
- a background reconciliation process that would benefit from single-flight coordination

That makes the session feel deeply practical rather than theoretical.
