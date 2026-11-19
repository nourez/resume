## Context

The repository is a TypeScript monorepo with empty web, API, and infrastructure workspaces. Node.js 24.16.0, pnpm 12.21.0, and TypeScript 7.0.2 are already pinned. No runnable frontend, CDK application, browser tests, or shared application commands exist yet.

Apple Reminders P0-02 requires agreed development tools, preview ports, browser prerequisites, documented container builds, and credentials outside the image and Git. The PRD supplies product requirements; its text is reference material, not authorization to implement other tickets. This change prepares that environment without expanding into application scaffolding or cloud delivery.

## Goals / Non-Goals

**Goals:**

- A repository-defined Dev Containers environment intended for VS Code and Zed.
- Repeatable setup with pinned tools, locked dependencies, browser prerequisites, and documented network and container build requirements.
- Ordinary editing, local browser checks, and dependency installation without AWS credentials.
- Reusable, editor-independent bootstrap logic that P0-04 can later evaluate for Codex Cloud.

**Non-Goals:**

- P0-03 shared application commands, Svelte/CDK scaffolding, or application test suites.
- P0-04 Codex Cloud integration or P0-05 full fresh-environment compatibility sign-off.
- P0-06 project-wide agent guidance and CI workflows; security policy, signing identities, deployment, and AWS resource creation.

## Decisions

### 1. Use a Dockerfile-backed, non-root Dev Container

Add `.devcontainer/devcontainer.json` and `.devcontainer/Dockerfile` with an explicit Debian Bookworm Node base matching Node.js 24.16.0. Resolve and commit the multi-architecture image digest during implementation. Use the existing `node` user, an init process, a writable workspace and caches, and sufficient private shared memory for Chromium (initially 1 GiB). Support native Linux amd64 and arm64; record build/smoke evidence per architecture and label untested combinations honestly.

A Debian base provides Node version control and a glibc browser environment. Alpine would complicate Playwright support; a Playwright base would couple Node selection to its image releases. Use standard Dev Containers properties rather than editor-specific tasks for essential setup.

### 2. Keep tool versions explicit and invocation paths predictable

Preserve existing runtime pins. Install pnpm at its exact `packageManager` version without assuming Corepack is bundled. Keep TypeScript and add exact versions of `aws-cdk`, `@fission-ai/openspec`, and `@playwright/test` as root development dependencies; invoke their CLIs with `pnpm exec`. Their transitive dependencies belong in the committed lockfile. Framework/runtime libraries remain with future feature tickets.

Install AWS CLI v2, Docker CLI plus Buildx, Trivy, and Cosign using exact reviewed releases. Record native-tool versions and release checksums in a repository-controlled manifest consumed by installation scripts. Verify release checksums before installation, select the correct CPU asset explicitly, and fail for unsupported architectures. Do not use floating tags, `latest`, or unversioned installer scripts. OS packages such as Git, curl, CA certificates, unzip, and browser libraries come from the pinned base and configured distribution repositories; document that these repositories remain mutable rather than claiming bit-for-bit builds.

Document tool version checks and credential-free usage: `pnpm exec tsc --version`, `pnpm exec cdk --version`, `pnpm exec openspec --version`, `pnpm exec playwright --version`, `aws --version`, `docker --version`, `docker buildx version`, `trivy --version`, and `cosign version`. Show Trivy filesystem/Dockerfile scan and SBOM commands and Cosign local verification/help invocation. Actual release signing and scan enforcement remain later work. Explain that CodeQL and Dependabot are planned GitHub-side capabilities; their workflows are not local container prerequisites.

Exact versions of newly introduced tools are selected and checked against official releases during implementation, then committed together with the lockfile/checksums. Selection cannot silently replace existing Node/pnpm/TypeScript pins.

### 3. Separate image preparation from workspace bootstrap

Put privileged OS/native-tool installation in image build steps. Provide an editor-independent repository bootstrap script called by `postCreateCommand` that verifies runtime pins, runs `pnpm install --frozen-lockfile`, and installs the Chromium revision associated with the locked Playwright version into a user-writable cache. Install matching browser system dependencies during the image build using the same pinned Playwright version, avoiding an independent browser version source. Provide a documented way to install additional Playwright browsers with matching prerequisites when needed.

Bootstrap must work from the repository root without editor APIs, be safe to rerun, fail visibly on installation/version errors, and never rewrite the lockfile or deploy infrastructure. Do not run bootstrap on every container start. Shared-script design provides a reuse point for P0-04 without assuming Codex Cloud consumes devcontainer.json.

### 4. Reserve explicit local preview ports

