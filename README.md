# Cloud Resume

Personal website and resume, organized as a TypeScript monorepo.

## Toolchain

- Node.js **24.16.0**, pinned in `.node-version` and `package.json`.
- pnpm **12.21.0**, pinned in `package.json`; `pnpm-workspace.yaml` defines the workspaces.
- TypeScript **7.0.2**, shared at the repository root.
- `pnpm-lock.yaml` records the dependency tree and belongs in Git.

Use a Node version manager that reads `.node-version` (for example, `fnm use`),
then verify `node --version` and `pnpm --version` match the versions above.
With Corepack installed, run `corepack enable` to enable the pnpm shim; it selects
the version pinned in `packageManager`. Alternatively, install the pinned version
with `npm install --global pnpm@12.21.0`.

From the repository root, install the locked dependencies:

```sh
pnpm install --frozen-lockfile
```

## Repository layout

```text
apps/
  web/          Frontend workspace (@cloud-resume/web)
  api/          API workspace (@cloud-resume/api)
infra/          Infrastructure workspace (@cloud-resume/infra)
content/        Resume and portfolio content
openspec/       Specifications and change proposals
tsconfig.base.json
pnpm-workspace.yaml
```

The three code workspaces have private package manifests and `src/` directories.
They will share the root TypeScript defaults as their build configurations are added.
Content and specifications are plain directories, not workspace packages.

## Current scope

This is the repository scaffold for **P0-01 — Set up the monorepo and choose the
toolchain** in Apple Reminders. It establishes the package manager, pinned tools,
workspace layout, and lockfile. It does not yet contain a runnable application.

The following work is tracked separately:

- **P0-02:** Dev container configuration.
- **P0-03:** Shared development, type-check, test, build, and CDK synth commands,
  with enough implementation to validate them.
- **P0-04–P0-05:** Codex Cloud setup and fresh-environment verification.
- **P0-06–P0-07:** OpenSpec initialization, project guidance, and workflow validation.

Frontend, API, and infrastructure framework dependencies will be added with their
implementation tickets.
