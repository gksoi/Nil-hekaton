# Canary Migration — vulnerabilities against ISO 9001

**Reviewed:** 13 November 2026. **Plan:** [canary-migration-plan.md](canary-migration-plan.md). **Normative baseline:** [document (2).pdf](../document%20%282%29.pdf), SIST EN ISO 9001:2026 as identified in the supplied file. References follow that edition.

## Scope and assessment method

This review examines all eight plan stages for weaknesses in consistent, conforming service delivery. It is a document gap assessment, not a security scan or certification audit. Missing information may exist in NIL's wider QMS; it must be supplied or linked before claiming a requirement is met. No actual metrics, contracts, configurations or test evidence were inspected.

Technical remedies below are implementation recommendations tied to quality requirements. ISO 9001 does not prescribe canary traffic percentages, minimum sample counts, a shared database or a particular routing product. Do not mistake engineering recommendations for verbatim standard requirements.

**Priorities:** P1 = resolve or demonstrate effective existing control before the relevant production exposure/promotion; P2 = complete before acceptance/closure, or earlier where needed. These are review priorities, not formal audit grades. Findings remain open pending evidence.

## Overall conclusion

The plan already acknowledges stable cohorts, per-cohort metrics, observation volume, shared-data compatibility and a separate database move. These are substantial strengths. The main vulnerability is that a small traffic percentage is not necessarily a small impact: all users still depend on the shared database and other common services. Each canary promotion and the later database handover need separate quality evidence.

## Findings and remedies

### CA-01 — Shared-state risk can exceed the traffic cohort · P1

**Plan evidence:** section 1 explicitly uses one authoritative database; section 3 warns about corruption. **Gap:** there is no concrete restriction/test showing canary writes, expensive queries or background jobs cannot harm all users.

**Failure scenario:** 5% of users trigger a locking query or incompatible update that breaks the shared service for everyone. **ISO basis:** 6.1.2, 8.1, 8.5.1, 8.5.3–8.5.4, 8.7.

**Fix / owner:** database/application owners test concurrency, locks, resource contention, transaction behavior and permissions; set approved resource/operational limits and contain unsafe writes. Document the effect on non-canary users. **Close with:** shared-state fault/load tests and a containment drill proving the agreed service limits; do not claim the blast radius equals the routed percentage.

### CA-02 — Promotion criteria lack an executable measurement design · P1

**Plan evidence:** sections 2–3 require absolute/comparative limits, minimum time and transaction volume. **Gap:** actual definitions, workload coverage, comparison groups and missing-data behavior are not supplied.

**Failure scenario:** an idle or unrepresentative canary looks healthy and is promoted despite untested customer workflows. **ISO basis:** 6.2, 7.1.5.1, 8.1, 8.6, 9.1.1.

**Fix / owner:** quality/operations owners define metric source, percentile or other measure, error denominator, observation interval, minimum representative transactions and explicit hold/abort behavior when evidence is insufficient. Account for customer/workload differences between cohorts. **Close with:** approved stage scorecards and verified telemetry using known failures. There is no universal ISO sample count; justify adequacy for this service. [S2] supports staged health checks.

### CA-03 — Cohort and session correctness need proof · P1

**Plan evidence:** sections 1–2 request stable cohorts and session checks. **Gap:** no deterministic routing rule, tenant boundary, request trace or behavior for retries/session expiry is specified.

**Failure scenario:** one user's workflow switches environments mid-session, or a tenant not in the pilot reaches the destination. **ISO basis:** 8.1, 8.5.1–8.5.2, 8.5.6, 8.7.

**Fix / owner:** network/application owners define selection and stickiness, applicable tenant boundaries and traceable request/cohort IDs. Test login/logout, long-lived connections, retries and routing failure. **Close with:** routing tests demonstrating only the intended cohort is exposed and transaction/session correctness survives reassignment; identify the routing configuration version in release evidence.

### CA-04 — Customer requirements and pilot communication are incomplete · P1

**Plan evidence:** section 5 says to tell affected users about the pilot where appropriate. **Gap:** no reviewed customer requirements, specific notification obligations, complaint handling or authorization for differential service exposure is linked.

