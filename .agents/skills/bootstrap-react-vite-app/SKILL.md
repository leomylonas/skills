---
name: bootstrap-react-vite-app
description: Create or adapt a production-working React and TypeScript Vite browser single-page application, optionally consuming a confirmed shared web-config workspace package. Use when bootstrapping or integrating a React Vite SPA.
---

# Bootstrap a React + TypeScript Vite app

Create or adapt a real browser SPA, not a template-only deliverable. Work in the target repository and preserve existing projects, package-manager choices, source structure, scripts, styles, and uncommitted changes unless a compatible migration requires a change. Use local project dependencies and package-manager commands only; never install tools globally.

## 1. Inspect before choosing a location or toolchain

1. Inspect the repository root and the target directory before writing: hidden files, package manifests and lockfiles, workspace configuration, Node/package-manager pins, `apps/` and existing application directories, source files, Vite/TypeScript/ESLint/Prettier/Stylelint configuration, CI, documentation, and real scripts. Determine the workspace root from its actual configuration; do not infer it from a directory name or assume pnpm.
2. If creating a new workspace app and the established workspace convention supports it, place it at `apps/<app-name>`. If the repository is not a workspace, has another established application location, or the requested target is explicit, use that actual location instead. Never move an existing app merely to make it fit `apps/`.
3. For an existing app, identify its package boundary, framework and Vite versions, React version, module format, route/application entry points, CSS system, TypeScript project references, source aliases, environment variables, test tooling, and commands. Adapt it in place. Do not overwrite its source, generated configuration, lockfile, or scripts with a starter template.
4. Select and retain the repository's package manager and lockfile. If the repository has conflicting package-manager metadata or no safe way to identify its package manager, report the conflict and request a decision rather than generating a second lockfile.

## 2. Detect shared web configuration and obtain consent

A shared web-config package is optional. Its discovery must be accurate, and its use always requires explicit user confirmation.

1. Search actual workspace packages and their `package.json` `name` fields, package exports, configuration files, dependency declarations, and existing consumers. Treat a package as a candidate only when it demonstrably exports all required entry points:

   - `eslint/react`
   - `prettier`
   - `stylelint`
   - `typescript/vite.json`

   A shared package that additionally exports `eslint/next` or `typescript/next.json` is compatible but those Next-only exports are irrelevant to this app. Do not guess a package name, scope, library directory, or filesystem import path from a skill name.
2. When a valid candidate exists, explain the exact package name, evidence for its exports, and the files/configs it would replace or compose with. Ask the user a direct yes/no question before adding the workspace dependency or referencing it. A previous use elsewhere, an implied preference, or silence is not confirmation.
3. If the user explicitly selects standalone configuration, no candidate exists, discovery is ambiguous, or the user gives no answer, use the standalone path below. If the user approves a candidate, use the shared path below. Do not block a requested app on an unanswered optional-config choice.
4. If an existing app has local configuration that conflicts with the approved shared config, enumerate the semantic conflict (rules, parser, file scope, TypeScript references, or style syntax) and ask how to resolve it. Preserve local behavior while waiting; do not silently discard config or layer duplicate plugin registrations. An existing app's config does not become shared merely because it looks similar.

## 3. Resolve one compatible version set

1. Determine current stable releases at execution time from authoritative registry metadata and upstream documentation. Do not hard-code aging versions, prereleases, or aliases such as TypeScript 6/7 compatibility packages.
2. Use **one standard `typescript` dependency version** for the app. Choose the latest stable TypeScript version simultaneously supported by the selected stable `typescript-eslint` packages and the selected Vite/framework/plugin peer ranges. Confirm those peer ranges before writing dependencies. If the newest TypeScript release is unsupported, choose the newest compatible stable standard `typescript` release; do not add a second compiler, npm alias, `@typescript/typescript6`, `@typescript/native`, `tsc6`, or a compiler shim.
3. Select the latest stable compatible releases of Vite, React, React DOM, `@vitejs/plugin-react`, ESLint, `typescript-eslint`, Prettier, Stylelint and the needed configs/plugins. Respect the existing app's supported major versions and Node engine. Never use `--force`, `--legacy-peer-deps`, overrides, or ignored peer warnings to make an invalid matrix install.
4. Add `vite-plugin-checker` at a version compatible with the selected Vite, TypeScript, ESLint, and Node versions. Configure it to surface both TypeScript and ESLint diagnostics during `vite` development. This plugin improves development feedback; it does not replace deterministic `typecheck` and `lint` scripts.
5. Place each dependency in the appropriate manifest using the repository's existing convention. React, React DOM, and runtime browser dependencies normally belong in `dependencies`; Vite, the React Vite plugin, build, typecheck, lint, formatting, stylelint, and checker tooling normally belong in `devDependencies`. Every package imported by a config must be declared where that config executes; do not depend on hoisting or undeclared transitive tools.

