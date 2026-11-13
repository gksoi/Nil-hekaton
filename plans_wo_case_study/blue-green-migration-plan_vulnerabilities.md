# Blue-Green Deployment — vulnerabilities against ISO 9001

**Reviewed:** 13 November 2026. **Plan:** [blue-green-migration-plan.md](blue-green-migration-plan.md). **Normative baseline:** [document (2).pdf](../document%20%282%29.pdf), identified on its cover as SIST EN ISO 9001:2026. Clause references follow that supplied edition, not the 2015 edition.

## Scope and assessment method

This is a document gap assessment of the eight-stage plan, not a penetration test or certification audit. “Vulnerability” means a weakness that could prevent consistent delivery of the agreed service. An omission in this short plan does not prove a failure in NIL's existing quality management system (QMS). No configurations, customer contracts, test results or implementation records were available for inspection.

Each finding separates the plan's existing provision from the remaining gap. Technical remedies are recommendations for satisfying the cited quality requirements; ISO 9001 does not mandate blue-green deployment, particular software, a specific outage limit or the proposed forms. Security mechanisms are considered here only where they affect service conformity and customer property. ISO 27001 and continuity conformity are not separately audited.

**Priorities:** P1 = resolve or demonstrate an existing effective control before production release; P2 = complete before acceptance/closure, or earlier where it controls a release prerequisite. Priorities are review judgments, not ISO audit grades or calculated risk scores. All findings remain open pending evidence.

## Overall conclusion

The plan is a sound outline: it explicitly isolates green, tests recovery after new writes, defines one authoritative database, requires go/no-go and retains blue. Its largest weakness is that a whole-service switch depends on evidence and detailed controls that are requested but not supplied. A fast traffic change does not establish safe state transfer or full-load readiness.

## Findings and remedies

### BG-01 — Customer requirements are not traceable to release tests · P1

**Plan evidence:** sections 2 and 5 call for agreed limits, test evidence and user notices. **Gap:** there is no reference to a reviewed customer/SLA requirements baseline, requirements changes, intended-use workflows or statutory/regulatory requirements relevant to the service.

**Failure scenario:** green passes internal tests but violates a customer's latency or interruption commitment. **ISO basis:** 5.1.2, 8.2.1–8.2.4, 8.1, 8.6.

**Fix / owner:** service owner links each applicable requirement to a test, limit, reviewer and evidence; resolve conflicting commitments before approval. Include contingency communication and changes to requirements. **Close with:** approved requirements review and requirement-to-test matrix. Every applicable release requirement has a result or an authorized, permissible concession.

### BG-02 — Database handover and rollback remain conceptual · P1

**Plan evidence:** sections 3, 4 and 6 explicitly require fencing, final synchronization and reconciliation after new writes. **Gap:** no selected engine/tool, verified checkpoint method, fencing mechanism, reverse-sync compatibility or exact reconciliation procedure is specified.

**Failure scenario:** green accepts transactions, but blue cannot safely consume them within the recovery deadline. **ISO basis:** 6.1.2, 8.1, 8.5.1, 8.5.4, 8.5.6, 8.7.

**Fix / owner:** database owner supplies supported, version-specific handover/recovery instructions covering database, queues, files and jobs. Define what proves source writes are blocked and when rollback becomes infeasible. **Close with:** timed rehearsal demonstrating transaction preservation after destination writes and correct handling of interrupted replication; checkpoint and reconciliation records match the approved limits. [S2] supports the technical concern; the ISO clauses establish the quality-control obligation.

### BG-03 — Full-load readiness and equivalence lack a controlled baseline · P1

**Plan evidence:** sections 1–2 require capacity, compatibility and representative load checks. **Gap:** there is no versioned source/target configuration comparison, peak-workload definition or proof that green and the return path can handle the required load.

**Failure scenario:** a missing certificate, capacity limit or forgotten job causes failure only after all users switch. **ISO basis:** 7.1.3, 8.1, 8.5.1–8.5.2, 8.5.6.

**Fix / owner:** virtualization/application owners baseline VM images, application versions, licenses, configuration, dependencies and intended differences. Test approved peak and relevant job cycles, including source recovery capacity. **Close with:** identified configuration versions, reviewed differences and reproducible load-test results meeting customer criteria.

### BG-04 — Routing and session failures need explicit fault tests · P1

