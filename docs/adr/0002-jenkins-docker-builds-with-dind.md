# ADR 0002: Build container images with Docker-in-Docker

- **Status:** Accepted
- **Date:** 2026-11-26

## Context

Jenkins is the project's CI tool and runs in Docker Compose on my Windows development machine with Docker Desktop and WSL2. This phase stays local to avoid cloud costs, as recorded in [ADR 0001](0001-cost-guardrails.md). Pipeline jobs use short-lived Docker agents with pinned image tags and run lint, tests, image builds, Trivy, Gitleaks, and pushes to a local registry.

The build stage needs a Docker CLI and a Docker daemon. I want to keep the daemon out of the Jenkins controller container and avoid giving Jenkins direct access to Docker Desktop's host socket.

## Decision

Use a separate `docker:dind` sidecar as the build daemon. Install the Docker Pipeline plugin in the Jenkins controller image and make the Docker CLI available on the Jenkins node that launches Docker agents (the controller may fill both roles in this single-node lab). Jenkins connects to DinD over TLS. Pin Jenkins, DinD, and agent image tags. Keep the DinD image cache in a separate volume that can be removed to reset it.

This is a lab-only choice. Jenkins' [Docker setup guide](https://www.jenkins.io/doc/book/installing/docker/) uses a TLS-enabled DinD service and requires privileged mode. The separate daemon avoids exposing the host Docker socket, but privileged mode is still a significant risk and is not strong isolation.

## Options considered

1. **Mount the host Docker socket (Docker-outside-of-Docker):** simplest and shares the host image cache, but socket access gives Jenkins root-equivalent control of the host Docker daemon.
2. **Docker-in-Docker:** selected; a separate, resettable daemon avoids direct access to the host socket, at the cost of privileged mode, TLS/network/volume setup, and a separate image cache.
3. **Daemonless builder (Kaniko, rootless BuildKit, or Buildah):** avoids a Docker daemon and can fit restricted Kubernetes agents. The [original Google Kaniko repository](https://github.com/GoogleContainerTools/kaniko) was archived in June 2025; forks have different maintenance and image-distribution arrangements, so Kaniko is not the default for this lab.

## Consequences (positive and negative)

- **Positive:** Jenkins and its Docker setup are reproducible from committed configuration; the host Docker socket is not mounted; deleting the DinD data volume resets its cache; lint, test, and scan stages can remain independent of the image builder.
- **Negative:** DinD needs privileged mode and additional TLS, network, and volume configuration. Its separate cache makes first builds slower. This local lab setup does not represent a typical restricted Kubernetes build environment. Moving to another builder will require changing build/push configuration and may require agent or credential changes.

## Revisit when

- Jenkins agents move to Kubernetes on kind or GKE; evaluate rootless BuildKit or Buildah for that environment.
- There is time at the end of the project for a builder migration; record it in a new ADR that supersedes this one.
