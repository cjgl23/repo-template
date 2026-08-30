# Promote Global Workflow SHA Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Populate `cjgl23/repo-template` with secure CI/Security starter callers and promote DevForge and JobPilot from the validated feature SHA to the merged, post-merge-validated central `main` SHA.

**Architecture:** All consumers reference one immutable central revision: `b25f7e44fd55ce5efd4c57175557e00789d64070`. The template supplies CI and Security only. Existing DevForge and JobPilot behavior is unchanged except for replacing the central workflow SHA; changes are delivered by isolated PRs and merged only after GitHub-hosted checks pass.

**Tech Stack:** GitHub Actions reusable workflows, GitHub pull requests, YAML.

**Spec:** `docs/superpowers/specs/2026-08-30-repo-template-design.md`

## Global Constraints

- Central workflow SHA is exactly `b25f7e44fd55ce5efd4c57175557e00789d64070`.
- No floating `@main` or movable tags.
- No `secrets: inherit`.
- CI and Security callers grant only `contents: read`.
- DevForge Release retains only `contents: write` and its existing trigger/inputs.
- The template includes no release workflow.
- Do not change application code, tests, dependency files, build commands, or runtime configuration in DevForge or JobPilot.
- Remove design/plan files from the final template tree after implementation so generated repos stay minimal.

---

### Task 1: Populate the repository template

**Files:**
- Create: `README.md`
- Create: `.github/workflows/ci.yml`
- Create: `.github/workflows/security.yml`
- Delete before final merge: `docs/superpowers/specs/2026-08-30-repo-template-design.md`
- Delete before final merge: `docs/superpowers/plans/2026-08-30-promote-global-workflow-sha.md`

**Interfaces:**
- Consumes: `cjgl23/github-workflows/.github/workflows/ci.yml@b25f7e44fd55ce5efd4c57175557e00789d64070`
- Consumes: `cjgl23/github-workflows/.github/workflows/security.yml@b25f7e44fd55ce5efd4c57175557e00789d64070`
- Produces: a minimal template tree that can be copied into new repos.

- [ ] **Step 1: Create the CI caller**

```yaml
name: CI

on:
  pull_request:
  push:
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    uses: cjgl23/github-workflows/.github/workflows/ci.yml@b25f7e44fd55ce5efd4c57175557e00789d64070
    with:
      project-type: auto
```

- [ ] **Step 2: Create the Security caller**

```yaml
name: Security

on:
  pull_request:
  push:
  schedule:
    - cron: '17 3 * * 1'
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: security-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  security:
    uses: cjgl23/github-workflows/.github/workflows/security.yml@b25f7e44fd55ce5efd4c57175557e00789d64070
```

- [ ] **Step 3: Create a template README**

README must state that generated repos receive CI and Security, that workflow refs are immutable SHA pins, that Release is intentionally omitted, and that repository-specific commands can be added by editing the copied callers.

- [ ] **Step 4: Verify policy statically**

Confirm both YAML files contain the exact SHA, neither contains `secrets: inherit`, neither grants write permissions, and `.github/workflows/release.yml` does not exist.

- [ ] **Step 5: Remove implementation-only docs from the final tree**

Delete the design and plan files after they have served as the implementation contract. Their commits remain in repository history.

- [ ] **Step 6: Open PR and run GitHub-hosted validation**

Expected: template CI succeeds in conservative generic mode and template Security succeeds with no HIGH/CRITICAL findings.

---

### Task 2: Promote DevForge callers

**Files:**
- Modify: `.github/workflows/ci.yml`
- Modify: `.github/workflows/security.yml`
- Modify: `.github/workflows/release.yml`

**Interfaces:**
- Preserve all existing triggers, permissions, custom commands, inputs, and release conditions.
- Change only central workflow refs from `c90cea7916f21bf35021b626e2a98ebf1c3e8a67` to `b25f7e44fd55ce5efd4c57175557e00789d64070`.

- [ ] **Step 1: Verify current refs**

Fetch all three workflow files and confirm they currently reference `c90cea7916f21bf35021b626e2a98ebf1c3e8a67`.

- [ ] **Step 2: Create an isolated branch**

Branch from current `main` as `chore/promote-global-workflows`.

- [ ] **Step 3: Replace only the SHA in all three callers**

No other YAML change is permitted.

- [ ] **Step 4: Open PR and verify CI/Security**

Expected: full DevForge Windows/Chrome CI passes and Security passes.

- [ ] **Step 5: Merge and verify post-merge behavior**

Expected: `main` CI and Security pass. Release may run because CI completed on a push, but workflow-only changes must not create a product release.

---

### Task 3: Promote JobPilot callers

**Files:**
- Modify: `.github/workflows/ci.yml`
- Modify: `.github/workflows/security.yml`

**Interfaces:**
- Preserve Node 22, auto detection, lint/test/build flags, triggers, permissions, and concurrency.
- Change only central workflow refs from `c90cea7916f21bf35021b626e2a98ebf1c3e8a67` to `b25f7e44fd55ce5efd4c57175557e00789d64070`.

- [ ] **Step 1: Verify current refs**

Fetch both workflow files and confirm the old immutable SHA is present.

- [ ] **Step 2: Create an isolated branch**

Branch from current `main` as `chore/promote-global-workflows`.

- [ ] **Step 3: Replace only the SHA in both callers**

No application or dependency changes are permitted.

- [ ] **Step 4: Open PR and verify both gates**

Expected: JobPilot CI passes install, lint, tests, and production build; Security passes Trivy HIGH/CRITICAL scanning.

- [ ] **Step 5: Merge and verify post-merge main runs**

Expected: both CI and Security succeed on the merge commit.

---

### Task 4: Final cross-repository verification

**Files:** None.

**Interfaces:**
- Produces: one known-good central SHA consistently used by template, DevForge, and JobPilot.

- [ ] **Step 1: Fetch final workflow files from all three repos**

Verify every `cjgl23/github-workflows/.github/workflows/...` reference is pinned to `b25f7e44fd55ce5efd4c57175557e00789d64070`.

- [ ] **Step 2: Verify no security regressions**

Confirm no `secrets: inherit`, no floating central refs, template/CI/Security remain read-only, and DevForge Release retains only `contents: write`.

- [ ] **Step 3: Verify final GitHub-hosted runs**

Require successful post-merge checks for the template, DevForge, and JobPilot before declaring completion.
