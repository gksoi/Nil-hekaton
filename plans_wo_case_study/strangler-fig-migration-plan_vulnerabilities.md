# Strangler Fig Pattern — vulnerabilities against ISO 9001

**Reviewed:** 13 November 2026. **Plan:** [strangler-fig-migration-plan.md](strangler-fig-migration-plan.md). **Normative baseline:** [document (2).pdf](../document%20%282%29.pdf), identified as SIST EN ISO 9001:2026. Clause references follow this supplied edition.

## Scope and assessment method

This assessment reviews the eight plan stages for service-quality and management-control weaknesses. It is not a penetration test or certification audit. No source code, interface specification, QMS records, contracts or execution evidence were available. A missing detail in this outline is a potential gap to close or link to existing controls, not proof of organizational nonconformity.

ISO 9001 does not prescribe strangler fig, a façade, event-driven architecture or a specific synchronization mechanism. The standard provides quality requirements; the technical fixes below are proposed applications. ISO 27001 and business-continuity conformity are not independently assessed here.

**Priorities:** P1 = resolve or demonstrate effective control before the relevant replacement is released; P2 = complete before closure, or earlier if needed for acceptance. These are reviewer judgments, not formal audit grades. Findings remain open pending evidence.

## Overall conclusion

The plan correctly treats strangler fig as application modernization, not a generic VM-copy method. It already addresses interfaces, data ownership, external effects, coexistence and retirement dependencies. However, the engineering and quality controls needed to prove each replacement meets intended use are still mostly instructions to create evidence rather than completed evidence. If the authorized scope is unchanged V2V relocation, its suitability must be resolved before development starts.

## Findings and remedies

### SF-01 — Modernization scope may exceed the authorized migration · P1

**Plan evidence:** the introduction explicitly requires feasible and authorized application changes. **Gap:** no approved decision establishes that redesign is in scope, affordable, supportable and compatible with customer requirements.

**Failure scenario:** the team begins rebuilding functionality when the customer requested only hosting relocation, delaying or altering the service. **ISO basis:** 4.3–4.4, 8.2.2–8.2.4, 8.3.2, 8.5.6; 6.3 where QMS changes result.

**Fix / owner:** sponsor/service owner approves business scope, intended outcomes, requirements, design responsibility, resources and schedule. Confirm interceptable requests and modifiable integration points. **Close with:** signed scope/requirements review and design-applicability decision. If these conditions fail, reject this option for the current V2V task rather than imply compliance through the pattern name. [S2] supports the suitability analysis.

### SF-02 — Design inputs and business semantics lack traceability · P1

**Plan evidence:** sections 1–2 define inputs/outputs, interface behavior and approved changes. **Gap:** no complete baseline links business rules, error semantics, calculations, permissions and intended-use requirements to replacement tests.

**Failure scenario:** the new reporting function returns plausible but financially different results, or loses a legacy rule not exercised by a happy-path test. **ISO basis:** 8.2.3–8.2.4, 8.3.3–8.3.5, 8.6.

**Fix / owner:** function/product owner and designer capture essential behavior, conflicting requirements and approved changes. Link each input to an output and test, including boundary/error cases and actual user workflows. **Close with:** reviewed design-input baseline, equivalence/intentional-difference decisions and traceable validation results.

### SF-03 — Design review, verification and validation need distinct outcomes · P1

**Plan evidence:** the plan calls for review and tests, and section 5 mentions the design/change process. **Gap:** it does not identify design stages, review participants, controlled outputs or evidence distinguishing verification against specifications from validation for intended use.

**Failure scenario:** the API meets its specification but the complete business workflow is unusable. **ISO basis:** 8.3.2, 8.3.4–8.3.6, 8.6.

**Fix / owner:** design lead defines reviews, required experts/customer involvement, verification and intended-use validation, defect handling and design-change authorization. Activities may be combined where appropriate, but their purposes/results must be evident. **Close with:** approved design outputs and evidence linking review findings, specification tests, user validation and corrections to the released version.

### SF-04 — Data-domain boundaries and consistency rules are not concrete · P1

**Plan evidence:** sections 1 and 3 require one authoritative writer, synchronization and ownership transfer. **Gap:** there is no domain/schema mapping, transformation rule, freshness limit, reconciliation method or cross-function transaction design.

**Failure scenario:** two functions rely on a former database transaction that is no longer atomic, producing inconsistent customer state across old/new systems. **ISO basis:** 6.1.2, 8.3.3–8.3.5, 8.5.1, 8.5.3–8.5.4, 8.7.

