# NIL migration plan — Canary Migration

**Status:** alternative plan requested for comparison; it does not replace the agreed swing plan. Both centres already operate; physical relocation is outside scope.

**Idea:** send a small, controlled group of real users or requests to a ready destination service. Compare its results with the source, then increase traffic only when checks pass. Users never receive an unfinished service.

## Plan overview

1. Prepare a destination that can safely coexist with the source.
2. Test routing, functionality and data consistency.
3. Define traffic stages and recovery rules.
4. Rehearse progression and rollback.
5. Approve stages and observation criteria.
6. Move production traffic gradually.
7. Accept the destination after all traffic has moved.
8. Remove source dependencies and retire old workloads.

## 1. Prepare safe coexistence

Prepare the destination application/VM environment with required capacity, dependencies, security, backups and SOC feeds. Add a router/load balancer that can select a stable cohort, such as named pilot accounts or tenants, and distinguish source/destination metrics.

**First step:** prove that the same service can operate safely in both centres during the canary period.

For this plan, both application environments initially use **one authoritative production database**, with compatible application behavior and tested cross-centre connectivity. Destination application writes reach that same database; do not create independently writable production clones. Verify session handling, files, caches, queues and singleton jobs too.

This initially migrates application traffic, not the database. Move the database later through a coordinated handover after validating source/destination compatibility and recovery. If shared state or properly isolated tenant ownership cannot be supported, canary is unsuitable for the complete stack as proposed.

## 2. Validate before the first real user

- Test VMs, configuration, application workflows, permissions and integrations in isolation.
- Verify controlled traffic routing, stable cohort assignment, sessions and exclusion of unintended users.
- Test both application environments against the intended data interface and validate transactions.
- Check cross-centre latency, capacity, SOC visibility, backup restore and source capacity to receive all traffic again.
- Prevent duplicate background jobs, messages, notifications and other side effects.

Record expected/actual results and evidence. Define absolute service limits and comparisons with source performance; do not rely only on an overall average that hides canary failures.

## 3. Define stages and rollback

An **illustrative**, not approved, progression is: named pilot group → 5% → 25% → 50% → 100%. Choose actual stages based on workload, cohort isolation and risk.

For each stage agree minimum observation time, representative transaction volume, business workflows, error/latency limits and SOC checks. Low traffic may require longer observation. The change authority approves each increase; a timer alone does not justify promotion.

- If application checks fail, stop promotion and return the affected cohort to the source after draining requests safely.
- With the shared database unchanged and compatible, accepted transactions remain there. Confirm the source can read data written by the destination before reopening traffic.
- Do not roll back the shared database merely to undo an application rollout; that could erase valid work by both cohorts.
- If destination behavior corrupts data or makes it incompatible, contain writes and use the tested repair/recovery procedure. Traffic reversal alone is insufficient.
- After the database moves, any database rollback requires fencing and reconciliation of new writes before source reopening.

## 4. Rehearse progression and recovery

1. Use isolated test environments and a shared test database.
2. Route a small stable cohort to the destination and the rest to the source.
3. Create and update test records from both cohorts; verify correct visibility and no lost updates or duplicated effects.
4. Increase traffic in test stages and verify per-cohort metrics and capacity.
5. Simulate a destination failure; stop promotion and reroute its cohort to source.
6. Confirm sessions, transactions and source capacity after reversal.
7. Rehearse the later database handover separately, including reconciliation after new writes and loss of cross-centre connectivity.

Record timings and failures; fix problems and repeat affected tests.

## 5. Schedule and approve

Choose low-impact staffed windows for the first cohort, major increases and database handover. Keep SOC coverage continuous. Tell affected users about the pilot where appropriate and agree support/escalation channels.

Name execution and decision owners. Approve downtime/data-loss targets, stage limits, minimum samples, observation periods, rollback triggers and recovery deadlines. Confirm recoverable backups, compatible state, healthy monitoring and tested source fallback capacity. Record **go or postpone** for entry and each promotion.

## 6. Perform the production migration

1. Confirm final readiness and retain the source application as the default route.
2. Enable the agreed pilot cohort at the ready destination.
3. Compare cohort-specific business results, errors, latency, integrity and SOC events with approved criteria.
4. Increase traffic only after each stage passes; stop or reverse if limits fail.
5. At 100%, continue observation. The migration is not complete while the authoritative database or other required dependencies remain at the source.
6. In its approved window, pause/drain all database writers and jobs, complete final synchronization, fence old database writes and promote the destination database.
7. Update and verify all required application/database connections, resume writes and repeat critical checks. If failing, use the rehearsed data-aware recovery procedure.

Maintain one authoritative writer per data set. Changing application traffic percentages does not partition database ownership automatically.

## 7. Monitor and accept

Check all cohorts and business workflows at full destination load, including jobs, integrations, SOC alerts and backup/recovery coverage. Confirm that required database, file and queue dependencies now operate at the destination. Obtain service-owner acceptance after the agreed observation period and resolution of required defects.

## 8. Close and retire source workloads

After acceptance and the recovery period, verify no traffic or required dependency remains at the source. Retain required evidence/backups, update documentation, remove temporary cross-centre access/routing, retire obsolete workloads through approval and record lessons learned.

**Example:** a small pilot group uses the destination application while everyone else uses the source application; both use the same authoritative database. After all traffic moves successfully, the database is moved in a separate controlled window. Failure of the pilot sends its users back without discarding valid shared-database transactions.

## Standards and suitability

References use the supplied editions: **SIST EN ISO/IEC 27001:2017** (6.1.2–6.1.3, 8.1, 9.1, A.12–A.13, A.17); **SIST EN ISO 22313:2020** continuity guidance (8.2–8.6); **SIST EN ISO 9001:2026** (8.2, 8.5.6, 8.6–8.7, 9.1, 10.2). Keep approved risks/applicability decisions, BIA, cohort tests, promotion approvals and recovery evidence. The plan does not establish full conformity.

Canary requires safe simultaneous service operation and controllable traffic. It adds complexity for a tightly coupled, stateful stack; it is not a generic hypervisor migration capability.

Pattern reference: [Kubernetes canary deployment explanation](https://kubernetes.io/docs/tutorials/stateless-application/canary-deployment/). The traffic-cohort principle applies here without assuming Kubernetes is used.