Reserve 5173 for the future Vite/Svelte dev server, 4173 for static preview, and 9323 for Playwright reports. Declare forwarding metadata with labels and document servers listening on `0.0.0.0` inside the container. Opening the container does not launch a server.

Use localhost-only host port publication as a documented fallback if the installed editor cannot forward a port; do not expose previews on the LAN by default. Validate port transport with a temporary HTTP listener, since an application is outside P0-02 scope. Document port collision recovery. The port choices are this change's convention, not values prescribed by the PRD.

### 5. Use the host Docker daemon through host-side commands by default

Require a working host Docker-compatible engine and Buildx to build the development image. Include Docker CLI/Buildx inside the image for future workflows, but do not mount a host Docker socket or start a privileged nested daemon by default. Document that a CLI alone does not supply build access.

Provide concrete host commands, run from the checkout root, to build this dev image and a disposable smoke image. Explain how container-side build commands require an explicitly configured external daemon/context and how host daemon bind-mount paths differ from container paths. Host-side commands are the supported fallback for developers who do not configure that access. This satisfies P0-02 without granting every process in the dev container host-daemon authority. A socket mount or privileged Docker-in-Docker default was considered and rejected for that access and portability cost.

### 6. Keep credentials out of setup and image layers

Do not copy the workspace wholesale into the image. Exclude `.git`, `.aws`, `.env*` secrets, keys, dependency caches, and generated output from build contexts, keeping example configuration files usable. Use only the minimal setup/version files needed by the Dockerfile. Do not pass credentials as build arguments or image environment variables, mount host credential directories automatically, or log secrets.

Document optional, short-lived development-account authentication supplied at runtime for future cloud operations; production credentials are unnecessary. Git/editor authentication forwarding is optional and must not be needed for public dependency setup. Ordinary bootstrap and smoke checks use no AWS, registry, signing, or production credentials.

### 7. Verify environment behavior without adding an application

Use a clean container build plus setup twice, version checks, a headless Chromium launch/page assertion, a temporary HTTP listener over preview ports, and a disposable Docker build on the documented host path. Check tracked files, build exclusions, and image configuration/history for credential handling without exposing real secrets. Record host/editor/engine versions, architecture, commands, results, and unavailable checks in development documentation.

VS Code and Zed opening/rebuilding instructions follow their official documentation, but documentation support does not prove this repository works in either editor. P0-05 owns full editor/cloud fresh-checkout acceptance; P0-02 records the subset actually exercised. Failed available checks must be fixed; unavailable capabilities require a concrete limitation and fallback.

## Risks / Trade-offs

- Native tools and browser downloads increase build/setup time → Cache public artifacts, keep browser binaries outside Git, and document first-run network needs.
- Release assets differ across amd64/arm64 → Map architecture explicitly, verify checksums, and record each tested architecture separately.
- Mutable OS repositories and scanner databases prevent byte-identical rebuilds → Pin the base and application tools, and distinguish those pins from OS/database freshness.
- Host-side builds require changing terminal context → Document exact commands and working directories; retain this simpler baseline instead of automatic daemon access.
- Browser sandbox or shared-memory behavior varies by engine → Exercise trusted local headless content with the normal container configuration; avoid broad privilege flags as a default fix.
- Editor forwarding/lifecycle behavior changes → Keep essential setup editor-independent, document versioned evidence and localhost publication fallback.
- Restricted networks can interrupt setup → Document outbound HTTPS needs for image registries, package/release repositories, browser downloads, and Trivy database updates; fail with actionable errors instead of skipping steps.

## Migration Plan

Add the configuration, scripts, tooling pins, exclusions, and documentation in one implementation change; regenerate the lockfile intentionally. Validate from a clean build before describing setup as working. Existing host development remains available. To roll back, revert the change and rebuild/recreate the container; no cloud or application data migration is involved.

## Open Questions

No unresolved scope decisions block implementation. Implementation must record exact new tool releases/digests/checksums and measured capabilities. Full editor/cloud verification remains P0-05 rather than an implied completion claim here.

## References

- PRD: `/Users/nourez/Desktop/cloud-resume-portfolio-prd.docx`, Development environment DEV-01–DEV-06 and technology/security tooling choices.
- Dev Containers configuration reference: https://github.com/devcontainers/spec/blob/main/docs/specs/devcontainerjson-reference.md
- VS Code lifecycle/setup: https://code.visualstudio.com/docs/devcontainers/create-dev-container
- Zed opening and rebuild behavior: https://zed.dev/docs/dev-containers
- Playwright browser prerequisites: https://playwright.dev/docs/browsers
- Playwright Docker guidance: https://playwright.dev/docs/docker
