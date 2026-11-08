---
name: bootstrap-pnpm-workspace
description: Bootstrap or adapt a pnpm monorepo with nvm-managed Node.js, separate apps and libraries, and support for non-Node projects. Use when setting up a pnpm workspace or converting an existing multi-language repository to use pnpm workspace tooling.
---

# Bootstrap a pnpm workspace

Set up the target repository for multiple projects, including projects that do not use Node.js. The target is the repository root unless the user specifies a different workspace root. Adapt existing repositories as well as empty ones; preserve their behavior and existing work.

Read the packaged [TypeScript and React style guide](references/styleguide.md). Apply its applicable conventions to TypeScript/React code produced by this skill; it does not impose a Node or React stack on non-Node projects.

## 1. Inspect and plan

- Inspect hidden files, root and project manifests, workspaces, lockfiles, Node version files, package manager configuration, CI, documentation, and existing build/test commands. Identify Node and non-Node projects, their native tooling, dependencies, and paths referenced by scripts, CI, containers, imports and docs.
- Preserve existing structure and conventions where compatible. For a new workspace use `apps/` for runnable or deployable applications and `libs/` for reusable libraries. The folders may contain projects in any language. Do not move projects merely to conform: if relocation is requested or necessary, update every affected reference and verify the result. Do not turn unrelated directories into packages.
- Identify existing package managers and lockfiles, Node constraints, pnpm versions, and workspace package names. Preserve untracked and existing files. If converting from npm, Yarn, or another manager, first inspect scripts and CI, generate and verify the pnpm lockfile with the selected pnpm version, and migrate the affected commands and references. Do not delete or overwrite another manager's lockfile or configuration without the user's approval; report any unresolved mixed-manager state. Do not assume different lockfiles are equivalent.
- If version requirements or existing settings conflict, or a safe conversion cannot be determined, explain the conflict and ask for a decision; complete independent safe changes. Do not invent commands, dependency versions, package names, or native toolchains.

## 2. Select compatible tools

- Respect explicit user choices and an existing `.nvmrc`; do not silently upgrade it. If there is no `.nvmrc` and the user has not specified a Node version, suggest the latest stable patch of the newest Node LTS release line, determined at execution time from the official [Node release schedule](https://nodejs.org/en/about/previous-releases). Pin the selected concrete version in root `.nvmrc`, not a moving `lts/*` alias. Check compatibility with existing CI versions, `engines`, dependencies, and pnpm before applying it. If the latest LTS conflicts with established requirements, explain the conflict and ask before changing those requirements or choosing an older version.
- Pin the selected exact pnpm version as `packageManager: "pnpm@<version>"` in root `package.json`; retain a compatible existing pin or explicit user choice. If no pnpm version is established, suggest the latest stable pnpm release (pnpm has no separate LTS line), resolved at execution time from the official [pnpm installation documentation](https://pnpm.io/installation) or registry. Check its Node compatibility and lockfile format before applying it; if it conflicts with established requirements, explain the conflict and ask before changing requirements or choosing an older version. Do not hard-code an aging version in this skill. Avoid forcing global installations or changing user shell profiles.
- Document `nvm install` and `nvm use` from the workspace root. `nvm` is a shell function and may need to be loaded in non-interactive shells; nvm-sh targets POSIX environments, macOS and WSL, not native Windows. Give native Windows users a compatible alternative without implying nvm-sh works there. Explain how to install or activate the pinned pnpm version for the chosen Node version. Corepack is optional and may need separate installation; do not assume it ships with Node or enable it globally without consent. Align existing CI version selection and install commands without replacing unrelated jobs.

## 3. Create or adapt the workspace

For a new repository, use this structure; preserve compatible existing paths in an established repository:

```text
.nvmrc
package.json
pnpm-workspace.yaml
pnpm-lock.yaml
apps/                # runnable/deployable projects in any language
libs/                # reusable libraries in any language
```

- Create a private root `package.json` with a valid repository-appropriate name and the exact pnpm pin. Merge into an existing manifest without discarding fields or scripts. A Node root is workspace tooling, not a requirement that every project use Node.
- Create or merge `pnpm-workspace.yaml` with package globs for the actual Node workspace projects. For the default one-level layout, include `apps/*` and `libs/*`; retain existing patterns, exclusions, and settings. pnpm includes matching directories with package manifests; it does not automatically run or manage projects in other languages. Do not wrap a non-Node project in a `package.json` just to make it discoverable.
- Keep empty `apps/` and `libs/` visible in Git with `.gitkeep` if needed. Respect existing ignore rules and add only appropriate workspace artifacts such as `node_modules/` to `.gitignore`; never ignore source, native lockfiles, or the root `pnpm-lock.yaml`.
- Generate `pnpm-lock.yaml` using the pinned pnpm version and the real manifests; do not hand-write a lockfile or discard existing dependencies. Use `workspace:` references when an actual Node package depends on a local workspace package, preserving compatible dependency ranges and publish behavior. Do not create example applications or libraries unless requested.
- Preserve each non-Node project's native manifest, lockfile, runtime, and build/test commands. Document how to install and run those commands independently. If unified root commands are required by the user or already exist, integrate real native commands explicitly, preserve failures and working directories, and verify they run; do not claim `pnpm -r` discovers native-only projects. Never create fake `package.json` files or no-op scripts. Do not add root recursive scripts that call themselves; root inclusion and missing-script behavior vary by pnpm configuration and version.

- Copy one complete copy of the packaged guide into the workspace root as `STYLEGUIDE.md`, never into each package and never as a link back to this skill installation. Link it from the existing README/contributor docs and instruct agents to read and follow it in root `AGENTS.md`, creating a minimal file if absent. Preserve existing instructions and avoid duplicate pointers. If an existing target guide differs, do not overwrite it: explain the conflict and ask before replacing or reconciling it. Verify the guide and references resolve.
- Inspect existing guide locations before copying. If another style guide or package-local copy already exists, do not create a duplicate or delete it; ask how to reconcile or relocate it to the root and update references.

## 4. Verify and document

- Using the pinned Node and pnpm versions where available, run `pnpm install` (use frozen lockfile mode on an existing committed lockfile where appropriate), then a clean `pnpm install --frozen-lockfile`; report network or tool prerequisites if unavailable. Run `pnpm -r list --depth -1` and check the discovered Node packages are the intended ones, with no duplicates or unintended projects. Exercise actual root and project scripts that were added or migrated; confirm failures propagate.
- Verify `.nvmrc` with `nvm install`/`nvm use` when nvm is available; otherwise report that it was not exercised. For each existing non-Node project affected by the change, run its actual safe native verification command when its tools are available; report exact blockers instead of implying pnpm verified it. Verify the result remains stable on a second application and that intended files are not ignored.
- Update the existing README or relevant contributor documentation with workspace layout, pinned Node/pnpm setup, install and verification commands actually supported, how to add Node packages versus native projects, and native tooling prerequisites. Do not overwrite existing documentation. Report created/changed files, migrations, checks run, and any remaining limitations.