**Failure scenario:** a customer participates in a pilot contrary to their contract or without required contingency information. **ISO basis:** 5.1.2, 8.2.1–8.2.4, 9.1.2.

**Fix / owner:** service owner reviews contractual/intended-use requirements and applies required notifications/approvals. Establish a feedback channel and correlate complaints to cohort/stage. **Close with:** reviewed requirement-to-test records, communication decisions and evidence; ISO 9001 does not universally require individual pilot consent, so determine actual obligations rather than assume them.

### CA-05 — Application compatibility and rollback boundary are not versioned · P1

**Plan evidence:** section 3 requires source readability and forbids casually rolling back the shared database. **Gap:** there is no compatibility matrix for application versions, schema, API/event formats or state written during canary operation.

**Failure scenario:** the destination writes a value the source application cannot interpret, so reversing traffic does not restore the service. **ISO basis:** 8.3.3–8.3.6 where design applies; 8.5.4, 8.5.6, 8.7 in operation.

**Fix / owner:** application/database owners identify supported version combinations and destructive-change boundaries. Keep schemas compatible during coexistence; defer irreversible changes until the recovery decision is approved. **Close with:** tests in which both application versions read/process destination-written state, and a recovery drill after new transactions. If code/schema is unchanged, document that baseline and still test configuration-dependent behavior. [S3] supports compatibility reasoning.

### CA-06 — The final database move needs its own release record · P1 before database handover

**Plan evidence:** section 6 explicitly says 100% application traffic is not complete and provides database handover steps. **Gap:** writers beyond the application, queue/file dependencies, data checkpoint procedure and database-specific approval are not instantiated.

**Failure scenario:** successful canary results are treated as proof for an untested whole-database switch while a batch job continues writing to the old database. **ISO basis:** 8.1, 8.5.1–8.5.2, 8.5.6, 8.6–8.7.

**Fix / owner:** change/database owners create a separately reviewed handover stage identifying every writer, synchronization evidence, fencing, connection update, acceptance test and rollback deadline. **Close with:** timed full handover/reconciliation rehearsal and signed release record. Canary app metrics cannot substitute for database-migration validation.

### CA-07 — Source fallback and cross-centre failure need quantified testing · P1

**Plan evidence:** sections 2, 4 and 5 request source fallback capacity and connectivity-failure rehearsal. **Gap:** load, source scale-down restrictions, network behavior and resulting service limits are undefined.

**Failure scenario:** a destination failure sends all traffic back to a source that can no longer sustain it, or link failure strands destination users away from the authoritative database. **ISO basis:** 6.1.2, 7.1.3, 8.1, 8.5.1.

**Fix / owner:** operations retains verified fallback capacity and tests 100% traffic return, link loss/latency, connection pools and retries. Review shared dependencies. **Close with:** measured failback/load results within approved limits and a decision on how long source capacity is retained.

### CA-08 — Promotion approvals and configuration history need controlled evidence · P1

**Plan evidence:** sections 3 and 5 require approval at every increase. **Gap:** no controlled record links each decision to cohort membership, traffic weight, build/schema version, metrics or the approving person.

**Failure scenario:** a later operator cannot determine which customers received which configuration or why a failing stage was promoted. **ISO basis:** 7.5.2–7.5.3, 8.5.2, 8.5.6, 8.6.

**Fix / owner:** change manager retains a stage ledger with exact configuration, start/end times, criteria/results, deviations and authorizing identity. Separate authorization for first exposure, each promotion and the later database switch. **Close with:** reviewed example ledger and actual records at execution; define access, version control and retention, including externally sourced runbooks.

### CA-09 — Resources, skills and provider quality need more than role names · P1 as applicable

**Plan evidence:** section 5 requests owners and staffed windows. **Gap:** no competence evidence, standby coverage or evaluation/verification of routing, connectivity and hosting providers is referenced.

**Failure scenario:** a routing/provider fault cannot be diagnosed during a small stage, or supplier limits invalidate the test. **ISO basis:** 5.3, 7.1.2–7.1.3, 7.1.6, 7.2–7.4, 8.4.1–8.4.3.