**Fix / owner:** data/design owners define ownership at each stage, mappings, consistency invariants and failure recovery. Test partial failure, ordering, stale reads and reconciliation using workload-appropriate mechanisms. **Close with:** reviewed data contracts and fault-test evidence showing business invariants hold within approved limits. A single writer alone does not establish consistency across domains.

### SF-05 — External side effects need an executable compensation procedure · P1

**Plan evidence:** section 3 explicitly says restoring a database cannot undo payments/notifications. **Gap:** compensation ownership, retry rules, deduplication where needed and reconciliation with external records are not defined.

**Failure scenario:** a timed-out request is retried through legacy after rollback and performs the same business action twice. **ISO basis:** 8.3.4, 8.5.1–8.5.2, 8.7, 10.2.

**Fix / owner:** application/business owner defines traceable transaction identity, allowed retries and safe compensation/reconciliation. Use idempotency or another verified mechanism where necessary. **Close with:** timeout/retry/rollback drills proving no unintended repeated action and a documented process for effects that cannot be reversed automatically.

### SF-06 — Façade availability and routing correctness need measured proof · P1

**Plan evidence:** section 1 calls for a highly available façade; sections 2 and 4 test proxy failure. **Gap:** no availability/capacity target, route versioning, bypass controls, identity propagation behavior or actual failover result is provided.

**Failure scenario:** one routing change disables both old/new functions, or authorization changes at the proxy. **ISO basis:** 7.1.3, 8.1, 8.3.4–8.3.5, 8.5.1, 8.5.6.

**Fix / owner:** network/design owners test capacity, failover, routing changes, identity/authorization, direct-call paths and rollback behavior. Review the façade as a controlled production dependency. **Close with:** configuration-specific routing/load/failure evidence meeting agreed service requirements. [S2] identifies the façade bottleneck/single-failure risk; ISO does not prescribe a particular HA design.

### SF-07 — Per-function success can hide end-to-end failure · P1

**Plan evidence:** sections 2, 4 and 7 explicitly require workflows crossing the old/new boundary. **Gap:** the dependency coverage, function-selection criteria and workflow test set are not instantiated.

**Failure scenario:** independently accepted components fail a billing/reporting/job cycle when combined, or a long-lived legacy dependency is missed. **ISO basis:** 4.4.1, 8.1, 8.3.4, 8.5.1, 8.6, 9.1.1.

**Fix / owner:** service/design owners maintain a dependency and intended-use workflow map, including scheduled/rare operations. Test approved combinations under partial failures and relevant business cycles. **Close with:** workflow-to-function coverage and end-to-end results for the actual coexistence configuration. Determine the necessary detail from service risk; a formal inventory format is not imposed by ISO.

### SF-08 — Per-function release and rollback boundaries lack a controlled ledger · P1

**Plan evidence:** section 3 lists function records and warns rollback order matters; section 6 approves each replacement. **Gap:** no controlled version links route, code, data owner/schema, dependencies, approvals and rollback eligibility.

**Failure scenario:** an old function is re-enabled after its data objects have changed or disappeared, or the wrong route version is restored. **ISO basis:** 7.5.2–7.5.3, 8.3.6, 8.5.2, 8.5.6, 8.6–8.7.

**Fix / owner:** change manager records per-function release versions, test results, authorizer, current ownership, rollback dependency order and irreversible boundaries. **Close with:** reviewed ledger plus recovery rehearsal against the exact version combination. Distinguish production exposure approval from later stabilization acceptance. Customer/relevant-authority approval applies to exceptions where required.

### SF-09 — Coexistence resourcing, suppliers and retained knowledge need evidence · P1 as applicable

**Plan evidence:** section 5 names roles and mentions resources; section 8 updates recovery instructions. **Gap:** no demonstrated competence for redesign/data transitions, provider quality controls or sustained support for both systems is linked.

**Failure scenario:** one legacy specialist leaves while migration remains incomplete, or an adapter supplier changes an interface unexpectedly. **ISO basis:** 7.1.1–7.1.3, 7.1.6, 7.2–7.4, 8.3.2, 8.4.1–8.4.3, 8.5.5.

**Fix / owner:** sponsor/technical/supplier leads fund coexistence, assign backups, preserve knowledge and evaluate provider capabilities/deliverables. Define interface-change notification and verification. **Close with:** staffing and competence records, effective handover demonstrations, provider evaluations and a support/exit plan for both environments.

### SF-10 — Retiring legacy parts can remove rollback too early · P2; agree the gate before first replacement

**Plan evidence:** sections 7–8 retain recovery periods and check remaining dependencies. **Gap:** there is no evidence-based rule for retiring data objects, routes, jobs and synchronization separately for each function and for the whole application.

