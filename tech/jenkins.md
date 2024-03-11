---
title: Jenkins - Pipelines, Agents, and the Build-Test-Deploy Loop
description: Jenkins is an extensible automation server that runs the build, test, and deploy pipeline every time code changes. Its core vocabulary - jobs, builds, the Jenkinsfile, the master-agent split - and a walk through a typical pipeline from commit to production.
date: 2024-03-11
draft: false
slug: /tech/jenkins
tags:
  - Tools
  - CI/CD
  - DevOps
---

The point of [continuous integration and delivery](/citadel/interview/ci-cd) is that merging code, running the test suite, and shipping a release stop being things a person does by hand and become things that happen automatically on every commit. Something has to actually run those steps, and for a long time the default answer has been **Jenkins** — an open-source automation server written in Java, built around a large plugin ecosystem.

This post covers the vocabulary you need to read a Jenkins setup, and walks one pipeline from a `git push` to a production deploy.

## What Jenkins is

At its core, Jenkins is a job runner with a web UI and an HTTP API. Its usefulness comes from **plugins** — thousands of them — that connect it to [Git](/citadel/tech/git), Maven, Gradle, npm, [Docker](/citadel/tech/docker), [Kubernetes](/citadel/tech/k8s), test frameworks like JUnit and Selenium, code-quality tools like SonarQube, and artifact stores like JFrog Artifactory and Nexus. Almost anything in a release process has a plugin, and where one doesn't exist you can drop to a shell script.

## The vocabulary

- **Job (or project)** — a configured unit of work: compile this, test that, package the other.
- **Build** — one execution of a job. It has a number, an outcome (success, failure, or unstable — tests passed but with warnings), and it can produce artifacts.
- **Pipeline** — the whole workflow expressed as code, in a **`Jenkinsfile`** committed to the repository alongside the app. Two syntaxes: *scripted* (Groovy, maximum flexibility) and *declarative* (a structured, opinionated form that's easier to read and the one most teams use).
- **Master and agents** — the **master** schedules jobs, hands builds to agents, tracks their status, and shows results. **Agents** are worker machines that actually run the builds. Splitting the two lets you run builds on different operating systems, isolate them from the controller, and add capacity by adding agents.
- **Workspace** — a scratch directory on an agent where a job checks out the code and runs.
- **Artifact** — an immutable file a successful build produces: a `.jar` or `.war`, a binary, a test report. Archived for later deployment or inspection.

Keeping the pipeline in a `Jenkinsfile` means the build process is versioned, reviewed, and diffed like any other code.

## A pipeline, end to end

1. **Commit.** A developer pushes to Git.
2. **Trigger.** Jenkins notices — either by *polling* the repository on a schedule, or, better, via a *webhook* the repo fires on push, which avoids the polling delay and load.
3. **Checkout and build.** An agent checks out the commit into a workspace and runs the build steps: compile, run unit and integration tests, run static analysis (SonarQube), and package the result — often a Docker image.
4. **Store the artifact.** On success, push the image to a registry, or the `.jar` to Artifactory.
5. **Further test stages.** Deploy the artifact to a QA environment for regression and performance tests, then to a UAT (user acceptance) environment.
6. **Deploy.** Once tests pass and any required approvals are in, promote to staging and then production — frequently with a [blue-green or canary strategy](/citadel/interview/deployments) rather than a straight swap.
7. **Notify.** At each stage, report status to the team over email or Slack.

## Where Jenkins sits

Jenkins is one piece of a toolchain: source control (GitHub, GitLab, Bitbucket), build tools (Maven, Gradle, Ant, npm), test frameworks (JUnit, Selenium, Cypress), quality gates (SonarQube, Checkstyle), artifact repositories (Artifactory, Nexus), containers and orchestration (Docker, Kubernetes), configuration management (Ansible, Chef, Puppet), and the cloud platform underneath. Jenkins is the thing that calls all of them in order.

## The takeaway

Jenkins is a job runner plus plugins plus a pipeline-as-code file. Newer hosted tools — GitHub Actions, GitLab CI — cover the same ground with less to operate, and many teams have moved. Jenkins stays common because of inertia, the plugin catalogue, and the fact that a `Jenkinsfile` can express build logic that hosted YAML formats make awkward. If you're choosing today, weigh "one more server to run" against "it can do anything."
