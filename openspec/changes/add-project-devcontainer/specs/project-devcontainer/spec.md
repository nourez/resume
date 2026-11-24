## ADDED Requirements

### Requirement: Repository-defined development container

The repository SHALL provide a Dev Containers configuration and image with a non-root development user, an init process, and a writable workspace. Zed SHALL be able to open and rebuild the container. The image SHALL support Linux amd64 and arm64 without forcing emulation.

#### Scenario: Open a clean checkout
- **WHEN** a developer opens a clean checkout in Zed using the documented Dev Containers prerequisites
- **THEN** the container starts with the checkout accessible and development terminals running as the non-root user
- **AND** dependency and browser caches are writable by that user

#### Scenario: Rebuild in Zed
- **WHEN** a developer follows the documented rebuild procedure in Zed
- **THEN** Zed opens the rebuilt container with the checkout and bootstrap available

#### Scenario: Unsupported architecture
- **WHEN** installation encounters an architecture without a supported tool asset
- **THEN** setup fails with an actionable error instead of installing a binary for another architecture

### Requirement: Git works inside the Zed container

Git SHALL be available to the non-root development user and able to discover and read the mounted checkout. Local Git operations SHALL work without remote credentials.

#### Scenario: Inspect a local change from Zed
- **WHEN** the development user makes a temporary tracked-file edit in a disposable checkout and runs `git status` and `git diff` from Zed's container terminal
- **THEN** Git recognizes the repository and reports the edit without an ownership error or credential prompt

### Requirement: Pinned development tools

The environment SHALL preserve Node.js 24.16.0, pnpm 12.21.0, and TypeScript 7.0.2 and provide CDK CLI, OpenSpec, Playwright, AWS CLI v2, Docker CLI, Buildx, Trivy, and Cosign. Tool versions and JavaScript dependencies SHALL be pinned in the repository, with JavaScript dependencies recorded in `pnpm-lock.yaml`. The base image SHALL be pinned by digest and native downloads SHALL be checksum-verified.

#### Scenario: Inspect installed tools
- **WHEN** the developer runs the documented version commands after successful setup
- **THEN** every required CLI resolves through its documented path and reports the committed version
- **AND** Node.js, pnpm, and TypeScript match the existing repository pins

#### Scenario: Unverified download
- **WHEN** a native-tool download fails its committed checksum verification
- **THEN** installation fails without installing or executing that asset

### Requirement: Repeatable workspace bootstrap

The repository SHALL provide an editor-independent bootstrap entry point invoked during container creation and reusable by a Codex Cloud install script. It SHALL validate runtime pins, run `pnpm install --frozen-lockfile`, and prepare the locked Playwright browser revision. It SHALL be safe to rerun, preserve the lockfile, propagate failures, and require no cloud deployment or credentials.

#### Scenario: Initial bootstrap and rerun
- **WHEN** bootstrap runs on a clean checkout and then runs a second time in the same container
- **THEN** both runs complete with usable dependencies and browser binaries
- **AND** neither run changes `pnpm-lock.yaml` or requires production credentials

#### Scenario: Version mismatch or installation failure
- **WHEN** a required runtime version is incorrect or a required dependency download fails
- **THEN** bootstrap exits unsuccessfully with an actionable diagnostic
- **AND** the environment is not reported as successfully prepared

### Requirement: Browser prerequisites

The environment SHALL provide the Linux libraries and Chromium executable required by the pinned Playwright version. Browser files SHALL be accessible to the development user, and container configuration SHALL provide adequate shared memory. Documentation SHALL describe how to install other matching Playwright browsers and their prerequisites without claiming that application or component tests already exist.

#### Scenario: Credential-free browser smoke check
- **WHEN** the non-root user runs a headless Chromium check against trusted local content using the locked Playwright package
- **THEN** the browser launches, evaluates a page assertion, and exits successfully without cloud access or additional manual system-package installation

### Requirement: Consistent preview access

The configuration SHALL reserve and label ports 5173 for frontend development, 4173 for static preview, and 9323 for Playwright reports. Documentation SHALL explain in-container binding to `0.0.0.0`, port conflicts, editor forwarding, and a localhost-only host publication fallback. Container startup SHALL not depend on a runnable application or launch application servers automatically.

#### Scenario: Reach a temporary preview server
- **WHEN** a temporary HTTP listener runs on a reserved port and the developer uses the documented forwarding or publication path
- **THEN** the host can reach that listener on the documented local URL
- **AND** host port publication uses the loopback interface by default

### Requirement: Documented container build access

The repository SHALL document host engine/Buildx prerequisites and exact host-side commands for building the dev image and a disposable smoke image from the checkout. It SHALL distinguish installed in-container Docker clients from daemon availability and describe external-daemon requirements for optional in-container builds. The default dev container SHALL not mount a Docker socket or require a privileged nested daemon.

#### Scenario: Build without container daemon access
- **WHEN** the dev container has no configured Docker daemon but the documented host engine is available
- **THEN** the documented host-side workflow builds the disposable smoke image successfully
- **AND** ordinary workspace bootstrap remains usable without in-container daemon access

### Requirement: Credentials excluded from setup and images

The development configuration SHALL keep credentials outside committed files and image layers, exclude secret-bearing files from build contexts, and avoid automatic host credential mounts. Ordinary dependency installation, editing, local browser checks, and image build smoke checks SHALL require no production credentials. Any optional future cloud authentication SHALL be documented as runtime-supplied temporary development access.

#### Scenario: Work with no cloud credentials
- **WHEN** the container is built and bootstrapped without AWS credentials, signing keys, or private registry credentials
- **THEN** public-tool installation and local smoke checks succeed
- **AND** setup does not attempt a cloud deployment or credential prompt

#### Scenario: Local secrets exist in the checkout
- **WHEN** ignored `.env` files, `.aws` files, or private keys exist locally
- **THEN** build-context exclusions prevent those files from being copied into the image
- **AND** container configuration does not inject their contents through image build arguments or environment settings

### Requirement: Setup documentation and honest verification evidence

The repository SHALL document Zed open/rebuild procedures, Git from the container terminal, tooling invocation, network prerequisites, browser setup, preview access, container builds, and credential handling. Evidence SHALL identify commands, results, host/engine/editor versions where applicable, and architecture. P0-05 owns full fresh-checkout verification.

#### Scenario: Follow setup guidance
- **WHEN** a developer follows the container documentation from a fresh checkout
- **THEN** prerequisites, setup entry points, tool commands, and recovery steps are explicit
- **AND** image registry, package registry, OS repository, release download, browser download, and scanner database network needs are stated
- **AND** the guide distinguishes OS package and scanner database updates from pinned tool versions

#### Scenario: A target was not exercised
- **WHEN** Zed, an architecture, or a Codex Cloud environment cannot be tested during P0-02
- **THEN** documentation labels its verification as pending or unavailable and states the limitation and fallback

### Requirement: Codex Cloud container workflow

The repository SHALL document how to check Docker daemon and download access in Codex Cloud and how to build and run the image where supported. P0-04 owns integration with the Codex Cloud environment install script.

#### Scenario: Codex Cloud supports container builds
- **WHEN** the managed environment has usable Docker access and can reach the required downloads
- **THEN** the documented commands build and run the repository image in that environment

#### Scenario: Codex Cloud container access is unavailable
- **WHEN** the managed Codex Cloud environment lacks usable Docker access or cannot reach the required downloads
- **THEN** the documentation records the limitation and points to the host build workflow
- **AND** it identifies the reusable bootstrap entry point for P0-04's managed environment setup
