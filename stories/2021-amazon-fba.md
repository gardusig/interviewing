## Amazon: Fulfillment by Amazon (FBA)

### Capacity Planning and Scaling for Black Friday Traffic

**Context**  
Amazon FBA invoice services are a critical component of downstream financial workflows for sellers, processing a large share of Brazil-specific compliance data. Ahead of Black Friday, traffic projections showed up to a **10× increase in requests**, making reliability and latency under peak load a high-risk concern.

**Goal**  
Validate that the service could safely handle Black Friday traffic and determine the **minimum required infrastructure** based on measured capacity, avoiding blind over-provisioning.

**Approach**  
- Analyzed several months of historical traffic to identify:
  - Request distribution patterns (read vs. write, lightweight vs. heavy paths)  
  - The top ~20% of endpoints responsible for the majority of CPU and DB usage  
- Used a TPS generator to simulate **realistic peak traffic**, matching production request ratios instead of uniform load.
- Load-tested individual EC2 instances to measure:
  - Sustained TPS before latency degradation  
  - CPU saturation points  
  - Database connection pool limits  
- Identified throughput bottlenecks driven primarily by CPU utilization and DB connection exhaustion.
- Modeled expected Black Friday capacity using measured per-host throughput, adding explicit safety headroom.
- Resized the EC2 fleet based on this model rather than static estimates.
- Implemented **dynamic auto-scaling policies** so the fleet could scale up during peak hours and scale back down automatically after traffic normalized.

**Outcome**  
- Successfully handled Black Friday traffic with **99.99% uptime** and no customer-visible incidents.
- Peak traffic was absorbed without latency SLO violations.
- Capacity planning was grounded in empirical measurements rather than assumptions.
- Automatic scale-down after the event reduced ongoing infrastructure costs while maintaining reliability.

---

### Making Load Testing a Mandatory Quality Gate

**Context**  
The team had a TPS (transactions-per-second) generator available, but it was unreliable and not part of the standard development workflow. As a result, performance regressions could ship unnoticed.  
This directly impacted the team’s **service reliability score**, a formal Amazon metric already trending low and triggering operational review pressure.

**Goal**  
Make load testing reliable, trusted, and enforceable so performance became a measurable, non-optional part of the release process.

**Approach**  
- Diagnosed why engineers avoided the TPS generator and found:
  - Default configurations produced unstable, misleading results  
  - Results varied significantly between runs, eroding trust in the tool  
- Iterated on configurations and validated results against known production behavior to stabilize measurements.
- Established a **production-aligned TPS baseline** that reflected real traffic patterns and latency expectations.
- Accelerated adoption by reusing a proven configuration from another team that had successfully passed reliability reviews.
- Integrated load testing into the **CI/CD pipeline**, configuring builds to automatically fail when TPS or latency thresholds were not met.
- Aligned with the team to treat performance failures as **release blockers**, not advisory signals.

**Outcome**  
- Load testing became a standard, automated quality gate for every deployment.
- Performance regressions were consistently caught in CI rather than in production.
- The team’s service reliability score improved over subsequent review cycles, reducing escalation risk.
- Release confidence increased, and performance ownership became part of the team’s normal engineering culture.

---

### Reducing On-Call Noise Through Root-Cause Fixes

**Context**  
When I joined the on-call rotation as a shadow, the service generated a high volume of alerts, particularly recurring **disk space alarms**.  
The common response was manual cleanup during incidents, which reduced immediate pressure but did not address the underlying issue.

**Goal**  
Even in a shadow role, proactively identify the root cause of recurring alerts and reduce on-call noise through a permanent fix.

**Approach**  
- Reviewed historical on-call tickets, logs, and disk usage metrics to identify alert frequency and patterns.
- Traced disk exhaustion to a specific invoice-generation workflow that:
  - Created temporary files during processing  
  - Failed to reliably clean them up after completion  
- Worked with the primary on-call engineer to implement a code fix ensuring temporary files were deleted as part of the normal execution path.
- Added targeted disk usage monitoring and alarms for the affected directories to enable early detection.
- Created a cleanup script for defensive recovery and updated runbooks to document:
  - Root cause  
  - Expected steady-state behavior  
  - Manual recovery steps if needed  

**Outcome**  
- Recurring disk space alerts were eliminated, significantly reducing on-call noise.
- Manual cleanup during incidents was no longer required.
- On-call tickets related to disk usage dropped by **~10%**.
- Improved observability and operational stability of the service.
- Enabled faster on-call ramp-up and reduced cognitive load for the rotation.

---

### Delivering Seller-Facing Invoice Improvements Under Tight Deadlines

**Context**  
Brazilian FBA sellers needed clearer visibility into invoice expiration rules, which vary by state and affect both legal compliance and the ability to transport goods to Amazon storage facilities.  
There was a **tight deadline to support two additional states**, while full nationwide coverage had no fixed timeline. I worked in pair with an engineer who had recently joined the team.

**Goal**  
Deliver correct, state-specific invoice expiration dates for the highest-priority states under a tight deadline, without introducing new services or blocking future extensibility.

**Approach**  
- Paired with the new team member to:
  - Transfer domain knowledge around Brazilian fiscal rules  
  - Accelerate delivery under time pressure  
  - Avoid knowledge silos and single-owner risk  
- Made an explicit short-term tradeoff:
  - Optimized for speed and correctness over full generalization  
- Implemented a **pragmatic interim solution**:
  - Extended existing **Java microservices** with state-specific expiration logic
  - Used hard-coded values where appropriate to meet the immediate deadline
  - Covered both invoicing and transport-to-fulfillment expiration constraints  
- Updated the **React-based UI** to clearly surface state-specific expiration dates to sellers.
- Ensured strict consistency between backend validation and frontend presentation.
- After stabilizing the solution, proposed a **long-term system design**:
  - A dedicated invoice expiration manager service
  - Automatic discovery of seller state
  - Centralized ownership of state-specific rules and metadata
  - Simpler extensibility for adding new states without duplicating logic

**Outcome**  
- Met the tight deadline for the two priority states without delaying dependent workflows.
- Sellers gained immediate, accurate visibility into invoice expiration rules.
- Reduced ambiguity around compliance and inbound shipment eligibility.
- Enabled the new engineer to ramp up quickly through pair programming and shared ownership.
- Established a clear migration path from hard-coded logic to a scalable, country-wide solution.
- Improved overall usability and correctness of FBA invoice tooling for Brazilian sellers.