**Fix / owner:** technical/supplier leads confirm competent executors, backups, training effectiveness and provider requirements/output verification. Make rollback knowledge available independently of affected systems. **Close with:** skill/rehearsal records, assignments, communications matrix and provider evaluation/acceptance evidence where external provision is relevant.

### CA-10 — Incident disposition and post-migration improvement are underspecified · P2; containment required before first exposure

**Plan evidence:** sections 3 and 8 contain unsafe behavior and record lessons learned. **Gap:** there is no explicit nonconformity record, customer-property notification path, concession/retest process, complaint analysis or effectiveness check after correction.

**Failure scenario:** a canary defect is fixed locally but corrupt records remain, or the same fault recurs at the next stage. **ISO basis:** 8.5.3, 8.5.5, 8.7, 9.1.2–9.1.3, 10.1–10.2.

**Fix / owner:** incident/quality owners track affected cohort and wider shared-state effects, customer notifications, containment, correction, disposition authority, retest, root cause and recurrence checks. Define acceptance/retention/decommission evidence and feed significant results to existing QMS reviews. **Close with:** demonstrated incident workflow and completed/assigned follow-up records. Traffic rollback does not itself prove affected data is corrected.

## Wider clause coverage and applicability

| Clauses | Assessment of the written plan | Required linkage |
|---|---|---|
| 4.1–4.4 | Migration purpose exists; broader context, interested parties and QMS interactions are outside this outline. | Applicable QMS scope/process map and context review, including organizational climate-relevance determination. No physical move is implied. |
| 5.1–5.3 | Execution authority is requested, not evidenced; policy/leadership are organizational. | Relevant quality policy, sponsor authorization and assigned authorities. |
| 6.1–6.3 | Risks/targets are requested; opportunity actions, effectiveness and planned QMS-change consequences are not linked. | Risk/opportunity evaluation, measurable objectives and relevant change records. 8.5.6 separately controls service changes. |
| 7.1–7.5 | Capacity, telemetry and staff are mentioned; suitability, competence and controlled records remain incomplete. | CA-02, CA-07–CA-09; awareness and suitable working conditions under 7.3 and 7.1.4. |
| 8.1–8.2 | Good staging outline; requirements review and executable criteria remain gaps. | CA-02 and CA-04. |
| 8.3 | Applicability needs a recorded decision. | If service behavior/architecture is designed or changed, retain design inputs, reviews, verification, intended-use validation, outputs and change approvals. An unchanged application still needs operational compatibility evidence. |
| 8.4–8.7 | Supplier, coexistence, release and incident handling need instantiated controls. | CA-01, CA-03, CA-05–CA-10. |
| 9.1–9.3, 10 | Per-cohort monitoring is strong conceptually; analysis, feedback and QMS review/effectiveness evidence remain external. | CA-02 and CA-10; link organizational internal audits/management review rather than assume a new audit per stage is mandatory. |

## Recommended disposition

**Conditional planning option; not production approval.** Before pilot exposure, demonstrate safe shared-state coexistence, reviewed customer requirements, reliable stage criteria and recovery. Before database cutover, independently close CA-06. Decide readiness from completed evidence, not a traffic percentage or elapsed time alone. Reference existing QMS controls where effective rather than duplicate records.

## Sources and research limits

