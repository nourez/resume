## Why

The P0-01 monorepo pins Node.js, pnpm, and TypeScript but leaves developers to assemble the remaining tools on their hosts. P0-02 needs a repository-defined dev container so frontend, infrastructure, browser, and security work starts from a repeatable environment without production credentials (PRD DEV-01, DEV-02, DEV-04, DEV-05).

## What Changes

- Add a Dev Containers configuration and image definition with a non-root development user and pinned project tooling: Node.js, pnpm, TypeScript, CDK CLI, Playwright, OpenSpec, Docker CLI/Buildx, AWS CLI, Trivy, and Cosign.
- Add repeatable bootstrap logic for locked dependency installation and browser setup, with explicit invocation paths for tools and security checks.
- Define frontend preview ports, browser system dependencies, supported CPU architectures, container build access, and network prerequisites.
- Document opening/rebuilding the container in Codex Cloud and Zed, host-side container build fallback, and credential boundaries. Record only capabilities actually checked.
- Keep P0-03 application scripts/scaffolds, P0-04 Codex Cloud integration, P0-05 full editor/cloud verification, P0-06 project-wide guidance, and delivery/security pipelines outside this change.

## Capabilities

### New Capabilities

- `project-devcontainer`: Repository-defined development environment, pinned tooling and bootstrap, browser and preview support, container build workflow, and credential-safe setup.

### Modified Capabilities

None. There are no existing capability specifications.

## Impact

Adds `.devcontainer/` configuration, reusable setup scripts, a tooling version manifest as needed, and development documentation. Updates root package metadata and `pnpm-lock.yaml` for CLI/browser development dependencies, preserving Node.js 24.16.0, pnpm 12.21.0, and TypeScript 7.0.2. Adds Docker build-context exclusions where needed. No application behavior, cloud resources, deployment permissions, credentials, or Reminders status changes are part of this proposal.

Source scope: Apple Reminders P0-02 — Define the project dev container; Cloud Resume Portfolio PRD, Development environment requirements DEV-01–DEV-06 and security tooling choices. DEV-03 and full DEV-06 evidence remain tracked by P0-04/P0-05.
