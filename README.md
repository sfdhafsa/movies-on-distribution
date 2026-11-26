# Movies on Distribution

An SRE and DevOps portfolio project built around the SUPINFO team application SUPCONTENT/SUPMOVIES. The application source for this project is maintained in the controlled [supmovies-sre repository](https://github.com/sfdhafsa/supmovies-sre). The project will develop a production-style delivery and operations platform for a Node.js/Express and React application, with PostgreSQL as its data store.

The goal is to demonstrate how an application can be built, delivered, operated, observed, and recovered—not just how to provision infrastructure.

## Project status

**Current phase:** foundations. The controlled application repository has been selected. This SRE repository currently contains the initial project structure, secret scanning, and the first cost-control decision record; application delivery and infrastructure automation are still planned work.

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
| `app/` | Integration notes and helper files; application source stays in the controlled [supmovies-sre repository](https://github.com/sfdhafsa/supmovies-sre) |
| `terraform/bootstrap/` | Terraform state and administrative bootstrap resources |
| `terraform/envs/` | Environment-specific infrastructure configuration |
| `terraform/modules/` | Reusable Terraform modules |
| `ansible/` | Host configuration and automation |
| `jenkins/` | Jenkins controller and local CI configuration; CI will build the app from the controlled repository |
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
3. Configure Jenkins to build from the controlled [supmovies-sre repository](https://github.com/sfdhafsa/supmovies-sre), then add CI and DevSecOps checks.
4. Package the app with Helm and use Argo CD to deploy it to local Kubernetes from GitOps configuration.
5. After the local delivery workflow works, prepare GCP budgets and remote Terraform state, then use Terraform for the zonal workload environment. Deploy the same versioned app image and Helm configuration there.
6. Add observability, SLOs, and actionable alerting.
7. Exercise reliability scenarios and publish runbooks, a postmortem, and portfolio documentation.

## Local setup

The SUPMOVIES source is maintained in the controlled [supmovies-sre repository](https://github.com/sfdhafsa/supmovies-sre), under `SUPCONTENT/`. Local setup instructions for Jenkins, Kubernetes, and the app integration will be added with the relevant phase so commands match the implemented state.

## Security

Potential secrets must not be committed. The repository ignores common local credential and Terraform state files, and the versioned pre-commit hook scans staged changes with Gitleaks. Keep real credentials in local or managed secret stores, never in tracked files. The hook is a safeguard, not a replacement for careful secret handling.

## License

See [LICENSE](LICENSE).
