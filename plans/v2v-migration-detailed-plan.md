# NIL virtual-to-virtual migration: integrated plan

**Status:** proposal for review; production execution requires completed fields, named owners and approved gates.

**Team:** [name] · **Members:** [full names] · **Change ID:** [ID] · **Service owner:** [name] · **Window:** [date/time/time zone]

## 1. Scope and standards

Migrate virtual servers from [source environment] to [target environment], maintaining agreed service quality, information security and business continuity. Physical relocation is outside scope. Plan and validate whole services and their dependencies, rather than treating each VM as an independent workload.

This plan uses all three PDFs currently present:

| Source | Edition shown in supplied PDF | Role in this plan |
|---|---|---|
| `document.pdf` | SIST EN ISO/IEC 27001:2017, based on ISO/IEC 27001:2013, with corrections | Security scope, risk assessment, treatment, controls and evidence |
| `document (1).pdf` | SIST EN ISO 22313:2020 | Guidance for business impact analysis, continuity strategies, response, recovery and exercises |
| `document (2).pdf` | SIST EN ISO 9001:2026 | Customer requirements, controlled execution, supplier quality, release and corrective action |

Clause references follow these supplied editions. ISO 22313 provides guidance on using ISO 22301; it is not a substitute for the ISO 22301 requirements document. The operational measures below are proposed applications, not prescribed migration commands or a declaration of certification. Confirm alignment with NIL’s applicable management-system editions before approval.

## 2. Governance and decision rights

Assign a named person and backup to each role. One person may hold multiple roles where appropriate, but retain independent review of critical changes.

| Role | Accountability |
|---|---|
| Sponsor / service owner | Approves scope, customer commitments, resources and final service acceptance |
| Change manager | Coordinates execution and authorizes go/no-go, rollback and closure |
| Business continuity lead | Owns impact analysis, recovery priorities, continuity activation and return to normal |
| Security / risk owner | Approves security risk treatment and accepts residual risks within delegated authority |
| Virtualization engineer | Validates platform support, prepares migration and executes VM tasks |
| Application / database owner | Controls writes, verifies consistency and reconciles data during recovery |
| Network / security engineer | Controls routing, DNS, segmentation, access and traffic changes |
| SOC / operations | Maintains monitoring, investigates alerts and operates the accepted target |
| Quality reviewer | Checks evidence, acceptance criteria, deviations and release authorization |
| Supplier coordinator | Confirms provider responsibilities, support availability and escalation commitments |

Keep the runbook, contacts and recovery instructions available through a protected channel independent of the systems being migrated. Never place credentials in this plan.

**References:** 27001 5.3, 7.2, 7.4–7.5; 22313 7.2, 7.4, 8.4.2–8.4.4; 9001 5.3, 7.2, 7.4–7.5.

## 3. Phase A — discover requirements and business impact

### A1. Inventory and map dependencies

For every service record:

- Customer, business process, service owner, VM names/IDs, OS and application versions, data classification and support status.
- Hypervisor versions, firmware mode, virtual disk formats/controllers, guest tools, encryption keys, licensing and supported conversion path.
- CPU, RAM, storage capacity/latency/IOPS, network bandwidth and measured peak demand.
- Databases, queues, file shares, directory services, DNS, NTP, certificates, APIs, scheduled jobs, backup and SOC feeds.
- Source/target subnets, VLANs, IP requirements, firewall rules, routes, load balancers and any external allowlists.
- Providers and shared failure domains: storage, identity, management plane, connectivity and backup repositories.

Confirm whether special workloads such as domain controllers, clustered databases or licensing servers require application-specific procedures. Do not assume a generic VM clone is safe for every workload.

### A2. Business impact analysis (BIA)

Assess operational, financial, contractual, reputational and other applicable impacts over increasing outage durations, including peak periods. Agree which activities must resume first and at what capacity.

| Service / activity | Impact over time | MTPD | Minimum capacity (MBCO) | RTO | RPO | Dependencies / owner |
|---|---|---|---|---|---|---|
| SOC monitoring | [assess] | [approve] | [feeds, detection and response capacity] | [approve] | [approve] | [complete] |
| Customer service group | [assess] | [approve] | [minimum transactions/users] | [approve] | [approve] | [complete] |