- **S1 — Normative:** supplied `document (2).pdf`, clauses 4–10, printed pages 5–24 (PDF pages 21–40). [ISO's quality-management overview](https://www.iso.org/standards/popular/iso-9000-family) supplies general context only; the PDF controls the clause mapping.
- **S2 — Technical:** [Microsoft safe-deployment guidance](https://learn.microsoft.com/en-us/azure/well-architected/operational-excellence/safe-deployments) supports progressive exposure and health-gated promotion. The specific cohort/measurement controls in CA-02–CA-03 are reviewer recommendations for this plan.
- **S3 — Technical:** [AWS blue-green data/schema compatibility guidance](https://docs.aws.amazon.com/whitepapers/latest/blue-green-deployments/best-practices-for-managing-data-synchronization-and-schema-changes.html) informs the shared-data compatibility concern in CA-05; it does not mandate an AWS platform or this canary design.

Actual provider/platform support, production metrics, customer impact and certification status remain unverified.


---

### Vulnerabilities according to SIST EN ISO 22301:2020 — Security and resilience — Business continuity management systems — Requirements (ISO 22301:2019)

**Additional assessment:** this section supplements the ISO 9001 review above; its earlier scope exclusions apply to that original review only. The normative source here is `document (5).pdf`, **SIST EN ISO 22301:2020**, adopting ISO 22301:2019. The full title is shown in the heading. “223091” is treated as a typographical error, not a different standard.

ISO 22301 contains BCMS requirements; ISO 22313:2020 supplies guidance and does not replace them. This comparison follows the supplied 2019-based text. Later amendments are not silently incorporated. Clause numbers below belong to ISO 22301; in particular, **10.1 covers nonconformity/corrective action and 10.2 continual improvement**.

These are gaps in a written migration plan, not confirmed operating failures or certification findings. Existing controlled BCMS evidence may close a gap. The proposed technical solutions are examples, not mechanisms prescribed by the standard. Priorities retain the definitions above; several P2 controls need definition before activation even though their execution occurs at closure. Related ISO 9001 and ISO 22301 findings may describe the same weakness from different requirements and should not be added as independent risk scores.

**What is already useful:** the plan contains preparation/testing, controlled change, recovery considerations and delayed retirement. What must additionally be demonstrated is continuation/recovery of prioritized business activities at predefined capacity and within BIA-derived time frames, including disruption during migration. A working VM alone is not sufficient evidence. RPO/data-loss criteria should be set where relevant to the chosen solution; no universal zero-data-loss target is inferred from ISO 22301.

#### BCCA-01 — Traffic percentages do not define acceptable business impact · P1

**Plan evidence:** Sections 3 and 5 require stage metrics, samples and time limits. **Gap:** No BIA links cohorts and business activities to unacceptable-impact deadlines and minimum acceptable capacity.

**Disruption scenario:** A small cohort contains a critical customer or rare high-impact workflow and breaches continuity needs despite low overall error rates. **ISO 22301 basis:** 8.2.1–8.2.2; 4.2; 6.2.

**Fix / owner:** Service/continuity owners map relevant activities and customer obligations to cohort selection, recovery priorities and capacity/time targets. Approve impact-based stage gates, not only percentages.

**Close with:** Reviewed BIA and stage requirements covering critical/rare workflows and the permitted impact on all affected users.

#### BCCA-02 — Shared data breaks the assumption of a small disruption radius · P1

**Plan evidence:** Sections 1 and 3 correctly use one authoritative database and warn about corruption. **Gap:** No evaluated disruption scenario proves canary queries, writes or jobs cannot exhaust or corrupt shared production resources.

**Disruption scenario:** A 5% destination cohort blocks the shared database and disrupts the entire business service. **ISO 22301 basis:** 8.2.3; 8.3.1–8.3.3.

**Fix / owner:** Architecture/database owners assess shared-state failure, select containment/continuity solutions and justify them by BIA, risk tolerance, cost and benefit. Include non-canary users in the impact analysis.

**Close with:** Shared-resource fault/load evidence and a demonstrated minimum-service recovery path for the whole affected service.

#### BCCA-03 — Cross-centre coexistence needs a continuity solution for link loss · P1

**Plan evidence:** Sections 2 and 4 check cross-centre latency and connectivity failure. **Gap:** No actual degraded behavior, timeout/retry policy or resource plan shows minimum service if destination apps lose the source database.

**Disruption scenario:** Destination users cannot transact and retries overload the remaining healthy source. **ISO 22301 basis:** 8.3.2–8.3.5; 8.4.4.

**Fix / owner:** Network/application owners implement a tested link-failure solution, such as controlled rerouting to verified source capacity. Avoid opening independent writable database copies as a shortcut.

**Close with:** Timed link-loss recovery with stable data ownership, acceptable retry effects and business capacity restored within target time.

#### BCCA-04 — Returning a cohort does not repair shared corrupted state · P1

**Plan evidence:** Section 3 says traffic reversal is insufficient if destination behavior corrupts data. **Gap:** The referenced repair/recovery procedure is not instantiated for data used by both cohorts.

**Disruption scenario:** Traffic returns to source but source users continue reading damaged records; restoring an old full backup erases valid work from both cohorts. **ISO 22301 basis:** 8.3.3–8.3.5; 8.4.1; 8.4.4–8.4.5.

**Fix / owner:** Database/continuity owners define containment, selective reconciliation or another supported recovery method, plus minimum-capacity alternatives when data cannot be repaired immediately.

**Close with:** Exercise preserving valid transactions from both cohorts while isolating/repairing the fault and measuring actual service recovery.

#### BCCA-05 — Final database migration is a separate whole-service disruption · P1 before database handover

**Plan evidence:** Section 6 explicitly moves the database after application traffic reaches 100%. **Gap:** No completed BIA/strategy refresh shows the independent database window, all writers and recovery constraints remain within continuity needs.

**Disruption scenario:** The canary appears successful, but the later database promotion interrupts every user beyond the agreed limit. **ISO 22301 basis:** 8.2.1; 8.2.2; 8.3; 8.4.4; 8.5–8.6.

**Fix / owner:** Change/database/continuity owners treat the data move as a separately assessed, resourced and approved recovery stage. Include external writers, checkpoints, fencing, connection updates and post-write rollback.

**Close with:** Separate timed database handover/recovery exercise and evidence meeting its business time/capacity targets; canary app metrics cannot substitute.

#### BCCA-06 — Stage approval is not a formal continuity response structure · P1

**Plan evidence:** Sections 3 and 5 require authority at every promotion. **Gap:** No incident-impact activation threshold, named alternate or coordination with SOC/business continuity teams is linked.

**Disruption scenario:** A promotion owner halts traffic growth but no one activates a wider response to an outage affecting both cohorts. **ISO 22301 basis:** 5.3; 7.2–7.3; 8.4.2.

**Fix / owner:** Continuity lead defines who assesses impact, stops/reroutes cohorts, declares the disruption, activates continuity and coordinates teams. Name competent alternates and priorities.

**Close with:** Activation/escalation drill with shared-service failure and primary decision-maker unavailable.

#### BCCA-07 — Pilot communication must expand to all affected stakeholders · P1

**Plan evidence:** Section 5 informs affected pilot users and defines support/escalation channels. **Gap:** A shared-resource incident may affect customers outside the cohort; procedures for incoming reports and surviving communications are not supplied.

**Disruption scenario:** Only pilot users receive updates although the database outage affects everyone. **ISO 22301 basis:** 7.4; 8.4.3; 8.4.4.3.

**Fix / owner:** Communications owner defines impact-based recipient expansion, warning triggers, inbound handling, backup channels and provider coordination. Link applicable wider authority/emergency/media procedures.

**Close with:** Exercised alert/update procedure for both canary and non-canary customers, including loss of the primary channel.

#### BCCA-08 — Fallback capacity and provider continuity must remain maintained · P1 as applicable

**Plan evidence:** Sections 2, 4 and 5 require source capacity for all traffic and support availability. **Gap:** No maintained resource commitments or provider capability evaluation protects fallback as traffic moves away from source.

**Disruption scenario:** Source is scaled down early or a routing/provider quota prevents rapid full traffic return. **ISO 22301 basis:** 7.1–7.2; 8.1; 8.3.4–8.3.5; 8.6(c).

**Fix / owner:** Operations/supplier owners retain verified source capacity and recovery staff/access until the approved gate, evaluate relevant provider continuity and test the required limits.

**Close with:** Full-traffic failback load results, supplier capability evidence and documented retention of recovery resources.

#### BCCA-09 — Rehearsal and promotion monitoring need programme-level evaluation · P1

**Plan evidence:** Section 4 rehearses progression, reversal and later data handover; sections 3 and 7 monitor stages. **Gap:** No continuing exercise/review programme, formal post-exercise report or change-driven revalidation is linked; data-only or quiet-traffic periods may hide missing business capability.

**Disruption scenario:** Traffic metrics pass while critical transactions are absent or a later change invalidates fallback. **ISO 22301 basis:** 8.5–8.6; 9.1; 7.5.

**Fix / owner:** Continuity/operations owners verify both stage and full-service capability against BIA targets; exercise multiple failure scenarios over time and test communications. Define measurement reviewers and insufficient-evidence hold rules.

**Close with:** Formal exercise reports, valid business-capacity/time measurements, improvement actions and scheduled/significant-change revalidation.

#### BCCA-10 — Final acceptance lacks an explicit stand-down and BCMS improvement path · P2; define before activation

**Plan evidence:** Sections 7–8 accept the service, remove source dependencies and record lessons. **Gap:** No explicit process restores normal operation from temporary cohort/routing measures, reconciles backlog and feeds continuity outcomes to BCMS review.

**Disruption scenario:** A temporary fallback or incomplete data ownership remains after the migration is called complete. **ISO 22301 basis:** 8.4.4.3(h); 8.4.5; 8.6; 9.2–9.3; 10.1–10.2.

**Fix / owner:** Service/continuity owners document stand-down and normal capacity/ownership checks, evaluate actual impact, update BIA/risks/plans and track cause/correction/effectiveness through existing BCMS reviews.

**Close with:** Stand-down record, completed dependency/data handover evidence and corrective-action effectiveness review; no new certification audit is implied.


#### Wider BCMS coverage and applicability

| Clauses | Required evidence or organizational linkage |
|---|---|
| 4.1–4.4 | Applicable context, interested parties, documented/up-to-date legal and other continuity requirements, justified BCMS scope/exclusions and process interactions. A project migration scope is not the complete BCMS scope. |
| 5.1–5.3 | Leadership commitment/resources, communicated continuity policy and authority for BCMS conformity/performance reporting. Existing organizational records may satisfy these. |
| 6.1–6.3 | Risks/opportunities concerning BCMS effectiveness, objectives with actions/resources/owners/dates/evaluation, and planned BCMS changes. Business disruption risk is separately assessed under 8.2.3. |
| 7.1–7.5 | Available resources, competence/awareness, communications and controlled, approved plans accessible during disruption, with version/access/retention rules. |
| 8.1–8.2 | Controlled migration/supply-chain processes; maintained BIA and disruption risk assessment reviewed at planned intervals and significant changes. |
| 8.3 | Solutions selected from BIA/risk outputs, considering time/capacity, risk tolerance and costs/benefits; required resources and actual maintained readiness. |
| 8.4 | Response structure, warnings, usable action plans, activation thresholds, minimum-capacity operations, stand-down and return from temporary measures. Link wider safety/welfare/environment and relevant authority/emergency/media arrangements without inventing a physical relocation workstream. |
| 8.5–8.6 | Exercise programme and formal outcomes/actions; evaluation of BIA, risks, solutions, plans and relevant suppliers at planned intervals, after incidents/activation and significant changes. |
| 9.1–9.3 | Valid continuity-performance measures/results, organizational internal audit programme and management review decisions/actions. A separate certification audit after every migration is not prescribed. |
| 10.1–10.2 | Containment/correction, cause analysis, corrective-action effectiveness and retained evidence; continual improvement using qualitative/quantitative results. |

#### Release and closure recommendation

 retain this as a planning approach, not production continuity approval. Resolve the applicable P1 gaps before the relevant exposure and agree stand-down/retirement/evaluation controls before activation. Link sufficient existing BCMS records rather than duplicate them. No actual conformity, recovery-time achievement or supplier capability is established by this review.

#### Evidence basis

 supplied `document (5).pdf`, clauses 4–10, printed pages 7–20 (PDF pages 21–34). [ISO's official ISO 22301 overview](https://www.iso.org/standard/75106.html) confirms the BCMS purpose and identifies a later amendment; it is background only and does not replace the supplied requirements. Technical plan evidence is identified by section in each finding. No production systems or external suppliers were tested.
