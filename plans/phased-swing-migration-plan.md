# NIL migration plan — Phased swing migration

This file records only ideas we have agreed to use.

## Plan overview

1. Prepare an isolated destination copy while users remain at the old centre.
2. Validate the copy against the agreed checks.
3. Rehearse the coordinated switch and recovery, including preserving new test data.
4. Review the rehearsal results, schedule the real switch and assign decision authority.
5. Confirm readiness and give the final go/postpone decision.
6. Complete final synchronization, switch users and validate the live service; roll back safely if needed.

## 1. Phased swing migration

**Decision:** use phased swing migration between the two operational centres. Physical relocation is outside scope.

Prepare and test the destination while the source continues serving users. Move suitable workloads in controlled stages, validate each stage and retain the source for an agreed recovery period.

### First step

Prepare an isolated copy of the source server and its workloads at the destination. Test the VMs, databases and applications without affecting production. Prevent duplicate IP addresses, duplicate scheduled jobs and unintended production writes.

### Migration sequence

1. Replicate data while the source remains operational.
2. Test a suitable workload or service group at the destination.
3. Complete final synchronization and switch service traffic in a controlled way.
4. Verify service operation, data consistency and monitoring before proceeding to the next stage.
5. Keep the source protected and recoverable until the agreed acceptance and recovery period ends.

### Conditions

- Move workloads separately only where their dependencies allow it. Tightly coupled workloads use phased preparation and testing followed by a coordinated cutover.
- Maintain one authoritative writer for each data set; prevent conflicting writes between centres.
- If the destination has accepted new writes, reconcile those changes before returning to the source.
- The approach supports controlled change, tested continuity and verified results under the supplied standards. Downtime and recovery targets still need to be agreed and tested.

## 2. Validate the destination copy before cutover

**Decision:** use a validation checklist with agreed expected results, comparing the isolated destination copy with the working source.

| Area | Check | Expected result |
|---|---|---|
| VMs | Start the expected VMs and verify CPU, memory, disks and configuration. | VMs boot successfully and match the intended configuration. |
| Data | Run database-native integrity checks and compare data at the same replication checkpoint. | Integrity checks pass and data matches the selected checkpoint. |
| Applications | Log in, read a record, create or update test data, and verify the result. | Representative workflows complete correctly within the isolated test environment. |
| Connections | Test required application, database and service connections; use test endpoints for external integrations. | Required connections work without unintended production interactions. |
| Security and monitoring | Test authorized and unauthorized access; generate a test event and verify SOC receipt. | Intended permissions are enforced and the SOC receives the event. |
| Performance | Run the same representative workload in both environments and compare response times and errors. | Results meet agreed service limits; record differences from the source baseline. |

Keep test writes separate from production. Compare data against a common checkpoint: a source that continues changing will not necessarily match a static test copy. Run source performance checks without disrupting production.

### Record and approve results

For each check record **expected result, actual result, pass/fail and evidence**. Set measurable performance limits before testing. Fix failures and repeat affected checks before approving the next stage.

### Repeat after cutover

After final synchronization and the traffic switch, repeat the critical data, application, connectivity, security, monitoring and performance checks against the live service. Passing isolated-copy tests alone does not establish that the live cutover succeeded.

## 3. Coordinated switch and rollback

**Decision:** prepare and test in phases, then switch connected applications and databases together. This refines the phased swing approach for tightly connected workloads; separate service cutovers require demonstrated independence.

### Switch sequence

1. Keep users on the old centre during copying and isolated testing.
2. Keep destination data synchronized using the selected replication method.
3. In an agreed window, pause writes where necessary and complete final synchronization.
4. Prevent the old environment from accepting writes, then switch users to the ready destination.
5. Repeat the critical validation checks and retain the protected source for recovery.

Testing must establish the expected interruption; uninterrupted service is not assumed.

### If the switch fails

- **Before switching users:** leave service at the old centre, fix the problem and test again.
- **After switching, without new destination writes:** fence the destination, return service to the old centre and verify operation.
- **After new destination writes:** stop destination writes, preserve and reconcile the new transactions back to the source before reopening it. If safe reconciliation is unavailable, keep writes stopped and follow the approved recovery procedure rather than reopening a stale source.
- Allow only one centre to accept writes for each data set at a time.

Before migration, agree which failures trigger rollback, who makes the decision and how long recovery may take. Rehearse rollback, including the case where the destination has accepted new writes.

## 4. Rehearse the switch and recovery

**Decision:** rehearse with the isolated copy before switching production users.

- Practice activating and validating the destination service.
- Simulate a failure and practice safe recovery to the source test environment.
- Include new test writes at the destination and verify that reconciliation preserves them during recovery.
- Keep the rehearsal isolated from production writes and external side effects.
- Record results and the time needed for synchronization, switching, checks and recovery. Fix failures and repeat affected tests before approval.

### Agreed rehearsal procedure