- **MTPD:** time after which disruption causes unacceptable impact.
- **MBCO:** minimum acceptable business service capacity during disruption.
- **RTO:** target time to restore the agreed capacity; allow a margin before MTPD.
- **RPO:** maximum acceptable data loss expressed as time before disruption.

Agree planned cutover interruption separately from disaster-recovery targets. Do not invent numerical targets: the service owner approves them based on customer commitments and the BIA. Include dependency recovery, validation and backlog processing in the recovery timeline.

### A3. Quality and customer requirements

Record acceptance criteria before selecting the solution: permitted interruption/data loss, transaction success, latency, throughput, error rate, security controls, monitoring coverage and support arrangements. Confirm relevant contracts, data-location/privacy obligations and required customer notices with their responsible owners. Review changes to requirements and communicate them to affected teams.

**Gate A — scope approved:** inventory, BIA, dependencies, customer requirements and measurable acceptance criteria approved; unknowns have owners and due dates.

**References:** 27001 4.1–4.3, 6.2, A.8.1.1, A.18.1.1; 22313 8.2; 9001 4.4, 6.2, 8.2.

## 4. Phase B — assess risks and design the migration

### B1. Risk register

Define likelihood/impact scales, prioritization rules and risk-acceptance authority. Assess confidentiality, integrity and availability risks alongside continuity and service-quality risks.

| Risk | Proposed treatment | Evidence before cutover |
|---|---|---|
| Data inconsistency or lost writes | Application-consistent replication/backup, controlled final sync and single writer | Restore and consistency test |
| Duplicate identities, IPs or writers | Isolated testing and explicit fencing of inactive instances | Isolation and fencing checks |
| Unsupported conversion / boot failure | Vendor-supported path; test firmware, drivers, guest tools and licensing | Successful representative rehearsal |
| Target overload | Peak-load baseline, headroom and capacity testing | Approved performance results |
| Security weakened during transfer | Protected transfer, encryption as justified, least privilege and logged administration | Access and transfer checks |
| SOC visibility lost | Preserve independent monitoring and validate all required feeds | End-to-end test alert received |
| Rollback fails after new writes | Rehearse reverse synchronization/reconciliation and define a recovery path | Timed recovery test |
| Common failure disables both environments | Review shared storage/network/control dependencies and backup independence | Dependency/failure review |
| Supplier unavailable | Agreed escalation, staffing, support and fallback arrangements | Provider confirmation |

For each risk add owner, initial score, action/deadline, residual score and acceptance. Record opportunities such as improved monitoring or recoverability without expanding the cutover beyond approved scope.

### B2. Select the technical approach

| Approach | Use when | Main condition |
|---|---|---|
| Supported live migration | Platforms and workload support it end to end | Rehearsal proves service-level targets and rollback feasibility |
| Replication plus controlled cutover | Cross-platform move or short interruption permitted | Final consistency checkpoint and tested recovery of new writes |
| Application-native HA / replication | Service must continue or workload requires application-aware migration | Tested failover, quorum/fencing and application consistency |
| Backup/restore or offline conversion | Approved interruption permits it | Measured restore/conversion time fits the agreed window |

Do not promise zero downtime from a VM copy alone. For continuous service, retain functioning service capacity while migrating components and prove the application-level failover path. Keep SOC detection and response available throughout.

Define service groups and migration order from the dependency graph. Recover dependencies before dependants; stop applications in the workload-appropriate reverse order. Start with a low-risk representative pilot, then migrate controlled waves.

### B3. Control selection and suppliers

Update the 27001 Statement of Applicability: necessary controls, inclusion reasons, implementation status and reasons for Annex A exclusions, using the PDF’s correction. Record supplier scope, supported methods, acceptance deliverables, access limits, data handling and escalation times. Evaluate provider capability and review delivered results.

**Gate B — design approved:** method is supported, resources funded, continuity targets feasible, risk treatment agreed and residual risks accepted by their owners.

**References:** 27001 6.1.2–6.1.3, 8.2, A.9, A.12, A.13, A.15, A.17; 22313 8.2.3, 8.3; 9001 6.1, 6.3, 8.3 where applicable, 8.4.

