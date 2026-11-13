# NIL migration plan — Blue-Green Deployment

**Status:** alternative plan requested for comparison; it does not replace the agreed swing plan. Both centres already operate; physical relocation is outside scope.

**Idea:** keep the complete current environment (**blue**) serving users while preparing a complete destination environment (**green**). After testing and final data synchronization, switch service traffic to green together. Blue remains protected for recovery.

## Plan overview

1. Prepare green while blue stays live.
2. Validate green in isolation.
3. Define the traffic switch, database handover and rollback rules.
4. Rehearse switching and recovery.
5. Approve the production window.
6. Switch to green and verify service.
7. Observe and accept green.
8. Retire blue after acceptance and the recovery period.

## 1. Prepare the destination

Create a complete green environment with the required VMs, applications, databases, network rules, certificates, access controls, backups and SOC monitoring. Verify capacity and platform compatibility.

Keep green isolated during testing: prevent duplicate identities/IP addresses, production writes, repeated scheduled jobs and unintended external notifications.

**Data approach for this plan:** blue's database remains authoritative while changes replicate to green. Green does not accept production writes before the handover. Database cutover needs a controlled consistency checkpoint; two environment copies do not automatically provide zero downtime.

**First step:** prepare and test the isolated green copy while users stay on blue.

## 2. Validate green

| Area | Check |
|---|---|
| VMs and configuration | Correct resources, disks, drivers and successful startup |
| Data | Database-native integrity checks and comparison at a common replication checkpoint |
| Applications | Login, read and isolated test-write workflows; required integrations |
| Traffic and sessions | Destination routing, certificates, session handling and connection draining |
| Security and SOC | Intended access restrictions, logs and end-to-end test alerts |
| Recovery and performance | Successful backup restore and representative load within agreed limits |

Record expected result, actual result, pass/fail and evidence. Fix failures and retest. Test writes must not contaminate the final replica; recreate or resynchronize it using a verified procedure before production use.

## 3. Define switching and rollback

Use a tested load-balancer/routing change to move traffic. If DNS is used, allow for caches and long-lived connections. Old endpoints must not continue accepting writes after handover.

- Before green accepts writes, pause/drain source activity as required, complete replication, confirm consistency and fence blue's writes.
- Resume writes only on the verified green service.
- If testing fails before handover, keep users on blue.
- After handover, stop green writes before recovery. If green accepted new data, reconcile/reverse-replicate it to blue through the tested database procedure before reopening blue.
- If safe reconciliation is unavailable, keep writes stopped and invoke the approved recovery procedure. Routing traffic back alone does not recover database state.

Define failure thresholds, a latest safe rollback start and the authorized decision-maker. Include application state, queues, file uploads and scheduled jobs in recovery, not only database rows.

## 4. Rehearse the switch and recovery

1. Prepare isolated blue and green test environments with matching data.
2. Run representative test traffic against blue; pause writes and complete synchronization.
3. Fence blue writes, activate green and switch test traffic.
4. Verify workflows, sessions, data, performance and SOC events.
5. Create identifiable records on green, then simulate a failure.
6. Stop green writes, reconcile new data, restore blue as authoritative and switch test traffic back.
7. Confirm no transactions were lost or duplicated. Record switching and recovery durations; correct failures and repeat.

## 5. Schedule and approve

Choose a low-impact window that avoids critical business cycles and conflicting jobs. Maintain continuous SOC coverage and ensure engineers/providers are available.

Name the service owner, change manager, application/database owner, engineers and SOC contact. Agree allowed interruption, recovery time, acceptable data loss, performance limits and observation duration. Use rehearsal timings to reserve recovery and validation time plus contingency.

Inform users of impact and update channels. The decision authority records **go or postpone** after confirming passing tests, accepted residual risks, healthy replication, usable backups, monitoring and available staff.

## 6. Perform the production switch

1. Confirm final approval and freeze unrelated changes.
2. Drain/pause blue writes and relevant jobs; record the last consistent checkpoint.
3. Finish synchronization and verify green's data.
4. Fence blue's writes and prevent duplicate background processing.
5. Activate green in dependency order, switch traffic and resume permitted operations.
6. Repeat critical validation checks. Proceed if criteria pass; activate rollback if agreed limits are reached.

Log actions, results and decisions. Blue remains protected, but may become stale after green accepts writes; recovery must follow the data-aware procedure.

## 7. Monitor and accept

Observe application errors, business transactions, latency, capacity and SOC alerts through an agreed period covering relevant service cycles. Confirm jobs, backups and recoverability. Service owners review evidence and formally accept green after required issues are resolved.

## 8. Close and retire blue

After acceptance and the recovery period, confirm no active dependencies remain on blue. Obtain retirement approval, preserve required backups/records, update diagrams and recovery procedures, remove temporary access and replication, securely retire obsolete copies and record lessons learned.

**Example:** users access the complete service on blue while green is tested. In the approved window, writes briefly pause, final replication completes and traffic switches to green. If green fails after accepting transactions, those transactions must be reconciled before blue can reopen.

## Standards and suitability

References use the supplied editions: **SIST EN ISO/IEC 27001:2017** (4.3, 6.1.2–6.1.3, 8.1, A.12.1.2, A.12.3–A.12.4, A.17); **SIST EN ISO 22313:2020** continuity guidance (8.2–8.6); **SIST EN ISO 9001:2026** (8.2, 8.5.6, 8.6–8.7, 10.2). Maintain risk treatment/applicability decisions, BIA, approved criteria, test evidence and release authority. This plan supports their application; it does not establish full conformity.

This is close to our coordinated swing plan: the distinction is the explicit pair of complete environments and whole-service traffic switch. It fits tightly connected workloads if capacity and database handover support it.

Pattern reference: [AWS blue-green deployment guidance](https://docs.aws.amazon.com/whitepapers/latest/blue-green-deployments/introduction.html). Vendor-specific capabilities must be checked for the actual platforms.
