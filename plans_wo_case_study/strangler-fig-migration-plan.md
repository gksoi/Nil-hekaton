# NIL migration plan — Strangler Fig Pattern

**Status:** alternative plan requested for comparison; it does not replace the agreed swing plan. Both centres already operate; physical relocation is outside scope.

**Idea:** gradually replace specific application functions at the destination while the old system continues providing the remaining functions. A routing layer directs each request to the implementation responsible for that function.

This is an application modernization pattern, not simply a way to copy whole VMs. Use it only if application changes, function separation and interface integration are feasible and authorized. If the task is strictly unchanged V2V relocation, prefer a supported infrastructure migration approach.

## Plan overview

1. Select a separable function and prepare the routing boundary.
2. Build and validate its destination replacement.
3. Define per-function ownership, switching and recovery.
4. Rehearse coexistence and rollback.
5. Approve the first function's production release.
6. Replace functions one at a time.
7. Accept each function and the complete destination service.
8. Retire the old application after all dependencies are removed.

## 1. Prepare the first replacement

Identify a small function with clear inputs, outputs and dependencies, such as read-only reporting. Define its existing interface and expected behavior before changing it.

**First step:** confirm one function can be separated safely, then prepare a highly available routing façade/proxy that can direct that function to the destination while keeping other functions at the source. Protect and monitor the façade; it becomes a critical dependency.

Build the replacement in an isolated destination environment. Record which system owns each function and data set. Where legacy interfaces differ, use an adapter with explicit contract and security checks.

Start with read-only functionality where practical. For writable functions, choose one authoritative writer per data set and a tested way to expose changes to the other system. Avoid naive dual writes: partial failures can leave systems inconsistent.

## 2. Validate the replacement

| Area | Check |
|---|---|
| Functional behavior | Same agreed business results and interface contracts as the existing function |
| Routing | Selected function reaches destination; unrelated functions remain at source |
| Data | Integrity, mapping, freshness and synchronization meet approved limits |
| Coexistence | Workflows spanning old/new functions complete correctly |
| Security and SOC | Identity/authorization survives routing; required logs and alerts reach SOC |
| Resilience | Proxy failure, unavailable dependency, backup restore and recovery are tested |
| Performance | Function and end-to-end workflow meet agreed limits |

For read-only requests, compare old/new results using controlled replay or protected test inputs where suitable. Do not mirror production writes to both systems unless a deliberately designed, verified mechanism makes side effects safe.

Record expected/actual results and evidence. If requirements change during redesign, obtain approval and update the tests.

## 3. Define function cutover and rollback

For every function document its route, data owner, interfaces, dependencies, acceptance tests, rollback trigger and decision authority.

- For a read-only function, rollback can return its route to a healthy legacy implementation after checking compatibility and availability.
- For a writable function, pause/drain affected operations, synchronize data, transfer ownership and fence the old writer before opening the new writer.
- Returning the route is safe only if the legacy function can access current, compatible data. Reconcile destination writes before transferring ownership back.
- If business actions produced external effects, such as notifications or payments, handle them through the approved business reconciliation/compensation procedure. Restoring a database cannot undo those effects automatically.
- If safe recovery is unavailable, contain the affected operations and use the continuity plan; do not silently reopen stale functionality.

Retain working old implementations through their approved recovery periods. Define an overall rollback strategy as well: individually reversible functions may still have dependencies that prevent arbitrary rollback order.

## 4. Rehearse coexistence and recovery

1. Prepare isolated legacy and replacement systems with representative test data.
2. Route only the selected function to the replacement; verify other functions still use legacy paths.
3. Execute workflows crossing the old/new boundary and verify data and permissions.
4. For a writable function, add identifiable test transactions and verify synchronization, event handling and deduplication where applicable.
5. Simulate replacement, proxy and connectivity failures.
6. Practice route rollback and, where needed, reconciliation plus ownership transfer back.
7. Confirm complete business results without missing or duplicated transactions. Measure recovery duration, fix failures and repeat affected tests.

## 5. Schedule and approve each replacement

Choose a low-impact, staffed window for each function. Keep SOC monitoring continuous and inform affected users of changed behavior or interruptions.

Name function/data owners, application engineers, the change authority, operations and SOC contacts. Agree interruption, data-loss/freshness limits, performance, observation period and recovery deadline. Confirm accepted risks, passing interface/coexistence tests, backups, monitoring and rollback readiness before recording **go or postpone**.

Each function is a controlled release. Changes to the application architecture need review and validation under the quality-management design/change process, not only infrastructure approval.

## 6. Replace production functions gradually

1. Confirm readiness and keep the rest of the application on the old system.
2. Complete required data preparation; transfer write ownership safely if applicable.
3. Change the façade's route for the selected function.
4. Validate its business behavior, cross-system workflows, data, security and performance.
5. Observe and accept that function before selecting the next suitable boundary.
6. Repeat until all in-scope functions and supporting dependencies have moved or been replaced.

Move legacy data ownership in controlled steps. Resolve cross-function transactions using a designed consistency approach; if functions cannot be separated safely, migrate that coupled group together or reconsider this pattern.

## 7. Monitor and accept

After each replacement, monitor both systems, routing, replication/event lag, business results and SOC events. Confirm destination backups and recovery. Obtain function-owner acceptance and retain rollback options for the agreed period.

Before accepting the whole service, verify all workflows, jobs, reports, data stores and integrations work without unapproved legacy dependencies. A new user interface alone does not mean the old system has been replaced.

## 8. Close and retire the legacy system

After full acceptance and recovery periods, confirm no requests, jobs, data ownership or required integrations remain at the old system. Preserve required records/backups, update diagrams and recovery instructions, remove temporary adapters/access and securely retire obsolete workloads with approval.

Retain the façade if it remains part of the target architecture; remove it only after clients can safely use the approved permanent routes. Record lessons learned and close the change records.

**Example:** move a read-only reporting function first while all updates remain in the old application. Then replace another separable function with an explicit data-ownership transition. Continue until all functions and data dependencies are at the destination; only then retire the legacy application.

## Standards and suitability

References use the supplied editions: **SIST EN ISO/IEC 27001:2017** (6.1.2–6.1.3, 8.1, A.9, A.12–A.14, A.17); **SIST EN ISO 22313:2020** continuity guidance (8.2–8.6); **SIST EN ISO 9001:2026** (8.2–8.3, 8.5.6, 8.6–8.7, 10.2). Keep BIA, security/applicability decisions, interface/design reviews, validation evidence and release approvals. This plan does not establish full conformity.

This approach suits gradual application replacement with clear functional boundaries. It usually requires more application engineering than moving an unchanged, tightly connected server stack.

Pattern reference: [Microsoft Strangler Fig pattern guidance](https://learn.microsoft.com/en-us/azure/architecture/patterns/strangler-fig).