## 5. Phase C — build, protect and rehearse

1. **Prepare the target:** capacity, storage, segmentation, routes, time synchronization, identity, certificates, patching, hardening, endpoint protection and least-privilege administration.
2. **Preserve recovery:** take application-consistent backups of data/configurations and required keys; verify access and perform restoration. Keep recovery copies protected from failure of the migration environment. A snapshot alone is not an independent backup.
3. **Set up observability:** application probes, service metrics, system logs, administrator activity and SOC feeds. Verify alert delivery and time correlation before migration.
4. **Stage replication/conversion:** use the supported tool; measure transfer duration and lag. Keep copied systems isolated from production identity/network conflicts, jobs and outbound integrations.
5. **Rehearse the service:** test boot, authentication, critical transactions, database checks, permissions, integrations and representative peak performance. Use protected test data and prevent accidental customer traffic.
6. **Exercise disruption:** simulate failed cutover, lost connectivity, unavailable dependency and rollback after target writes. Test decision authority, communications and access to recovery instructions.
7. **Review results:** record expected/actual results, timings and defects. Assign corrective actions; retest affected scenarios after correction.

### Minimum test record

| Test | Required result | Actual result / evidence | Owner | Status |
|---|---|---|---|---|
| Restore | Recover agreed service/data within approved targets | [link] | [name] | [pass/fail] |
| Business transaction | Agreed critical workflows succeed | [link] | [name] | [pass/fail] |
| Data consistency | Workload-specific checks and reconciliation pass | [link] | [name] | [pass/fail] |
| Security / SOC | Access restrictions effective; required logs and test alerts received | [link] | [name] | [pass/fail] |
| Performance | Approved peak-load limits met | [link] | [name] | [pass/fail] |
| Failover / rollback | Recovery, fencing and data reconciliation meet agreed targets | [link] | [name] | [pass/fail] |

**Gate C — ready to migrate:** required tests pass, critical defects are closed, suppliers/staff are available, backup and monitoring are healthy, and the change manager authorizes the window. Any permitted deviation requires recorded authority and customer approval where applicable; it must not bypass mandatory obligations.

**References:** 27001 7.2, 7.5, 8.1, A.12.3–A.12.4, A.17; 22313 8.3.5, 8.5–8.6; 9001 7.1–7.2, 8.1, 8.5.1, 8.6–8.7.

## 6. Phase D — cutover runbook

Before execution replace each task with platform-specific, reviewed instructions, expected output, named executor and estimated duration. Validate commands against vendor documentation for the actual versions. Record all execution times and decisions.

**Time control:** define latest rollback start as the approved recovery deadline minus rehearsed rollback/reconciliation/validation duration and contingency. Start rollback before that point if completion cannot be demonstrated.

| Step | Action | Stop / acceptance checkpoint |
|---|---|---|
| 1 | Open the change bridge; confirm approvals, staff, contacts, backups, replication, monitoring and target health. Freeze unrelated changes. | Change manager records GO |
| 2 | Confirm pilot/wave dependency readiness and baseline probes. Notify affected users/providers as agreed. | No unresolved prerequisite |
| 3 | Drain traffic and pause jobs/writes where the chosen method requires it. Record last consistent application/database checkpoint. | Application owner confirms consistency |
| 4 | Finish synchronization and verify lag/completion against the approved RPO. | Data checkpoint verified |
| 5 | Fence source workloads as required; prevent duplicate IP/identity or writable instances. | One authoritative writer per data set |
| 6 | Activate target dependencies and applications in the tested sequence. | Infrastructure and application health pass |
| 7 | Switch traffic using rehearsed routing/load-balancer/DNS steps. Account for DNS caches; retain protection against old endpoints accepting writes. | End-to-end client path verified |
| 8 | Run critical transactions, integrity, permissions, performance and SOC checks. | Service meets approved minimum capacity and criteria |
| 9 | Resume jobs and normal traffic gradually; monitor throughout the agreed observation period. | Owner approves continued target operation |

For supported live migration use the vendor workflow in place of unnecessary write pauses, while retaining consistency, security and service checks. Do not convert sequential cold moves into a claim of uninterrupted service.