**Plan evidence:** section 3 acknowledges DNS caches, long-lived connections and old-endpoint writes; section 2 checks sessions. **Gap:** the actual entry points, routing change, drain timeout, session behavior and stale-client handling are not defined.

**Failure scenario:** some users retain blue connections while others use green, causing failed sessions or inconsistent operations. **ISO basis:** 6.1.2, 8.1, 8.5.1, 8.5.6, 8.7.

**Fix / owner:** network and application owners test fresh and cached clients, in-flight requests, persistent connections, drain expiry and failed routing updates. Specify safe retries and source protection. **Close with:** client-path and session test evidence showing writes only reach the authoritative environment and failures remain within approved service limits.

### BG-05 — Measurement reliability is assumed · P1

**Plan evidence:** section 2 records pass/fail; sections 6–7 use performance, integrity and SOC results. **Gap:** measurement sources, sampling intervals, time alignment, missing-data handling and tool verification are unspecified.

**Failure scenario:** stale dashboards or incomplete probes report success while user transactions fail. **ISO basis:** 7.1.5.1, 9.1.1; 7.1.5.2 only where measurement traceability is required or essential.

**Fix / owner:** operations defines each metric's source, units, query/version, interval, workload and reviewer. Verify probes using known successful and failed transactions; halt approval when required telemetry is absent. **Close with:** fitness-for-purpose checks and repeatable test results. Do not assume every software metric needs national-standard calibration.

### BG-06 — Change and release records need explicit content · P1

**Plan evidence:** sections 5–6 require final approval and logs; section 7 requires formal acceptance. **Gap:** the plan does not bind approval to an exact configuration/runbook version or specify the evidence of conformity and authorizing person for service release.

**Failure scenario:** the tested green build differs from the one approved, or traffic opens before required release checks finish. **ISO basis:** 7.5.2–7.5.3, 8.5.2, 8.5.6, 8.6.

**Fix / owner:** change manager records change ID, versions, approved differences, check results, time and authorizing identity. Distinguish authorization to start cutover, authorization to expose service and later stabilization acceptance. **Close with:** controlled release record tied to the executed build. Any early release follows 8.6 authority/customer approval as applicable and cannot waive mandatory obligations.

### BG-07 — External-provider controls stop at availability · P1 when providers are involved

**Plan evidence:** section 5 checks engineer/provider availability. **Gap:** no provider selection/evaluation evidence, output verification, competence requirements or re-evaluation criteria are referenced.

**Failure scenario:** a provider supports the window but supplies an unsupported replication method or misses an acceptance obligation. **ISO basis:** 8.4.1–8.4.3.

**Fix / owner:** supplier coordinator identifies relevant hosting, networking, virtualization and backup providers; agrees deliverables, support boundaries, required competencies and verification. **Close with:** provider evaluation, communicated requirements and verified results; reference existing procurement controls where sufficient. If no relevant external provision exists, record the applicability assessment rather than inventing a supplier.

### BG-08 — Named roles do not establish competence or operational knowledge · P1

**Plan evidence:** section 5 names roles; section 8 updates recovery procedures. **Gap:** competence criteria, training effectiveness, backup operators and an accessible handover package are not specified.

**Failure scenario:** the only engineer who knows reconciliation is unavailable, or operations cannot recover green after project closure. **ISO basis:** 5.3, 7.1.2, 7.1.6, 7.2–7.4, 8.5.1.

**Fix / owner:** technical leads map critical tasks to demonstrated skills and available cover; operations practices recovery from the approved instructions. Keep controlled instructions accessible if either centre fails. **Close with:** competence evidence, responsibility assignments and a successful operator handover demonstration.

### BG-09 — Failed service and customer-property incidents need controlled handling · P1

**Plan evidence:** sections 3 and 6 stop writes/roll back; section 7 resolves required issues. **Gap:** no explicit record for nonconforming output, customer-property damage notification, concession authority or verification after correction.

**Failure scenario:** technically restored service still contains missing customer records and is reopened without documented disposition. **ISO basis:** 8.5.3–8.5.4, 8.7.1–8.7.2, 10.2.

**Fix / owner:** incident/service owner records affected service/data, containment, customer impact, notifications, disposition, approving authority and retest. Notify the customer/provider if their property is lost, damaged or unsuitable. **Close with:** rehearsed defect/disposition record, correction verification and preserved evidence; rollback alone does not close the nonconformity.

### BG-10 — Closure does not yet prove retention, feedback or corrective-action effectiveness · P2

