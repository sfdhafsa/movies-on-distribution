# ADR 0001: Cost guardrails for cloud resources

- **Status:** Accepted
- **Date:** 2026-11-17
- **Decision owners:** Project maintainers

## Context

This portfolio project will eventually use GCP for infrastructure and Kubernetes exercises. An earlier GCP project incurred unexpected costs, so avoiding uncontrolled spend is a hard requirement. Cloud infrastructure is useful for demonstrating Terraform, GKE, deployment, and operations, but the project does not need to keep a workload environment running between learning sessions.

The project also needs a durable remote Terraform state location. That shared administrative resource has a different lifecycle from temporary workload infrastructure and should not be mixed into the workload teardown process.

## Decision

Use a local-first workflow and treat workload cloud infrastructure as temporary:

1. Develop and validate locally wherever practical. Move to GCP only for a planned, time-limited exercise.
2. Keep Terraform state and required bootstrap resources in a dedicated GCP administrative project, separate from workload resources.
3. Use a zonal GKE cluster for the workload environment. Prefer small node configurations and Spot capacity where the workload can tolerate interruption.
4. Configure GCP budget alerts before workload provisioning. Alerts provide visibility; they are not a hard spending cap.
5. Apply consistent ownership, environment, and expiry/TTL labels to temporary resources. Labels support inventory and cleanup but do not expire or delete resources automatically.
6. Make teardown part of each cloud session. Verify the workload resources are removed and run an orphan-resource check before ending the session.
7. Add automated scale-down or scheduled cleanup later, after the manual provisioning and teardown workflow is documented and reliable.

## Consequences

### Benefits

- Most development does not incur cloud costs.
- Workload resources have a defined lifecycle and are separated from Terraform state infrastructure.
- Budgets, labels, teardown steps, and orphan checks provide multiple ways to spot forgotten resources.
- Zonal and Spot choices reduce cost while keeping the environment suitable for disposable learning exercises.

### Trade-offs and limitations

- A zonal cluster has less availability than a multi-zone production cluster; this is acceptable for a temporary portfolio environment.
- Spot nodes can be interrupted and are unsuitable for workloads that require uninterrupted service.
- Budget alerts can arrive after spend has occurred and do not prevent charges.
- Labels are metadata, not an automatic deletion mechanism. Cleanup remains an explicit operational responsibility until automation is added.
- Local-first development and deliberate cloud sessions require more planning and may take longer than leaving an environment running.

## Implementation notes

Before the first workload deployment, document the administrative and workload project identifiers, budget thresholds and recipients, required labels, supported node sizing, session start/stop procedure, and orphan-resource checks. Keep credentials out of Git and use the approved local or cloud credential mechanism.

Revisit this decision if the project gains a persistent demo requirement, if GCP pricing or project constraints change materially, or after cleanup automation is implemented and evaluated.
