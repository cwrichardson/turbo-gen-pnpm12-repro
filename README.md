# Repro: `turbo gen` + pnpm 12 → `ERR_PNPM_IGNORED_BUILDS` (esbuild)

Minimal reproduction for vercel/turborepo.

`pnpm turbo gen` installs `@turbo/gen` via an isolated pnpm dlx-style path that does **not** use the workspace `allowBuilds`. Under pnpm 12 (strict dep builds), that fails when `@turbo/gen` pulls `esbuild` (build scripts required). Project-level `allowBuilds: { esbuild: true }` does not help.

Bootstrapped with:

```sh
npx create-turbo@canary -e with-shell-commands
```
(package manager: pnpm)

## Environment
- `turbo` / `@turbo/gen`: 2.10.14-canary.4
- `pnpm`: 12.x (**need to manually set** in `package.json` as canary still installs 11.x)

## Setup
```sh
pnpm install
pnpm approve-builds
```
(allow `esbuild`)

## Fail (bug)
```sh
pnpm turbo gen hello
```

Expected error:

```sh
ERR_PNPM_IGNORED_BUILDS
Ignored build scripts: esbuild@…
```

Note: the install uses a dlx cache under `~/Library/Caches/pnpm/dlx/…` (or the OS equivalent), not the workspace `node_modules`.

## Workaround
`pnpm --allow-build=esbuild dlx @turbo/gen hello`

## Notes
- Root `pnpm-workspace.yaml` already has `allowBuilds.esbuild: true`; workspace `pnpm install` succeeds.
- `pnpm approve-builds` reports nothing pending in the workspace — the failure is only on the isolated gen install path.