**Communication events:** window start; go/no-go; impact or incident; rollback; service restored; closure. Define recipients, channel, sender and update cadence in advance. Escalate to the continuity response structure when recovery targets or critical business activities are threatened.

**References:** 27001 8.1, 8.3, A.12.1.2, A.16; 22313 8.4.2–8.4.5; 9001 8.5.1–8.5.2, 8.5.6, 8.6.

## 7. Rollback and continuity response

**Activation triggers:** failed critical transactions, suspected corruption, loss of essential security/SOC controls, performance outside approved limits, or reaching the rollback decision deadline. Define measurable thresholds and observation intervals before execution.

1. Change manager stops the wave and declares rollback or continuity activation. Notify the service owner, SOC and required customer/provider contacts.
2. Stop target writes and fence it. Capture logs and checkpoints; determine whether new transactions were accepted.
3. **No target writes:** restore the source as authoritative and reverse traffic using the rehearsed path.
4. **Target writes exist:** use tested reverse replication/reconciliation before reopening source writes. A stale source restart can lose transactions. If safe reconciliation is unavailable, keep writes stopped and invoke the approved alternate recovery solution.
5. Restore dependencies and prioritized activities to the agreed minimum capacity. Validate data, access, SOC visibility and transactions before reopening service.
6. Record actual outage/data loss, affected customers, decisions and evidence. Maintain updates until stable service is confirmed.
7. Investigate cause, correct the issue and retest before rescheduling. Preserve failed outputs/evidence and prevent unintended reuse of the failed target.

Rollback restores the previous environment; continuity response may instead sustain service on an approved alternate platform. Test both where the risk assessment requires them.

**References:** 27001 A.16–A.17, 10; 22313 8.4–8.6; 9001 8.7, 10.2.

## 8. Phase E — acceptance, stabilization and closure

The service owner approves the observation duration based on risk and service cycles. Keep source and recovery options protected until the agreed retirement gate.

### Release and acceptance checklist

- Critical customer workflows and integrations pass agreed tests.
- Data consistency is verified using workload-appropriate checks; all accepted transactions are accounted for.
- Actual interruption, recovery time and data loss meet approved criteria.
- Target capacity/performance meets the measured service requirements.
- Access, segmentation, encryption where required, backups, logs and SOC alerts operate correctly.
- Operations has procedures, support access, training and ownership; suppliers have completed agreed deliverables.
- Service owner and release authority sign the evidence record. Record any concession, its authority and required customer agreement.

After acceptance, monitor through the agreed stabilization period, review customer feedback and verify scheduled jobs and backup cycles. Retire the source only after recovery, retention and disposal requirements are approved. Securely remove unneeded copies, temporary access/rules and obsolete configuration; retain required evidence.

Update inventory, architecture/dependency diagrams, continuity and recovery procedures, risk register and applicability decisions. Hold a lessons-learned review; record root causes, corrective actions, owners and effectiveness checks. Feed results into relevant internal audits and management reviews.

**References:** 27001 7.5, 9–10, A.8.1.1, A.11.2.6; 22313 8.4.5, 8.6, 9–10; 9001 8.5.5, 8.6–8.7, 9–10.

## 9. Evidence pack and decision

Keep one controlled evidence index linking: approved requirements/BIA; inventory/dependencies; risks and treatment approvals; Statement of Applicability; supplier agreements; design/runbook versions; backup/restore and exercise results; cutover/incident logs; release sign-off; actual service metrics; retirement approval; corrective actions. Restrict access and retain records under the agreed policy.

**Current planning recommendation: GO WITH CONDITIONS.** Production cutover remains unapproved until Gates A–C pass and named authorities authorize it. Redesign if rehearsal cannot demonstrate required continuity, consistency or recovery; defer when evidence or resources are missing.

Three immediate next actions:

1. Complete the service inventory and BIA with NIL’s owners; approve interruption, recovery and data-loss targets.
2. Select the supported migration/continuity method and complete the risk treatment and acceptance criteria.
3. Rehearse a representative pilot, including restore and rollback after new writes, and review evidence before scheduling production.
