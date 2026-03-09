# Case Study: Solving 10s Latency in Data-Intensive Property Management Systems

**🚀 Performance Leap: 9.95s ➔ < 1.0s (90% Reduction in P99 Latency)**

## 1. Executive Summary
As a Lead Java Developer handling a large-scale **SaaS Property Management System (PMS)**, I encountered a critical performance bottleneck. A core API responsible for querying property staff evaluation tasks was suffering from **near 10-second timeouts** during peak hours. This caused cascading UI failures and DB connection pool exhaustion.

This repository documents the pragmatic approach I took to diagnose, refactor, and stabilize this distributed system without introducing heavy, over-engineered frameworks.

## 2. The Bottleneck: "Data Fan-out" & Legacy Permissions
The legacy system relied on a flawed, property-based data permission model. In a large property management context, the data retrieval flow was:
`Manager -> Property/Compound IDs (Hundreds) -> Staff IDs (Thousands) -> Evaluation Tasks (Massive)`

This extreme "Fan-out" effect caused massive `IN` clause database queries and blocked Feign client cross-service calls, pushing the P99 latency to 9.95 seconds.

### Evidence: Real-time Chaos
Below is the Grafana dashboard capturing the system's instability, with Max latency hitting **9.95s**:
![Grafana P99 Spikes](./images/grafana-p99-spikes.png)
![Grafana API List](./images/grafana-api-list.png)

## 3. Diagnosis: Tracing Without Heavy APM
In an environment lacking heavy APM tools like SkyWalking, I engineered a lightweight distributed tracing solution using **TraceId** and Alibaba Cloud Log Service (SLS).

By tracking the TraceId across microservices, I pinpointed that the vast majority of the latency was consumed by redundant cross-property staff data aggregation and unindexed database scans.
![SLS Trace Log](./images/sls-trace-log.png)

## 4. The Pragmatic Solution: Restructuring Data Flow
Instead of blindly scaling hardware or adding complex cache layers, I tackled the root architectural flaw:
1. **Permission Flattening (Database Level):** Refactored the data scope. Instead of recursively traversing property nodes, I introduced a dual-permission check (`Ownership Property` + `Assigned Property`), drastically reducing the SQL result set.
2. **Small Result Set Driving Large Sets:** Rewrote the massive `JOIN` and `IN` queries into targeted, index-hit queries based on the flattened permission model.
3. **Targeted Process Locks (Memory Protection):** For edge cases where a regional manager queried over 1,000 staff members simultaneously, I implemented a targeted memory pagination lock (`Targeted Resource Isolation`). This prevented Pod OOM (Out of Memory) crashes without blocking standard user traffic.

## 5. Quantitative Results & ROI
* **P99 Latency:** Dropped from extreme spikes (**9.95s**) to a stable **< 1.0s** (averaging around 500ms).
* **System Stability:** Eliminated Pod restarts caused by memory leaks during large dataset pagination.
* **Business Value:** Restored user trust for regional managers and drastically reduced cloud database CPU consumption, providing a high ROI solution with minimal code disruption.

---
*Looking for an expert to rescue your legacy Java systems or resolve deep-level performance bottlenecks? Let's connect on Upwork.*