## 4. Create or adapt the application

### New application

1. Create a browser React + TypeScript Vite application with real source files, an HTML entry document, an application mount element, a Vite config, TypeScript configuration, and a minimal usable rendered screen. The app must build and run, not merely contain a copied generator skeleton. Use the current Vite React template only as an initial source when it is compatible with the selected version matrix; inspect and complete its result rather than assuming it remains correct.
2. Configure `vite.config.ts` with `@vitejs/plugin-react` and `vite-plugin-checker`. Configure checker TypeScript diagnostics against the app's actual TypeScript configuration and ESLint diagnostics against its actual ESLint configuration. Do not point either at guessed paths; use paths relative to that config file only after those files exist. Preserve existing Vite plugins and settings when adapting an app.
3. Use a strict browser-oriented TypeScript configuration suitable for Vite: DOM libraries, React JSX support, bundler-compatible module resolution, isolated transpilation, and `noEmit`. Include the app source and Vite/client declaration files as needed by its real layout, excluding generated output. Keep Node-only Vite config typing and browser application typing appropriately scoped; do not give all browser source Node globals or types just to typecheck the config.
4. Keep Vite's standard `dev`, `build`, and `preview` behavior available through real scripts. Add noninteractive scripts that actually fail on violations:

   - `typecheck`: invokes the selected local TypeScript compiler against the app configuration.
   - `lint`: invokes ESLint over the app's relevant source/config files using flat configuration when selected.
   - `format:check`: invokes Prettier in check mode over the explicitly relevant files.
   - `lint:styles`: invokes Stylelint only when the app has CSS or another supported stylesheet syntax.

   Add `lint:fix`, `format`, or style fixes only as explicit destructive counterparts; never make the verification scripts rewrite files. Do not create no-op scripts, use shell globs that behave differently across platforms, or make root recursive scripts invoke themselves.
5. Include styles actually rendered by the application. If there are no styles, do not add Stylelint merely to satisfy a checklist. If there are CSS files, configure Stylelint for the syntax actually present and make its script inspect real style files. Do not configure SCSS, CSS Modules, Tailwind, or plugins without a consumer.

### Existing application

1. Retain the actual React mount, pages/routes, assets, Vite configuration, aliases, environment handling, and public API. Add missing tooling incrementally and merge scripts/configuration without deleting semantics.
2. Preserve TypeScript project references where present. Add Vite-compatible typechecking in the app's existing architecture rather than replacing referenced configs with a flat generated `tsconfig.json`. Keep build emit/declaration behavior intact for anything that is not this SPA.
3. Add `vite-plugin-checker` without registering duplicate React/ESLint plugins or changing production build behavior. Scope checker to the existing app's real configuration and source files.
4. Do not mass-format or alter unrelated application code to make a new check pass. Report pre-existing violations separately from violations introduced by the change.

## 5. Configure the selected configuration path

### Approved shared configuration

1. Add the confirmed workspace dependency using the repository's workspace dependency protocol and exact discovered package name. Ensure the app can resolve it from its own package directory.
2. Create or merge thin local bridge files rather than copying the shared configuration:

   - `eslint.config.mjs` imports/composes the package's `eslint/react` export, preserving necessary local file-scoped overrides and placing `eslint-config-prettier` compatibility last if the shared config contract requires it.
   - `prettier.config.mjs` re-exports or imports the package's `prettier` export using the package's documented module shape.
   - `stylelint.config.mjs` re-exports or extends the package's `stylelint` export only when the app has supported styles.
   - `tsconfig.json` extends the package's `typescript/vite.json` export, retaining app-specific `include`, `exclude`, files, aliases, and references locally.

3. Follow the package's actual export map and module type. Do not assume `default` versus named exports, CommonJS interoperability, or that JSON exports can be imported without the supported syntax. Do not apply its Next exports to Vite.

### Standalone configuration