**Failure scenario:** deleting a legacy table removes another function's dependency or makes a supposedly available rollback impossible. **ISO basis:** 8.5.3–8.5.6, 8.6–8.7, 7.5.3.

**Fix / owner:** service/data owners approve retirement only after dependency observation, business-cycle checks, retention review and recovery-boundary approval. Preserve required records; verify removal without damaging shared resources. **Close with:** per-function/whole-service acceptance, dependency evidence, retention/disposal approval and updated recovery instructions. Removing a legacy object is a separately controlled change, not routine housekeeping. [S2] supports this technical risk.

### SF-11 — Quality measurements, customer feedback and corrective action need closure rules · P2; measurement controls before exposure

**Plan evidence:** sections 2 and 7 monitor results and section 8 records lessons. **Gap:** no verified measurement sources, customer-feedback analysis, nonconformity disposition or root-cause/effectiveness process is instantiated.

**Failure scenario:** semantic defects recur in successive replacements or measurement gaps hide stale data. **ISO basis:** 7.1.5.1, 8.5.3, 8.7, 9.1.1–9.1.3, 10.1–10.2.

**Fix / owner:** quality/operations owners verify probes using known failures, define data-freshness and workflow metrics, gather user feedback and track defects through correction, cause analysis, retest and effectiveness review. Assess required customer-property notifications. **Close with:** verified metric definitions, feedback/defect records and effective corrective-action evidence. Apply 7.1.5.2 measurement traceability only where required or essential; universal tool calibration is not assumed.

## Wider clause coverage and applicability

| Clauses | Assessment of this plan | Required linkage |
|---|---|---|
| 4.1–4.4 | Transitional architecture is scoped; organizational context/QMS boundary and interactions need links. | Approved scope and process map, interested-party requirements and organizational climate-relevance assessment. Physical relocation remains outside scope. |
| 5.1–5.3 | Functional roles are requested; leadership/policy evidence is organizational. | Sponsor authorization, quality policy, assigned authorities and escalation. |
| 6.1–6.3 | Architectural risks are explicit; actual evaluations, opportunities, measurable objectives and QMS change controls need evidence. | Risk/opportunity actions and effectiveness checks; planned QMS changes where applicable. 8.3.6/8.5.6 govern design/service changes. |
| 7.1–7.5 | Skills, knowledge, measurement and document controls remain proposals. | SF-08, SF-09, SF-11; adequate working conditions and awareness under 7.1.4 and 7.3. |
| 8.1–8.2 | Scope and intended function behavior need customer review and communication. | SF-01–SF-02, feedback and changed-requirement records. |
| 8.3 | Directly relevant to the proposed application replacement; detailed design controls are missing. | SF-01–SF-08, especially separate review/verification/intended-use validation purposes. |
| 8.4–8.7 | Suppliers, preservation, release and nonconforming output need executable controls. | SF-04–SF-11. |
| 9.1–9.3, 10 | Monitoring/lessons exist conceptually; analysis, improvement and organizational review need links. | SF-11 and relevant QMS internal audit/management-review records. ISO does not demand a separate audit after every function replacement. |

## Recommended disposition

**Confirm suitability and scope before choosing this method.** If unchanged V2V relocation is the requirement, this option likely adds redesign work without meeting the intended project boundary. If modernization is approved, close the P1 findings before the relevant release, and agree retirement/closure controls before starting. Require design and data-ownership evidence, not only infrastructure test results. Existing effective QMS records can close gaps without being duplicated here.

## Sources and research limits