1. Prepare isolated source and destination test copies with matching data. Keep both separate from production writes and external side effects.
2. Pause test writes, complete synchronization, disable source test writes and direct test traffic to the destination.
3. Log in, run a representative application workflow and verify data and monitoring.
4. Create identifiable new test records at the destination.
5. Simulate failure by stopping a test application or disconnecting its test network.
6. Stop destination writes, reconcile the new records back to the source test copy and switch test traffic back.
7. Confirm that the new records survived, services work and only one side accepts writes. Record switching and recovery times.

If any check fails, correct the procedure and repeat the affected rehearsal before production approval.

## 5. Schedule and approve the real switch

**Decision:** after a successful rehearsal, agree the production window and obtain a final readiness decision.

- Choose a suitable time and communicate the expected interruption to affected users.
- Name the people executing the switch and the person authorized to approve, postpone or roll back.
- Agree failure thresholds and the latest safe rollback start, allowing time for reconciliation, recovery and validation within the approved service limits.
- Confirm that required staff and support are available.
- Immediately before execution, verify backup/recovery readiness, synchronization health, monitoring and completion of required tests.
- Record the responsible person's final **go or postpone** decision. Start only after approval.

### Choosing the window and checking readiness

- **Low business impact:** choose a period when interruption affects the fewest users and critical activities. Evenings or weekends may be suitable, but check payroll, month-end work, backups and batch jobs. Confirm engineer and supplier support availability. SOC monitoring needs continuous coverage regardless of the hour.
- **Time allowance:** use measured rehearsal times for switching, validation and recovery, including data reconciliation. Add a contingency margin and agree a latest safe rollback start. For example, 30 minutes to switch and validate plus 30 minutes to recover requires a window that also includes contingency; these figures are illustrative, not approved targets.
- **Responsibilities:** name the executor for each action and one decision authority for proceeding, postponing or rolling back.
- **User communication:** state the start time, affected services, expected interruption and where progress updates will appear.
- **Readiness checks:** confirm recoverable backups with successful restore evidence, replication without unresolved errors, working monitoring, passing required tests and available staff/support.
- **Decision:** proceed when the agreed checks pass; postpone if a required prerequisite fails. During execution, roll back when agreed failure thresholds or the latest safe rollback start are reached.

A low-impact window reduces the consequences of disruption; service and security requirements still apply throughout it.

## 6. Perform the production switch

**Decision:** execute the real switch only after the final readiness approval in part 5.

1. Pause new changes at the old centre where needed.
2. Complete final data synchronization and verify that it succeeded.
3. Prevent the old centre from accepting further writes.
4. Activate destination services in the tested dependency order and switch users to the new centre.
5. Check applications, data consistency, access, performance and SOC monitoring against the agreed acceptance criteria.
6. Continue at the new centre if checks pass. Follow the rollback procedure in part 3 if failure thresholds or the latest safe rollback start are reached.

Record actions, times, check results and decisions. Keep the old environment protected and available for recovery until the new environment has been accepted and its agreed recovery period has ended. Preserve a single authoritative writer for each data set throughout the switch.

## 7. Monitor and accept the new environment

**Decision:** observe and verify normal operation before approving the migration as complete.

- Monitor applications, performance, errors and SOC alerts during an agreed observation period.
- Confirm scheduled jobs and backups complete successfully.
- Ask service owners to verify that normal business tasks work correctly.
- Record results and resolve problems before formal acceptance.
- Keep the old environment protected throughout observation. Remove it only after formal acceptance and the agreed recovery period have ended.

## 8. Close the migration and retire the old environment

**Decision:** close the migration and retire the old workloads only after formal acceptance and the agreed recovery period have ended.

- Confirm the service owner's acceptance and authorization to retire the old workloads.
- Preserve required records and backups according to retention rules.
- Update system diagrams, inventories and recovery instructions to reflect the new centre.
- Remove temporary access, migration copies and obsolete configurations.
- Verify that no active service still depends on the old workloads, then securely retire them. Preserve shared infrastructure and anything still in use.
- Review lessons learned, assign any remaining improvement actions and close the change record.

### Example: closing a migrated application service

An application and its database now run at the destination centre. During the agreed observation period, users complete normal tasks, the SOC receives expected events, scheduled jobs succeed and destination backups complete. Recovery testing provides evidence that the destination backups are usable.

1. The service owner reviews the evidence and signs acceptance.
2. After the agreed recovery period ends, the responsible owner authorizes retirement of the old application and database VMs.
3. Operations checks that no users, integrations or jobs still depend on those VMs and that required records and recovery copies are retained under policy.
4. The team updates diagrams and recovery instructions, removes temporary migration permissions and replication jobs, and securely removes the obsolete VM copies through the approved platform/storage procedure.
5. The team records disposal evidence and lessons learned, then closes the change record. Other workloads and shared infrastructure at the old centre remain operational.

The observation and recovery periods are agreed in advance; this example does not prescribe a fixed duration.