**Plan evidence:** sections 7–8 observe cycles, retain records, update documents and record lessons. **Gap:** no retention authority, deletion verification, customer-feedback method, root-cause/effectiveness review or QMS review linkage is defined.

**Failure scenario:** blue is deleted with required evidence, or a recurring cutover fault is recorded but never corrected. **ISO basis:** 7.5.3, 8.5.5, 9.1.2–9.1.3, 10.1–10.2; 9.2–9.3 at organizational level.

**Fix / owner:** service/quality owners set acceptance and retention criteria, collect feedback, approve disposal and track corrective actions through effectiveness checks. Feed significant results into the existing audit/management-review processes. **Close with:** acceptance, dependency/retirement evidence, feedback summary and closed or assigned follow-up actions. ISO 9001 does not require a separate certification audit for every migration.

## Wider clause coverage and applicability

| Clauses | Assessment of this plan | Evidence to obtain |
|---|---|---|
| 4.1–4.4 | Project scope is present; organizational context, interested parties and QMS process interfaces are not established here. | Link applicable QMS scope/process map and context review; climate relevance is an organizational assessment, not a requirement to relocate physical assets. |
| 5.1–5.3 | Named operational roles are requested; policy, leadership and accountability need organizational evidence. | Sponsor authorization, relevant quality policy and assigned authorities. |
| 6.1–6.3 | Residual risks and targets are mentioned; effectiveness measures, opportunities and change consequences need links. | Risk actions, measurable objectives, opportunity evaluation and relevant planned-change records. 6.3 applies to QMS changes; service changes also engage 8.5.6. |
| 7.1–7.5 | Infrastructure/testing are addressed conceptually; measurement, competence and document controls are incomplete. | BG-03, BG-05, BG-06 and BG-08; suitable staffing/work conditions under 7.1.4 and awareness under 7.3. |
| 8.1–8.2 | Testing and user notices are present; reviewed customer requirements need traceability. | BG-01. |
| 8.3 | Applicability is not decided. Unchanged VM relocation does not automatically make every design activity irrelevant. | Assess whether the target service or migration service is designed/developed within the QMS; if applicable, retain inputs, reviews, verification, intended-use validation, outputs and design-change approvals. |
| 8.4–8.7 | Providers, execution, release and recovery appear but need operational records. | BG-02–BG-09. |
| 9.1–9.3, 10 | Monitoring/lessons are present; feedback, analysis, QMS audits/review and corrective-action effectiveness need links. | BG-05 and BG-10, plus organizational records. |

## Recommended disposition

**Keep as a planning option; do not treat this outline as production approval.** Close BG-01 through BG-09 as applicable before exposure, and agree BG-10 closure controls before retirement. Critical evidence is the tested data-aware rollback, full-load/configuration validation and authorized release record. Existing controlled QMS records may satisfy findings without duplicating them in the plan.

## Sources and research limits