1. Create local configs only for tooling actually used. Use current ESLint flat configuration with JavaScript/TypeScript recommended rules, React and React Hooks rules, TypeScript-aware rules where the selected setup supports them, browser globals scoped to browser files, and Node globals scoped only to Vite/config files. Exclude generated/build output. Register each plugin once and do not apply browser globals to config files blindly.
2. Add a small shareable Prettier configuration only when formatting is used. Prefer existing repository choices or Prettier defaults over an invented style policy. Put `eslint-config-prettier` after conflicting ESLint rules when using both tools.
3. Add a Stylelint recommended-standard configuration only for real CSS. Scope extra syntax parsers or plugins to actual file types. Do not lint generated assets or unsupported stylesheet formats.
4. Keep TypeScript settings local and app-specific. Do not use a shared package path, fictional config file, or a second compiler to imitate the shared option.

### ESLint rule baseline

For standalone configs, compose `@eslint/js` `configs.recommended`, `typescript-eslint` `configs.recommended`, `@eslint-react/eslint-plugin` `configs.recommended`, and `eslint-plugin-react-hooks` `configs.flat.recommended`; put `eslint-config-prettier` last. Keep `@typescript-eslint/no-unused-vars`, `react-hooks/rules-of-hooks`, and `react-hooks/exhaustive-deps` enabled. Disable overlapping `@eslint-react/rules-of-hooks` and `@eslint-react/exhaustive-deps` when present to avoid duplicate diagnostics. Apply Hooks rules to `.js`, `.jsx`, `.ts`, and `.tsx`, including custom Hooks. The TypeScript preset is syntax-based; add type-aware rules only with a configured project service. Do not enable Next rules for Vite.

For approved shared config, use its `eslint/react` preset rather than registering these plugins a second time; verify the effective config and add a narrowly scoped local override only if a required baseline rule is missing or project needs require it. Preserve existing local rules; document overrides. Preset membership can change with upstream versions, so pin the installed versions and inspect `eslint --print-config`.

## 6. Install, document, and verify the real app

1. Install through the detected package manager from the correct workspace/package context and update its real lockfile. Do not hand-write a lockfile. For an existing committed lockfile, use the repository's appropriate immutable/frozen install mode when safe after updating it.
2. Update existing project documentation or the app's existing contributor/run documentation with the actual prerequisites, selected package-manager command, app location, development command, production build/preview command, quality commands, configuration choice, and any environment variables the app really needs. Do not create a generic README that claims commands or paths were not implemented.
3. Run meaningful smoke checks using the app's real scripts and report exact commands and results:

   - dependency installation followed by the package manager's frozen/immutable install validation when applicable;
   - `typecheck` using the single selected standard TypeScript compiler;
   - ESLint through `lint`;
   - Prettier through `format:check` when configured;
   - Stylelint through `lint:styles` when configured;
   - Vite production `build`;
   - repeat `format:check` after the build; exclude generated output such as `dist/` or select source/config files explicitly without hiding source;
   - start `dev`, confirm Vite serves the app without checker configuration/load errors, introduce temporary TypeScript and ESLint errors, and confirm the running checker reports both and clears them after repair; remove the fixtures and stop the server afterward;
   - `preview` of the built result when the environment permits; and
   - verify the resolved ESLint configuration actually enables React and React Hooks rules for application `.tsx` files, and use a temporary invalid Hooks example to prove diagnostics are reported; remove the fixture afterward.

4. If a command cannot run because of unavailable runtime, network, credentials, or an established unrelated failure, report the exact command and blocker. Do not claim a template, generated files, or dependency installation proves the application runs. Summarize changed files, the resolved version set and compatibility evidence, selected shared/standalone path, checks exercised, and remaining blockers.

## References

Use current upstream documentation when implementing: [Vite React](https://vite.dev/guide/), [Vite TypeScript](https://vite.dev/guide/features.html#typescript), [vite-plugin-checker](https://github.com/fi3ework/vite-plugin-checker), [ESLint flat config](https://eslint.org/docs/latest/use/configure/configuration-files), [typescript-eslint](https://typescript-eslint.io/getting-started/), [TypeScript tsconfig reference](https://www.typescriptlang.org/tsconfig/), [Prettier sharing configurations](https://prettier.io/docs/sharing-configurations), and [Stylelint configuration](https://stylelint.io/user-guide/configure/).
