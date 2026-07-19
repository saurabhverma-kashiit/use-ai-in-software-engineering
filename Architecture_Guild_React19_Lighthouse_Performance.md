# Architecture Guild --- Modern React Performance

## React 19.2 + Lighthouse + Performance as an Architecture Quality Gate

**Duration:** \~40 minutes\
**Audience:** Architects, Senior Developers, Tech Leads, Frontend
Engineers\
**Theme:** Building modern React applications and making frontend
performance measurable and enforceable

------------------------------------------------------------------------

# Slide 1 --- Today's Technology Radar

### Part 1 --- Modern React

-   React 19 and 19.2
-   New capabilities
-   Enterprise relevance

### Part 2 --- Measuring Performance

-   Lighthouse
-   Core Web Vitals
-   Lab vs real-user performance

### Part 3 --- Performance Governance

-   Performance budgets
-   CI/CD quality gates
-   Preventing regressions

### Speaker Notes

We will connect recent React capabilities with practical performance
measurement and engineering governance.

------------------------------------------------------------------------

# Slide 2 --- Why Frontend Performance Is an Architecture Concern

``` text
Fast Backend API
        +
Large JavaScript Bundle
        +
Slow Rendering
        +
Unoptimized Images
        +
Third-Party Scripts
        ↓
Slow User Experience
```

Architecture quality attributes: - Performance - Scalability -
Accessibility - Reliability - Maintainability - User experience

> A fast API does not guarantee a fast application.

------------------------------------------------------------------------

# Slide 3 --- React Evolution

``` text
React 18
   ↓
Concurrent Rendering Foundations
   ↓
React 19
   ↓
Actions and Modern Async Patterns
   ↓
React 19.2
   ↓
Additional Performance and UX Capabilities
```

Architecture questions: - Which features simplify application design? -
Which improve user experience? - Which affect rendering? - What matters
for enterprise SPAs? - What matters for Micro Frontends?

------------------------------------------------------------------------

# Slide 4 --- React Actions

Traditional async mutation flow:

``` text
Submit → Set Loading → Call API → Handle Error → Update State → Clear Loading
```

Actions can simplify: - Pending-state handling - Error handling -
Optimistic updates - Form workflows - Async transitions

### Speaker Notes

The architectural value is reducing inconsistent boilerplate across
teams and establishing clearer async UI patterns.

------------------------------------------------------------------------

# Slide 5 --- `useActionState`

``` jsx
const [state, submitAction, isPending] =
  useActionState(saveCustomer, initialState);
```

``` text
User Submits Form
       ↓
Action Executes
       ↓
Pending State
       ↓
Success / Error
       ↓
UI Updates
```

### Architecture Benefit

More consistent patterns for asynchronous user actions.

------------------------------------------------------------------------

# Slide 6 --- Optimistic UI with `useOptimistic`

Traditional:

``` text
Click → Wait for Server → Update UI
```

Optimistic:

``` text
Click → Update UI Immediately → Server Request → Confirm or Roll Back
```

Good use cases: - Likes - Status changes - Simple low-risk updates

Use carefully: - Financial transactions - Irreversible operations -
Complex validation - Strong-consistency scenarios

------------------------------------------------------------------------

# Slide 7 --- React 19.2 Capabilities to Watch

-   `<Activity />`
-   `useEffectEvent`
-   `cacheSignal`
-   Performance Tracks
-   Partial Pre-rendering related capabilities
-   SSR improvements

``` text
New Feature
    ↓
Does It Solve a Real Problem?
    ↓
Compatibility?
    ↓
Performance Impact?
    ↓
Adopt / Trial / Assess
```

### Speaker Notes

Do not turn every new framework feature into an architecture standard.
Evaluate against real application needs.

------------------------------------------------------------------------

# Slide 8 --- Real Sources of React Performance Problems

