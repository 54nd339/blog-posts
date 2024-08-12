---
title: CI/CD - From Commit to Production, Automated
description: Continuous integration, continuous delivery, and continuous deployment are three distinct things. The pipeline stages from source to production, the tooling at each one, and how GitOps and DevSecOps extend the model.
date: 2024-08-12
draft: false
slug: /interview/ci-cd
tags:
  - System Design
  - Interview Prep
  - CI/CD
---

How does a line of code a developer wrote this morning reach users this afternoon, safely, without anyone running deploy commands by hand? The answer is a pipeline: a fixed sequence of automated steps from version control to production, where a failure at any step stops the change from going further.

Three terms get abbreviated to CI/CD and they are not the same thing. This lays out what each one means, walks the pipeline stage by stage, names the tools that fill each stage, and covers the two directions the model has grown — GitOps and DevSecOps.

## Three terms, three commitments

- **Continuous integration (CI)** — developers merge to a shared branch often, several times a day, and each merge triggers an automated build and test run. The point is to surface conflicts and regressions within minutes of the commit that caused them, while the change is small and fresh.
- **Continuous delivery (CD)** — every change that passes CI is automatically taken to a deployable state and pushed through staging environments, so the software is *always* releasable. A human may still click "go" for the production step.
- **Continuous deployment (also CD)** — the same, minus the button: every green pipeline goes to production with no manual gate. This only works with deep trust in the test suite and the rollout mechanism.

![A CI/CD pipeline end to end — commit and push feed continuous integration (checkout, build, unit and integration tests); a green build feeds continuous delivery (package, deploy to staging, end-to-end tests, deploy to production, monitor), drawn as two linked cycles.](../images/ci-cd.png "A CI/CD pipeline from commit to production: integration builds and tests every merge; delivery packages a green build, promotes it through staging and end-to-end tests to production, then monitors. Source: Security Zines x ByteByteGo.")

## The pipeline, stage by stage

1. **Source** — a developer commits to a feature branch and opens a pull request against `main`.
2. **Trigger** — the merge (or a push to a watched branch) starts the CI server: Jenkins, GitHub Actions, GitLab CI, CircleCI.
3. **Build** — check out the commit, compile, and package one deployable artifact: a JAR, a binary, a Docker image.
4. **Test** — the gate that matters. Unit tests for isolated logic; integration tests for modules together; static analysis (SonarQube) for bugs and code smells without running anything; dependency and code security scans. A failure here stops the change.
5. **Store** — a passing artifact is versioned and pushed to a repository: a container registry, Artifactory, Nexus. Built once, deployed everywhere from here.
6. **Deploy to staging** — the same artifact is rolled to pre-production: dev, QA, UAT.
7. **Test in staging** — longer-running checks: end-to-end flows, load and performance tests, user acceptance testing.
8. **Release to production** — after tests pass and any approval is given, the artifact goes live, usually via a [low-risk rollout strategy](/citadel/interview/deployments) — blue-green, canary, or feature-flagged.
9. **Monitor** — production is watched (Prometheus, Grafana, Datadog, the ELK stack); alerts and incidents feed back into planning.

The build/test/store split maps onto [the twelve-factor](/citadel/interview/12-factor-app) build, release, run stages: build once, combine with per-environment config to make a release, run the release.

## The tooling at each stage

| Stage | Common tools |
| --- | --- |
| Version control | Git — GitHub, GitLab, Bitbucket |
| CI server | Jenkins, GitHub Actions, GitLab CI, CircleCI, Azure DevOps |
| Build | Maven, Gradle, npm, Bazel |
| Package / run | Docker, Kubernetes |
| Artifact store | Artifactory, Nexus, container registries (ECR, GCR) |
| Infra as code | Terraform, Ansible, CloudFormation |
| Test | JUnit, TestNG, Selenium, Cypress, Playwright, JMeter, Gatling |
| Deploy orchestration | Spinnaker, Argo CD |
| Monitoring | Prometheus, Grafana, ELK, Datadog, Splunk |

## Two directions the model has grown

**GitOps** makes a Git repository the single source of truth for infrastructure as well as code. Kubernetes manifests are stored declaratively; a controller like Argo CD or Flux watches the repo and continuously reconciles the live cluster to match it. A deploy becomes a merge; a rollback becomes a revert.

**DevSecOps** moves security checks into the pipeline instead of a pre-release audit ("shift left"): static analysis (SAST), running-app scans (DAST), container image scanning, secret detection, and dependency vulnerability checks, all automated as pipeline steps.

## At scale

**Netflix** builds with Gradle, packages applications as machine images, and deploys with **Spinnaker** using canary rollouts — new version to a slice of traffic, automated anomaly detection (Kayenta) comparing it against the baseline, promote or roll back on the result. **Uber** runs a monorepo built with Bazel, packages services into containers via an internal build system on Buildkite, and also deploys with Spinnaker, backed by custom monitoring and capacity-planning tools.

## The pipeline is the smallest part

The stages and tools are the visible half; the working half is the discipline that keeps them meaningful — small commits so a red build points at one change, a test suite trusted enough that people stop the line for it, and rollouts structured so a bad release is a shrug rather than an incident. A pipeline that teams route around when they are in a hurry is just a slow way to do manual deploys.
