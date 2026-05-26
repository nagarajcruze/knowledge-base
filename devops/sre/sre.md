# Site Reliability Engineering (SRE) Notes

Site Reliability Engineering (SRE) is responsible for the availability, latency, performance, efficiency, change management, monitoring, emergency response, and capacity planning of services.

> [!IMPORTANT]
> **100% is the wrong reliability target for basically everything.** Trying to hit 100% reliability is extremely expensive, slows down feature deployment, and offers diminishing returns because users' internet service providers, devices, and networks already introduce a baseline of failure.

---

## Alerting Philosophy
A system that requires a human to read an email and decide whether or not to take action is fundamentally flawed. Monitoring should never require a human to interpret alerts. Instead, software should do the interpreting, and humans should be notified only when they must take action.

- **Alerts**: Signify that a human needs to take action immediately in response to something that is either happening or about to happen in order to prevent an outage or degrade service.
- **Tickets**: Signify that a human needs to take action, but not immediately. The system cannot automatically handle the situation, but no damage will result if a human resolves it in a few days.
- **Logging**: Recorded for diagnostic or forensic purposes. No one reads logs unless prompted by another incident or investigation.

---

## Reliability Metrics

Reliability is a function of:
- **MTTF (Mean Time To Failure)**: The average time a system runs before failing.
- **MTTR (Mean Time To Repair)**: The average time required to bring a failed system back to health.

The most relevant metric in evaluating the effectiveness of emergency response is how quickly the response team can bring the system back to health (MTTR).

---

## Change Management

SRE has found that roughly **70% of outages are due to changes** in a live system. Best practices in change management use automation to accomplish:
1. **Progressive Rollouts**: Deploying changes to a small fraction of servers or users first.
2. **Quick and Accurate Detection**: Monitoring key metrics to spot anomalies instantly.
3. **Safe Rollbacks**: Automatically reverting to the last known stable state when problems arise.

These practices minimize the aggregate number of users and transactions exposed to bad changes.

---

## Capacity & Demand Planning

Demand forecasting and capacity planning ensure that there is sufficient capacity and redundancy to serve projected future demand with the required availability targets.
- **Organic Growth**: Natural product adoption and usage growth by customers.
- **Inorganic Growth**: Growth resulting from business-driven changes (feature launches, marketing campaigns, etc.).

---

## Borg: Distributed Cluster OS
Borg is Google's distributed cluster operating system (comparable to Apache Mesos or Kubernetes) that manages jobs at the cluster level.
- Tasks can use local disk as a scratch pad, but permanent storage is written to cluster-level filesystems (comparable to Lustre and HDFS).
- Communication uses protocol buffers (similar to Apache Thrift) and gRPC.

---

## SLIs, SLOs, and SLAs

We define these targets based on intuition, experience, and user needs to build system control loops.

- **SLI (Service Level Indicator)**: A quantitative measure of the level of service provided. Example: latency, error rate, throughput.
- **SLO (Service Level Objective)**: A target value or range of values for a service level that is measured by an SLI (e.g., $SLI \leq target$).
- **SLA (Service Level Agreement)**: A business or legal agreement with users outlining the SLOs, along with the consequences of failing to meet them (e.g., financial refunds). SREs generally focus on SLOs, not SLAs.

### Control Loops using SLOs
1. Monitor and measure the system's SLIs.
2. Compare the SLIs to the SLOs, and decide whether action is needed.
3. If action is needed, figure out what needs to happen to meet the target (e.g., scaling up instances if CPU-bound).
4. Take that action.

---

## Availability Targets

Availability targets are typically tracked over a period (e.g., quarterly) and monitored weekly or daily.

### Time-Based Availability
Based on the percentage of uptime in a given year.
- **99.99% Availability**: Max allowable downtime is **52.56 minutes per year**.

### Aggregate (Request-Based) Availability
Based on the percentage of successful requests.
- **99.99% Availability**: Out of 2.5 million daily requests, the system can serve up to **250 errors** and still hit its daily target.

---

## Developer Velocity vs. Reliability
One of the main benefits of SRE engagement is not just improved reliability, but improved product development output. A reduced Mean Time to Repair (MTTR) for common faults increases developer velocity, as engineers do not have to waste time and focus cleaning up or firefighting recurring issues.