``` text
Large Bundle
     +
Too Much JavaScript
     +
Unnecessary Re-renders
     +
API Waterfalls
     +
Large Images
     +
Third-Party Scripts
     ↓
Poor User Experience
```

Other causes: - Loading everything initially - Poor code splitting -
Large dependencies - Expensive component rendering - Duplicate Micro
Frontend dependencies - Excessive client-side processing

------------------------------------------------------------------------

# Slide 9 --- Code Splitting and Lazy Loading

Instead of:

``` text
Initial Load → Entire Application Bundle
```

Use:

``` text
Initial Load → Core Application → Load Feature When Needed
```

``` jsx
const Reports = React.lazy(() => import("./Reports"));

<Suspense fallback={<Loading />}>
  <Reports />
</Suspense>
```

### Benefit

Reduce initial JavaScript download and execution.

------------------------------------------------------------------------

# Slide 10 --- Enter Lighthouse

Lighthouse audits: - Performance - Accessibility - Best Practices - SEO

``` text
Open Application
      ↓
Run Lighthouse
      ↓
Collect Metrics
      ↓
Identify Opportunities
      ↓
Optimize
      ↓
Run Again
```

> Lighthouse is a diagnostic tool, not the final definition of user
> experience.

------------------------------------------------------------------------

# Slide 11 --- Important Performance Metrics

## LCP --- Largest Contentful Paint

How quickly does main content become visible?

## CLS --- Cumulative Layout Shift

How visually stable is the page?

## INP --- Interaction to Next Paint

How responsive is the application?

## FCP --- First Contentful Paint

When does the first content appear?

## TBT --- Total Blocking Time

How much is the main thread blocked in lab testing?

### Speaker Notes

Different metrics describe different parts of the user experience.
Understand them before defining targets.

------------------------------------------------------------------------

# Slide 12 --- Example: Poor React Application

-   5 MB JavaScript bundle
-   4 MB hero image
-   No route-based code splitting
-   Multiple third-party scripts
-   Sequential API requests
-   Expensive rendering

``` text
Browser
  ↓
Download Large JS
  ↓
Parse
  ↓
Compile
  ↓
Execute
  ↓
Fetch Data
  ↓
Render
```

> The API is fast, but the application still feels slow.

------------------------------------------------------------------------

# Slide 13 --- Optimization Example

Before:

``` text
Single Large Bundle
Large Images
Sequential API Calls
Unnecessary Re-renders
```

Improvements:

``` text
Route-Based Code Splitting
        +
Lazy Loading
        +
Image Optimization
        +
Parallel Independent API Calls
        +
Caching
        +
Render Optimization
```

After:

``` text
Smaller Initial Payload → Less Main-Thread Work → Earlier Content → Better Responsiveness
```

------------------------------------------------------------------------

# Slide 14 --- Lighthouse: Lab vs Real Users

## Lab Testing

``` text
Controlled Environment → Repeatable Test → Lighthouse
```

Useful for: - Development - Regression testing - CI/CD - Diagnostics

## Real User Monitoring

``` text
Real Devices + Networks + Geography + User Behavior
```

Useful for: - Production experience - Field performance - Device and
network differences

> Use lab data to prevent regressions. Use field data to understand real
> users.

------------------------------------------------------------------------

# Slide 15 --- Performance Budgets

Example:

``` text
JavaScript Bundle  < Defined Limit
LCP                < Target
CLS                < Target
Accessibility      > Target
Performance Score  > Minimum Threshold
```

Without a budget:

``` text
Feature + Feature + Dependency + Dependency → Gradual Degradation
```

With a budget:

``` text
Performance Regression → Detected Early → Engineering Action
```

------------------------------------------------------------------------

# Slide 16 --- Lighthouse in CI/CD

``` text
Developer
    ↓
Pull Request
    ↓
Build React Application
    ↓
Deploy Test Build
    ↓
Run Lighthouse CI
    ↓
Check Performance Budget
   ↙                 ↘
Pass                Fail
 ↓                    ↓
Continue           Investigate
```