- **S1 — Normative:** supplied `document (2).pdf`, clauses 4–10, printed pages 5–24 (PDF pages 21–40). [ISO's quality-management overview](https://www.iso.org/standards/popular/iso-9000-family) provides general context; the supplied PDF determines clause wording/edition.
- **S2 — Technical:** [Microsoft Strangler Fig guidance](https://learn.microsoft.com/en-us/azure/architecture/patterns/strangler-fig) explains interception, coexistence, façade and legacy-retirement concerns. SF-01, SF-06 and SF-10 apply these concerns to this plan; ISO mappings are reviewer analysis.
- **S3 — Technical:** [Microsoft safe-deployment guidance](https://learn.microsoft.com/en-us/azure/well-architected/operational-excellence/safe-deployments) supports controlled staged releases and health checks. The controls proposed here remain platform-neutral.

Application separability, provider support, actual service quality and certification status remain unverified.


---

### Vulnerabilities according to SIST EN ISO 22301:2020 — Security and resilience — Business continuity management systems — Requirements (ISO 22301:2019)

**Additional assessment:** this section supplements the ISO 9001 review above; its earlier scope exclusions apply to that original review only. The normative source here is `document (5).pdf`, **SIST EN ISO 22301:2020**, adopting ISO 22301:2019. The full title is shown in the heading. “223091” is treated as a typographical error, not a different standard.

ISO 22301 contains BCMS requirements; ISO 22313:2020 supplies guidance and does not replace them. This comparison follows the supplied 2019-based text. Later amendments are not silently incorporated. Clause numbers below belong to ISO 22301; in particular, **10.1 covers nonconformity/corrective action and 10.2 continual improvement**.

These are gaps in a written migration plan, not confirmed operating failures or certification findings. Existing controlled BCMS evidence may close a gap. The proposed technical solutions are examples, not mechanisms prescribed by the standard. Priorities retain the definitions above; several P2 controls need definition before activation even though their execution occurs at closure. Related ISO 9001 and ISO 22301 findings may describe the same weakness from different requirements and should not be added as independent risk scores.

**What is already useful:** the plan contains preparation/testing, controlled change, recovery considerations and delayed retirement. What must additionally be demonstrated is continuation/recovery of prioritized business activities at predefined capacity and within BIA-derived time frames, including disruption during migration. A working VM alone is not sufficient evidence. RPO/data-loss criteria should be set where relevant to the chosen solution; no universal zero-data-loss target is inferred from ISO 22301.

#### BCSF-01 — Function selection is not yet driven by a business impact analysis · P1

**Plan evidence:** Section 1 proposes a small separable function; section 5 sets per-function limits. **Gap:** No BIA connects functional boundaries to prioritized business activities, impacts over time, minimum capacity and unacceptable-impact deadlines.

**Disruption scenario:** A seemingly small replacement breaks a critical cross-function process whose recovery priority was missed. **ISO 22301 basis:** 8.2.1–8.2.2; 6.2.

**Fix / owner:** Service/continuity owners assess whole business workflows and dependencies before choosing function order. Derive minimum capacity/time targets for both individual functions and end-to-end activities.

**Close with:** Reviewed BIA linked to the replacement sequence and tested business-workflow recovery criteria.

#### BCSF-02 — Long coexistence changes the disruption risk over time · P1

**Plan evidence:** Sections 1 and 6 acknowledge cross-function dependencies and controlled replacement. **Gap:** No maintained transition-state risk assessment covers mixed versions, changing data ownership, unavailable legacy specialists or prolonged dual-system operation.

**Disruption scenario:** An earlier safe recovery plan becomes invalid after another function or data domain moves. **ISO 22301 basis:** 8.2.1; 8.2.3; 8.3.1–8.3.3; 8.6.

**Fix / owner:** Architecture/continuity owners evaluate each material coexistence state, compare continuity options against time/capacity, risk and cost/benefit, and update solutions as dependencies change.

**Close with:** Reviewed transition-state risks/selected solutions and change-driven revalidation before each relevant release.

#### BCSF-03 — The façade needs a tested continuity capability, not only HA intent · P1

**Plan evidence:** Sections 1, 2 and 4 require a highly available façade and proxy-failure tests. **Gap:** No measured failure/recovery design establishes how the required business capacity survives loss of routing or its control plane.

**Disruption scenario:** The façade fails and makes both healthy legacy and replacement functions unreachable. **ISO 22301 basis:** 8.2.3; 8.3.2–8.3.5; 8.4.4.

**Fix / owner:** Network/continuity owners implement and test the selected redundant or recovery route, including configuration access, identity propagation, failover capacity and communications.

**Close with:** Timed façade/control-plane failure exercise meeting end-to-end activity targets; two backends alone are insufficient.

#### BCSF-04 — Per-domain write ownership does not ensure whole-workflow recovery · P1

**Plan evidence:** Sections 1 and 3 define single-writer ownership, reconciliation and external-effect compensation. **Gap:** No selected implementation proves recovery of cross-domain transactions, event ordering/backlog, transformations and partially completed business actions.

**Disruption scenario:** One domain rolls back while another retains a newer transaction, leaving the business process inconsistent or duplicated. **ISO 22301 basis:** 8.3.3–8.3.5; 8.4.1; 8.4.4–8.4.5.

**Fix / owner:** Data/application/continuity owners define invariants, supported reconciliation/compensation and minimum-service alternatives for partial failure. Include external effects and data-freshness limits.

**Close with:** Fault exercises across function/data boundaries showing consistent outcomes and recovery within approved capacity/time targets.

#### BCSF-05 — Function rollback may be impossible in the current transition state · P1

**Plan evidence:** Section 3 explicitly warns that arbitrary rollback order can be unsafe and legacy data may be incompatible. **Gap:** The actual rollback dependency order, point of no return and alternative continuity path are not supplied.

**Disruption scenario:** A failed function cannot return to legacy because a previously migrated dependency/schema no longer exists. **ISO 22301 basis:** 8.3.3–8.3.5; 8.4.4; 8.4.5.

**Fix / owner:** Design/continuity owners identify the recovery path for each supported version/ownership combination and what happens when reversal is unsafe; retain necessary legacy objects through the approved gate.

**Close with:** Version-specific recovery rehearsal and an approved alternate solution where rollback is no longer possible.

#### BCSF-06 — Multiple release owners need one coordinated disruption response · P1

**Plan evidence:** Section 5 assigns function/data owners, engineers and change authority. **Gap:** No common response-team structure/alternates or impact-based activation rules link per-function incidents to whole-service continuity.

**Disruption scenario:** Separate teams perform conflicting recoveries on interdependent functions during the same incident. **ISO 22301 basis:** 5.3; 7.2–7.3; 8.4.2.

**Fix / owner:** Continuity lead defines command/coordination, decision rights, competent alternates, impact thresholds and priorities across legacy, replacement, data and SOC teams.

**Close with:** Multi-team activation exercise demonstrating coordinated decisions and recovery sequence when a primary owner is absent.

#### BCSF-07 — Communications must follow business impact across both systems · P1

**Plan evidence:** Section 5 informs affected users and agrees support channels. **Gap:** No procedure follows impact across old/new functions, survives channel failure, receives stakeholder reports and coordinates relevant suppliers.

**Disruption scenario:** Customers get contradictory messages from legacy and replacement support teams while the common workflow is down. **ISO 22301 basis:** 7.4; 8.4.3; 8.4.4.3.

**Fix / owner:** Communications owner sets one impact-aware incident/update record, send/receive roles, alternate channels and coordination rules; link applicable wider authority/emergency/media procedures.

**Close with:** Exercised communications using a cross-function incident and unavailable primary channel, with consistent stakeholder updates.

#### BCSF-08 — Coexistence and supplier resources need sustained readiness · P1 as applicable

**Plan evidence:** Sections 5 and 8 require resources, support and retained recovery options. **Gap:** No continuity resource plan proves ongoing legacy knowledge, adapter/provider capability, keys/data access and funding over a potentially long programme.

**Disruption scenario:** The legacy platform is unsupported or expertise disappears before all replacement recovery gates pass. **ISO 22301 basis:** 7.1–7.2; 8.1; 8.3.4–8.3.5; 8.6(c).

**Fix / owner:** Sponsor/technical/supplier owners maintain required people, ICT, information, support and finance, evaluate relevant supplier capability and define a viable coexistence period.

**Close with:** Resource/skills coverage, support commitments and tested retrieval/access to recovery information at relevant transition stages.

#### BCSF-09 — Exercises must validate changing end-to-end capability over time · P1

**Plan evidence:** Section 4 rehearses coexistence, new transactions and proxy/connectivity failures. **Gap:** No continuing exercise programme or formal report requires revalidation after function ownership, routing or schema changes.

**Disruption scenario:** Every component passed once, but the current mixed configuration has never demonstrated recovery of the business activity. **ISO 22301 basis:** 8.5–8.6; 9.1; 7.5.

**Fix / owner:** Continuity/design owners maintain scenarios tied to current transition states, exercise communications and supplier involvement, report outcomes/actions formally and evaluate minimum business capacity rather than component uptime alone.

**Close with:** Exercise schedule, current-state reports and evidence that improvements and significant changes were retested; no single event must cover every possible scenario.

#### BCSF-10 — Retirement and return to normal need business continuity gates · P2; agree before each retirement

**Plan evidence:** Sections 7–8 accept functions, retain recovery periods and retire legacy after dependencies end. **Gap:** No explicit stand-down/return process or BCMS evaluation establishes when temporary adapters, backlogs and fallback data can safely be removed.

**Disruption scenario:** Deleting a legacy object destroys recovery for another function or leaves unresolved work after closure. **ISO 22301 basis:** 8.4.4.3(h); 8.4.5; 8.6; 9.2–9.3; 10.1–10.2.

**Fix / owner:** Service/data/continuity owners verify business-cycle dependencies, backlog, normal capacity and operational ownership before ending temporary measures. Update BIA/risks/plans and feed significant outcomes/corrective actions into existing BCMS audits/reviews.

**Close with:** Per-stage and whole-service stand-down/retirement evidence, recovery-boundary approval and corrective-action effectiveness records.


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
