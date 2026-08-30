# Repository Template Design

## Purpose

`cjgl23/repo-template` is a public GitHub template repository used to create new repositories with secure CI and security automation from the first commit.

It contains only reusable-workflow callers and documentation. It must never contain application secrets, cloud credentials, private keys, API tokens, or private application code.

## Scope

The initial template contains:

- `.github/workflows/ci.yml`
- `.github/workflows/security.yml`
- `README.md`

Automatic release automation is intentionally excluded from the default template. A repository should add release automation only after its versioning and release policy are explicitly chosen.

## Reusable workflow source

Both template workflows call `cjgl23/github-workflows` at the same immutable validated commit SHA:

`c90cea7916f21bf35021b626e2a98ebf1c3e8a67`

No caller may use `@main`, a movable version tag, or another floating reference.

## CI caller

The CI caller:

- triggers on `pull_request`, `push`, and `workflow_dispatch`
- does not hard-code a default branch name
- grants only `contents: read`
- calls `.github/workflows/ci.yml` in the central repository
- uses `project-type: auto`
- relies on the central workflow's safe unknown-project behavior
- passes no application secrets

Using unfiltered `push`/`pull_request` events avoids assuming that every new repository will use `main` forever. Repository-specific trigger narrowing can be added later when needed.

## Security caller

The Security caller:

- triggers on `pull_request`, `push`, `workflow_dispatch`, and a weekly schedule
- grants only `contents: read`
- calls `.github/workflows/security.yml` at the same central commit SHA
- passes no application secrets
- keeps the central HIGH/CRITICAL security threshold unchanged

The weekly schedule uses Monday at 03:17 UTC, matching the established security cadence.

## Security properties

The template must preserve these controls:

- central reusable workflows are pinned by full 40-character SHA
- no `secrets: inherit`
- no application secrets in template files
- CI and Security remain read-only
- no release/deployment/cloud permissions
- no write permissions

## Template behavior

Creating a repository from this template copies the caller workflow files into the new repository. The copied repository therefore receives CI and Security immediately.

The template is a copy mechanism, not live inheritance. Existing repositories do not automatically receive later changes to `repo-template` or new central workflow SHAs. Updating a consumer's central SHA must remain an explicit reviewed change.

## Acceptance criteria

The template is ready when:

1. `README.md`, `.github/workflows/ci.yml`, and `.github/workflows/security.yml` exist on `main`.
2. Both callers reference exactly `c90cea7916f21bf35021b626e2a98ebf1c3e8a67`.
3. Neither workflow contains `secrets: inherit`.
4. Both workflows declare only `permissions: contents: read`.
5. No release workflow is included.
6. The repository contains no secrets or application-specific configuration.
7. The resulting YAML is syntactically valid and structurally consistent with the validated central reusable-workflow interfaces.
