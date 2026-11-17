# Movies on Distribution

An SRE and DevOps portfolio project built around the SUPINFO team application SUPCONTENT/SUPMOVIES. The project will develop a production-style delivery and operations platform for a Node.js/Express and React application, with PostgreSQL as its data store.

The goal is to demonstrate how an application can be built, delivered, operated, observed, and recovered—not just how to provision infrastructure.

## Project status

**Current phase:** foundations. The repository currently contains the initial project structure, secret scanning, and the first cost-control decision record. Application and infrastructure automation are planned work.

## Planned platform

- **Application:** Node.js, Express, React, PostgreSQL
- **Containers and orchestration:** Docker, Kubernetes, Helm
- **Infrastructure and configuration:** Terraform, Ansible
- **Delivery and GitOps:** Jenkins, Argo CD
- **Cloud:** Google Cloud Platform (GCP), using a cost-controlled GKE environment
- **Observability:** Prometheus, Grafana, and service-level objectives (SLOs)

Components will be introduced as the project reaches each phase; this list describes the target, not features already implemented.

## Repository layout

| Path | Purpose |
| --- | --- |
| `app/` | SUPMOVIES application integration and container assets |
| `terraform/bootstrap/` | Terraform state and administrative bootstrap resources |
| `terraform/envs/` | Environment-specific infrastructure configuration |
| `terraform/modules/` | Reusable Terraform modules |
| `ansible/` | Host configuration and automation |
| `jenkins/` | Jenkins pipeline and local CI configuration |
| `helm/` | Helm charts for application components |
| `gitops/` | Argo CD and GitOps configuration |
| `observability/` | Metrics, dashboards, alerting, and SLO material |
| `scripts/` | Local workflows and operational helper scripts |
| `docs/adr/` | Architecture Decision Records |
| `docs/runbooks/` | Operational procedures |
| `docs/postmortems/` | Reliability incident reviews and learning |

## Cost guardrails

Cloud resources are used only for short, intentional deployment and validation sessions. Local development is the default. The GCP plan uses a separate administrative project for shared setup and remote Terraform state, and a zonal workload cluster with small or Spot-capable nodes where suitable.

Before provisioning cloud resources, the project will provide budget alerts, resource labels including an expiry/TTL, and checks for orphaned resources. Every cloud session must include explicit teardown and a post-session resource check. Budgets and labels support visibility and cleanup; they do not by themselves guarantee a spending cap or delete resources.

The rationale and trade-offs are recorded in [ADR 0001: Cost guardrails](docs/adr/0001-cost-guardrails.md).

## Development approach

Work proceeds in stages so each layer can be exercised locally before adding cloud cost or operational complexity:

1. Establish repository, secret-scanning, documentation, and cost-control foundations.
2. Set up WSL2 as the Linux workspace, then run Ansible, Jenkins, and local Kubernetes there. Vagrant is not part of the current plan.
3. Build Jenkins CI and add DevSecOps checks.
4. Bootstrap Terraform state and provision the cost-controlled GCP environment.
5. Package workloads with Helm and deploy through Argo CD.
6. Add observability, SLOs, and actionable alerting.
7. Exercise reliability scenarios and publish runbooks, a postmortem, and portfolio documentation.

## Local setup

The initial foundation does not yet include the application or local platform configuration. Setup instructions will be added with the relevant phase so commands match the implemented state.

## Security

Potential secrets must not be committed. The repository ignores common local credential and Terraform state files, and the versioned pre-commit hook scans staged changes with Gitleaks. Keep real credentials in local or managed secret stores, never in tracked files. The hook is a safeguard, not a replacement for careful secret handling.

## License

See [LICENSE](LICENSE).
