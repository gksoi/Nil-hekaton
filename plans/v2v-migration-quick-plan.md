# NIL V2V migration — quick plan

Based on the supplied **SIST EN ISO/IEC 27001:2017** (security), **SIST EN ISO 22313:2020** (continuity guidance) and **SIST EN ISO 9001:2026** (quality). References and templates: [detailed plan](v2v-migration-detailed-plan.md).

- **Name owners:** service owner, change manager, engineers, application/database owner, security/risk owner, SOC and continuity lead; assign backups.
- **Map the service:** inventory VMs, data, platforms, capacity and dependencies, including identity, DNS, databases, integrations, backups and SOC feeds.
- **Agree business limits:** assess impact over time; approve maximum tolerable disruption, minimum service capacity, recovery time, acceptable data loss and customer requirements.
- **Assess risks:** prioritize outages, corruption, duplicate writers, unsupported conversion, exposure, capacity failure and monitoring loss. Assign treatments and accept residual risks.
- **Choose the method:** supported live migration, replication/cutover, application HA or backup/restore. Prove continuous service if required; VM copying alone does not prove it.
- **Prepare the target:** verify capacity, compatibility, network rules, access, certificates, hardening, logging and supplier support.
- **Protect recovery:** create application-consistent backups and test restoration. Keep recovery copies protected; snapshots alone are insufficient as independent backups.
- **Rehearse:** test transactions, integrity, performance, security, SOC alerts, failover and rollback after new target writes. Correct failures and retest.
- **Approve readiness:** proceed only with passing evidence, achievable service targets, accepted risks, available staff and signed go/no-go authorization.
- **Cut over in waves:** pilot first; drain/pause where required, complete final sync, fence source, activate target in dependency order, switch traffic and validate.
- **Watch the rollback deadline:** stop on corruption, critical failures, lost security/monitoring or exceeded limits. Reserve time for reconciliation, recovery and validation.
- **Roll back safely:** stop target writes; reconcile any new transactions before reopening source writes. Escalate to the continuity plan if safe rollback is unavailable.
- **Accept and stabilize:** record critical tests, actual outage/data loss and service-owner release approval; monitor through agreed service cycles and confirm backup coverage.
- **Close with evidence:** update inventory, risks, applicability decisions and recovery plans; remove temporary access; retire source only after approval; review lessons and corrective actions.

**Decision: GO WITH CONDITIONS** — a planning recommendation. Production cutover requires the readiness and approval gates in the detailed plan.