> What if a major performance regression failed the build like a failed
> unit test?

### Speaker Notes

Thresholds need careful design because Lighthouse scores vary. Focus on
meaningful regressions rather than unstable gates.

------------------------------------------------------------------------

# Slide 17 --- Micro Frontend Performance Considerations

Micro Frontends may introduce: - Duplicate dependencies - Multiple
framework versions - Larger bundles - Additional network requests -
Inconsistent caching - Fragmented performance ownership

``` text
Micro Frontend A + Micro Frontend B + Micro Frontend C
                         ↓
                One User Experience
```

> Team independence should not mean performance independence.

------------------------------------------------------------------------

# Slide 18 --- Technology Radar

  Capability             Position         Why
  ---------------------- ---------------- -------------------------------------
  React 19/19.2          ASSESS / TRIAL   Modern capabilities
  React Actions          TRIAL            Simplifies async workflows
  Optimistic UI          TRIAL            UX benefits for suitable operations
  Lighthouse             ADOPT            Practical diagnostics
  Performance Budgets    ADOPT / TRIAL    Prevent degradation
  Lighthouse CI          TRIAL            Automated regression detection
  Real User Monitoring   ADOPT / ASSESS   Production experience

------------------------------------------------------------------------

# Slide 19 --- Proposed POC: Frontend Performance Quality Gate

1.  Choose one representative React application.
2.  Capture baseline:
    -   Bundle size
    -   Lighthouse Performance
    -   LCP
    -   CLS
    -   TBT
3.  Apply optimizations.
4.  Compare before and after.
5.  Introduce a CI performance check.

Success criteria: - Detect meaningful regressions - Stable test
execution - Actionable reports - Minimal developer friction

------------------------------------------------------------------------

# Slide 20 --- Final Takeaway

``` text
Modern React → Better Development Patterns
Lighthouse → Measurable Performance
Performance Budgets → Explicit Expectations
CI/CD → Continuous Enforcement
```

> Performance should not be something we test only before production. It
> should be an architectural quality attribute that we measure
> continuously.

------------------------------------------------------------------------

# Suggested 40-Minute Timing

  Section                           Duration
  ------------------------------- ----------
  Introduction                         2 min
  React 19/19.2                       10 min
  React Performance Patterns           7 min
  Lighthouse and Metrics               8 min
  Before/After Demo                    6 min
  Performance Budgets and CI/CD        4 min
  Radar and Closing                    3 min

------------------------------------------------------------------------

# Suggested Live Demo

1.  Run Lighthouse against an unoptimized React application.
2.  Record Performance, LCP, TBT, CLS and recommendations.
3.  Apply route-based code splitting.
4.  Add `React.lazy` and `Suspense`.
5.  Optimize images.
6.  Parallelize suitable independent API calls.
7.  Remove an unnecessary large dependency.
8.  Run Lighthouse again and compare.

------------------------------------------------------------------------

# Useful References

-   React Versions: https://react.dev/versions
-   React 19.2: https://react.dev/blog/2025/10/01/react-19-2
-   React Documentation: https://react.dev/
-   Chrome DevTools Lighthouse:
    https://developer.chrome.com/docs/devtools/lighthouse
-   Lighthouse Performance Scoring:
    https://developer.chrome.com/docs/lighthouse/performance/performance-scoring
-   Web Vitals: https://web.dev/articles/vitals
-   Lighthouse CI: https://github.com/GoogleChrome/lighthouse-ci

------------------------------------------------------------------------

# Optional Discussion Questions

1.  Do we measure frontend performance consistently?
2.  Should each Micro Frontend have an independent performance budget?
3.  Should the combined application have a separate budget?
4.  Which metrics should become CI quality gates?
5.  How much Lighthouse score variation should we tolerate?
6.  Do we have Real User Monitoring to complement lab testing?
7.  Which React 19 features could simplify our current applications?