- **S1 — Normative:** supplied `document (2).pdf`, clauses 4–10, printed pages 5–24 (PDF pages 21–40). The supplied PDF controls the clause mapping; public summaries do not replace it. [ISO's quality-management overview](https://www.iso.org/standards/popular/iso-9000-family) supplies general context only.
- **S2 — Technical:** [AWS guidance on blue-green data synchronization and schema changes](https://docs.aws.amazon.com/whitepapers/latest/blue-green-deployments/best-practices-for-managing-data-synchronization-and-schema-changes.html) explains why data freshness and compatibility matter to both environments. Applied to BG-02 as engineering reasoning, not an ISO-mandated implementation.
- **S3 — Technical:** [Microsoft safe-deployment guidance](https://learn.microsoft.com/en-us/azure/well-architected/operational-excellence/safe-deployments) supports health-gated release and recovery. Platform-specific support remains unverified until NIL selects versions and tools.

No production vulnerability, actual data loss, vendor support claim or certification result is asserted by this review.


---

### Vulnerabilities according to SIST EN ISO 22301:2020 — Security and resilience — Business continuity management systems — Requirements (ISO 22301:2019)

**Additional assessment:** this section supplements the ISO 9001 review above; its earlier scope exclusions apply to that original review only. The normative source here is `document (5).pdf`, **SIST EN ISO 22301:2020**, adopting ISO 22301:2019. The full title is shown in the heading. “223091” is treated as a typographical error, not a different standard.

ISO 22301 contains BCMS requirements; ISO 22313:2020 supplies guidance and does not replace them. This comparison follows the supplied 2019-based text. Later amendments are not silently incorporated. Clause numbers below belong to ISO 22301; in particular, **10.1 covers nonconformity/corrective action and 10.2 continual improvement**.

These are gaps in a written migration plan, not confirmed operating failures or certification findings. Existing controlled BCMS evidence may close a gap. The proposed technical solutions are examples, not mechanisms prescribed by the standard. Priorities retain the definitions above; several P2 controls need definition before activation even though their execution occurs at closure. Related ISO 9001 and ISO 22301 findings may describe the same weakness from different requirements and should not be added as independent risk scores.

**What is already useful:** the plan contains preparation/testing, controlled change, recovery considerations and delayed retirement. What must additionally be demonstrated is continuation/recovery of prioritized business activities at predefined capacity and within BIA-derived time frames, including disruption during migration. A working VM alone is not sufficient evidence. RPO/data-loss criteria should be set where relevant to the chosen solution; no universal zero-data-loss target is inferred from ISO 22301.

#### BCBG-01 — Full-environment readiness is not a business-impact baseline · P1

**Plan evidence:** Sections 2 and 5 require full-load testing and agreed interruption/recovery limits. **Gap:** No BIA explains which business activities need what minimum capacity by what deadline, or how contractual continuity requirements shape the whole-service switch.

**Disruption scenario:** Green is technically healthy but cannot provide the minimum SOC or customer-service capacity required during disruption. **ISO 22301 basis:** 4.2; 8.2.1–8.2.2; 6.2.

**Fix / owner:** Service/continuity owners reference a reviewed BIA covering impact over time, unacceptable-impact thresholds, activity priorities and dependencies. Derive recovery targets and acceptance tests from it.

**Close with:** Approved business capacity/time requirements linked to rehearsal evidence, including applicable legal/contractual continuity requirements.

#### BCBG-02 — Two colours do not establish independent continuity capability · P1

**Plan evidence:** Section 1 prepares a complete green environment and section 8 retains blue. **Gap:** Shared failure domains and access to resources during a source/management-plane outage are not assessed.

**Disruption scenario:** Both environments depend on the same identity, storage, network or backup control plane and become unusable together. **ISO 22301 basis:** 8.2.3; 8.3.1–8.3.4.

**Fix / owner:** Continuity/architecture owners identify common dependencies and evaluate alternate solutions against impact, risk tolerance, cost and benefit. Test the selected recovery resources under the failure they are intended to address.

**Close with:** Dependency/failure assessment and tested continuity solution; two running environments alone are not closure evidence.

#### BCBG-03 — A stale blue environment is not an immediately usable recovery solution · P1

**Plan evidence:** Sections 3–4 explicitly reconcile new green writes before reopening blue. **Gap:** No selected supported reverse-sync process or alternate recovery path proves time/capacity targets can be met after write ownership transfers.

**Disruption scenario:** Green fails after a large volume of transactions; reconciliation takes longer than the tolerated disruption. **ISO 22301 basis:** 8.3.2–8.3.5; 8.4.4; 8.4.5.

**Fix / owner:** Database/continuity owners document timed reconciliation, irreversible schema/state boundaries and recovery when blue cannot be safely reopened. Include files, queues and external effects where required.

**Close with:** Realistic-volume recovery exercise after green writes, with measured restoration of prioritized activities and an approved alternative for failed rollback.

#### BCBG-04 — The routing switch can outlast the approved disruption budget · P1

**Plan evidence:** Sections 2–3 mention sessions, draining and DNS caches. **Gap:** Client-path behavior, traffic control-plane availability and a tested decision deadline are not instantiated against continuity objectives.

**Disruption scenario:** Stale clients remain on blue while green is promoted, or failed routing updates leave neither usable within target time. **ISO 22301 basis:** 8.1; 8.3.4–8.3.5; 8.4.1; 8.4.4.

**Fix / owner:** Network/application owners define routing/fencing checkpoints, drain limits, maximum decision times and a surviving method to restore traffic. Test cached clients, persistent sessions and control-plane failure.

**Close with:** Timed routing/fencing/failure results meeting BIA-based limits rather than only a successful load-balancer change.

#### BCBG-05 — Go/no-go authority needs disruption activation and alternates · P1

**Plan evidence:** Section 5 names a decision authority and technical/SOC roles. **Gap:** No business continuity team relationship, impact threshold, alternate decision-maker or authority to activate a non-blue recovery option is supplied.

**Disruption scenario:** The team continues trying to restore blue when a broader continuity response should already be active. **ISO 22301 basis:** 5.3; 7.2–7.3; 8.4.2.

**Fix / owner:** Continuity lead links migration command to the formal disruption response. Name teams and alternates, predefine impact-based activation/escalation and assign authority to use the selected continuity solutions.

**Close with:** Response/authority record and exercised activation with a primary person unavailable.

#### BCBG-06 — User notices are not an exercised incident communication procedure · P1

**Plan evidence:** Sections 5–6 inform users and log actions/results. **Gap:** No procedure ensures the update channel remains usable, incoming reports are handled, providers coordinate or wider stakeholder communications are invoked when needed.

**Disruption scenario:** An outage removes the usual channel and the team cannot coordinate or explain the customer impact. **ISO 22301 basis:** 7.4; 8.4.3; 8.4.4.3.

**Fix / owner:** Communications owner defines send/receive roles, triggers, recipients, message cadence, backup channels and incident log. Link applicable authority/emergency/media procedures from the wider BCMS.

**Close with:** Warning/communications exercise including primary-channel failure and provider coordination.

#### BCBG-07 — Complete green capacity does not prove all recovery resources · P1 as applicable

**Plan evidence:** Sections 1 and 5 check target capacity, staff, backups and provider availability. **Gap:** No continuity resource plan validates alternate staff, keys/access, provider recovery capability and retained source capacity over the recovery period.

**Disruption scenario:** The team can run green but cannot recover either environment when the required supplier or operator is absent. **ISO 22301 basis:** 7.1–7.2; 8.1; 8.3.4–8.3.5; 8.6(c).

**Fix / owner:** Operations/supplier leads confirm and maintain people, information, ICT, access, support and financial resources needed by each selected continuity solution; evaluate relevant provider capability.

**Close with:** Resource readiness and supplier capability evidence supported by a recovery-access/staffing drill.

#### BCBG-08 — A successful switch-back rehearsal needs a continuing exercise programme · P1

**Plan evidence:** Section 4 rehearses new green writes, failure, reconciliation and switch-back. **Gap:** No exercise programme, change-driven retest rule or formal post-exercise improvement report is referenced.

**Disruption scenario:** The rehearsed build is replaced by a new database/routing version without revalidating recovery. **ISO 22301 basis:** 8.5; 8.6; 7.5.

**Fix / owner:** Continuity/change owners define scenarios, aims, review intervals and retest triggers; include source loss, shared dependency failure and communications over time. Record recommendations and verify resulting improvements.

**Close with:** Programme link, formal post-exercise report and evidence that the released configuration retains validated capability.

#### BCBG-09 — Accepting green does not define standing down temporary recovery · P2; define before activation

**Plan evidence:** Sections 7–8 observe green and retire blue after acceptance/recovery periods. **Gap:** No explicit process restores routine support/capacity after temporary measures or resolves backlog before standing down.

**Disruption scenario:** The change is closed while temporary permissions, workaround routing or unprocessed transactions remain. **ISO 22301 basis:** 8.4.4.3(h); 8.4.5; 8.6.

**Fix / owner:** Service/continuity owners define the stand-down authority, exit checks, backlog/data reconciliation, operational ownership and removal of temporary measures. Keep source retirement separately authorized.

**Close with:** Documented stand-down/return-to-normal results with routine operations and dependency checks.

#### BCBG-10 — Continuity effectiveness and improvement require their own evidence · P2; monitoring criteria before release

**Plan evidence:** Sections 7–8 monitor errors/cycles and record lessons learned. **Gap:** No explicit comparison of actual recovery against BIA targets or evaluation of strategies, suppliers and BCMS effectiveness is linked.

**Disruption scenario:** A business continuity shortfall is treated as an acceptable technical incident and is not corrected. **ISO 22301 basis:** 8.6; 9.1–9.3; 10.1–10.2.

**Fix / owner:** Continuity owner evaluates actual disruption/capacity, exercises/incidents and stakeholder feedback; updates BIA/risks/plans, escalates significant outcomes to BCMS management review/audit and verifies corrective actions.

**Close with:** Capability review with corrective-action ownership/effectiveness records; use the existing BCMS programme rather than invent a mandatory audit per cutover.


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
