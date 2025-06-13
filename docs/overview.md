# 📦 Buildpack Automation Bot — Product Technical Description

## 1. Overview

The **Buildpack Automation Bot** is a GitHub App designed to streamline build configuration across an organization by automatically detecting codebase types and injecting appropriate [CNCF Buildpacks](https://buildpacks.io/) into new and existing repositories. It eliminates boilerplate setup for CI/CD and containerization, encourages standardization, and accelerates developer onboarding.

### Primary Capabilities:

* Detects new repositories within a GitHub organization.
* Scans for application subfolders containing buildable projects.
* Determines technology stack (Node.js, Python, Go, Java, etc.).
* Injects GitHub Actions workflows using CNCF Buildpacks.
* Generates Pull Requests with configuration files and build instructions.
* Supports on-demand migration of legacy repos to Buildpacks via subscription.

## 2. Goals

* Standardize CI/CD onboarding across multiple repositories.
* Accelerate containerization for polyglot codebases.
* Minimize manual setup and reduce misconfigurations.
* Enable consistent use of secure, production-ready builders.

## 3. Bot Lifecycle

### a. Trigger Scenarios

1. **Repository Created** → webhook from GitHub triggers the bot.
2. **Manual Subscription** → triggered via a CLI, comment command (e.g. `/setup-buildpacks`), or GitHub workflow dispatch.
3. **Folder Discovery** → bot rescans repos periodically or on commit to detect new application folders.

### b. Detection Pipeline

* Clone or read contents via GitHub API.
* Walk directory tree.
* For each folder, run heuristics:

  * `package.json` → Node.js
  * `pyproject.toml`, `requirements.txt` → Python
  * `go.mod` → Go
  * `pom.xml` → Java/Maven
  * Multiple signals → treat as polyglot (e.g., React + Express).

### c. Configuration Injection

* Injects or updates `.github/workflows/<folder>-build.yml`
* Uses official Buildpack GitHub Actions (`setup-pack`, `pack build` or `docker/build-push-action` if CNB-based builder)
* Adds project-specific `project.toml` if necessary
* Creates a Pull Request titled `Setup Buildpack Workflow for <app-folder>`

### d. Build Output

* GitHub Action triggers on push to `main`
* Builds app using CNB
* Pushes to container registry (configurable)
* Optional Slack notification on build success/failure

## 4. Stack Architecture

### Components:

| Component            | Description                                         |
| -------------------- | --------------------------------------------------- |
| GitHub App           | Receives events and interacts with GitHub API       |
| Detection Engine     | Determines tech stack based on directory heuristics |
| Buildpack Workflow   | YAML template used to wire CI                       |
| PR Engine            | Uses Octokit to push changes and open PRs           |
| Web Dashboard (opt.) | Optional UI for usage statistics or control         |

### Dependencies:

* GitHub Webhooks (repository.created, push, workflow\_dispatch)
* Octokit (GitHub API SDK)
* CNCF Buildpacks (Pack CLI or GitHub Action)
* GitHub Actions Runner
* Docker registry (e.g., GitHub Container Registry, GCR, Docker Hub)

## 5. Technical Analysis

### Tech Stack

* **Language**: TypeScript (Node.js with Probot SDK)
* **Build Detection**: Heuristic-based + optional CNB detection
* **Build Orchestration**: GitHub Actions using Buildpacks
* **PR Management**: Octokit client

### Security & Permissions

* Requires read access to org repositories
* Requires write access to commit workflow files and open PRs
* Optional: Read/write to GitHub Container Registry for build testing

### Limitations & Constraints

* Does not currently support monorepos with complex dependencies between apps
* Assumes one build per application folder
* Detection relies on common project files (subject to edge cases)

## 6. High-Level Epics

### EPIC 1: GitHub App Installation & Bootstrap

* Create a GitHub App with webhook listeners
* Configure permissions for repo read/write
* Setup Probot or equivalent framework with event dispatcher

### EPIC 2: Language/Folder Detection Engine

* Implement recursive folder scanner
* Identify language via file heuristics
* Mark folders as buildable units

### EPIC 3: Workflow Generator

* Create reusable YAML templates
* Inject `.github/workflows/<app>-build.yml` per app
* Include Buildpack invocation and container publishing

### EPIC 4: Pull Request Automation

* Generate a branch with changes
* Open PR against default branch
* Add context messages (e.g., CI badge or usage instructions)

### EPIC 5: Manual Subscription Support

* Allow repos to opt-in via command (`/setup-buildpacks`)
* Create CLI or GitHub Action trigger

### EPIC 6: Periodic Folder Discovery

* Add scheduled workflow to detect new app folders
* Create additional workflows as needed

## 7. Sample User Stories

### 🎯 As a platform engineer:

* I want to install the Buildpack Bot across my org so that all new services have standardized builds.
* I want to receive a PR for each new buildable subfolder discovered.

### 🧑‍💻 As a developer:

* I want to subscribe my legacy repo to Buildpack Bot so that I don’t need to handwrite GitHub workflows.
* I want to receive a PR with the correct Buildpack config for my Node.js app.

### 🔁 As a devops engineer:

* I want the bot to scan repos every week to find newly added applications.
* I want to approve build config PRs before they go live.

## 8. Future Extensions

* Add support for Dockerfile-based fallback if Buildpacks fail
* Provide metrics dashboard for build success/failure per repo
* Implement auto-approval for build PRs based on org policy
* Extend support for monorepos with build ordering and dependencies

## 9. Success Metrics

* % of repos with standardized Buildpack builds
* PR merge success rate
* Build failure rate across org
* Time-to-onboarding for new services (pre vs post bot)

---

> Next: We'll generate the full product document for Bot Concept 2: Terraform Drift Detection & Notification Bot.
