## 1. Pin the development toolchain

- [ ] 1.1 Resolve reviewed exact versions for CDK CLI, OpenSpec, Playwright, AWS CLI v2, Docker CLI, Buildx, Trivy, and Cosign; resolve the Node.js 24.16.0 Bookworm multi-architecture base digest and native amd64/arm64 asset checksums from official sources.
- [ ] 1.2 Add exact root development dependencies for CDK CLI, OpenSpec, and Playwright and intentionally update `pnpm-lock.yaml`, preserving Node.js 24.16.0, pnpm 12.21.0, and TypeScript 7.0.2.
- [ ] 1.3 Commit native-tool versions/checksums in a consumed manifest and implement architecture-aware installation with checksum enforcement and clear unsupported-architecture failures.

## 2. Define the container and bootstrap

- [ ] 2.1 Add the pinned Dockerfile with pnpm, Git, required OS utilities, native tooling, and matching Playwright Chromium system dependencies; use only the minimal setup/version files as build inputs.
- [ ] 2.2 Add editor-independent bootstrap logic to verify runtime pins, install frozen dependencies, and install the locked Chromium revision into a development-user-writable cache; make repeat execution safe and failures visible.
- [ ] 2.3 Add `.devcontainer/devcontainer.json` with the non-root user, writable workspace/caches, init, private shared memory, bootstrap lifecycle hook, and labelled preview ports 5173, 4173, and 9323.
- [ ] 2.4 Add build-context secret/output exclusions and confirm the default configuration has no credential injection, automatic host credential mounts, Docker socket mount, or privileged nested daemon.

## 3. Document the developer workflow

- [ ] 3.1 Document host engine/Buildx requirements, VS Code and Zed open/rebuild steps, bootstrap reruns, tool invocation/version checks, network prerequisites, and version-update procedure; link this guidance from the root README.
- [ ] 3.2 Document preview binding, localhost-only host publication fallback, port collision recovery, Chromium setup, and installation of additional matching Playwright browsers and prerequisites.
- [ ] 3.3 Provide exact checkout-root host commands to build the dev image and a disposable smoke image; explain optional external-daemon setup, client-versus-daemon availability, and host bind-mount path constraints.
- [ ] 3.4 Document credential-free local usage, optional temporary development authentication, Trivy filesystem/Dockerfile/SBOM invocations, and Cosign invocation; identify CodeQL/Dependabot workflows, application commands, Codex Cloud integration, and full environment verification as separate work.

## 4. Validate P0-02 behavior and record evidence

- [ ] 4.1 Validate container configuration and shell syntax; build from a clean context and record the tested host engine/version and architecture. Exercise amd64/arm64 where available and label any unavailable architecture verification explicitly.
- [ ] 4.2 Run bootstrap twice without cloud credentials, confirm a non-root writable workspace/cache, check all required CLI versions, and verify that the lockfile remains unchanged.
- [ ] 4.3 Run a headless Chromium launch and page assertion as the development user using the locked Playwright package; fix any missing libraries, cache permissions, or shared-memory issues.
- [ ] 4.4 Use temporary HTTP listeners to verify reserved preview port access through the available forwarding/publication path, and build a disposable smoke image with the documented host workflow.
- [ ] 4.5 Check runtime mismatch/installation failure diagnostics and native checksum rejection; verify build exclusions with harmless secret-shaped fixtures and inspect image configuration/history without exposing real secrets.
- [ ] 4.6 Record commands, results, environment versions, limitations, and fallbacks in development documentation; leave full VS Code/Zed/Codex Cloud sign-off to P0-05 and do not claim unexecuted checks passed.
- [ ] 4.7 Run OpenSpec validation and review the implementation against every `project-devcontainer` acceptance scenario; confirm the final diff stays within P0-02 scope.
