---
name: bootstrap-nextjs-app
description: Create or adapt a production-ready Next.js and React TypeScript application, including optional integration with a confirmed shared web-config workspace package. Use when bootstrapping or maintaining a Next.js browser UI with its server boundary, tooling, scripts, and verification.
---

# Bootstrap a Next.js application

Create a real, runnable Next.js application, or safely adapt an existing one. The target is the user-specified application; in an existing workspace, use its established application location, or `apps/<app-name>` for a new app when `apps/` is the workspace convention. Do not create a template-only deliverable, globally install tools, replace an existing application, or disturb unrelated projects.

Use plain CSS Modules (`*.module.css`) for project-owned component and feature styles. At most one application-wide global CSS entry is allowed for resets, base elements and shared tokens; import it once at the appropriate application root. Do not introduce SCSS/Sass or other project-owned non-module stylesheets/imports. Non-module CSS required by an installed package or library is an exception; import it as documented. For an existing conflicting styling system, preserve behavior, report the conflict and ask before a migration; don't silently rewrite unrelated styles.

## Required project conventions

Before generating or changing TypeScript, TSX, CSS, tooling, or project documentation, read this skill's packaged `references/styleguide.md`. Apply it in generated source: organize by domain or responsibility with PascalCase directories and ordinary source filenames, except Next's required paths and names. Give substantive components a same-named PascalCase directory and colocate its CSS module, private hooks, private child components, and tests. A standalone `.tsx` file is only for lightweight composition such as a layout; keep private children beneath their owner and promote code only when actual feature or cross-feature reuse warrants it.

Generate non-framework React components as named function declarations with named exports. Define props as `type ComponentNameProps`, accept `props: ComponentNameProps`, and destructure on the first line of the body. `app/page.tsx`, layouts, and other Next-required files and default exports take framework precedence. Use `type` for ordinary shapes, `interface IName` only for class-oriented contracts, `import type` for type-only dependencies, strict TypeScript, and precise types or `unknown` rather than `any`; allow exceptional `any` only behind a narrow explanatory lint suppression. Use direct relative imports to defining source modules; never add `index.ts`/`index.tsx` barrels or broad re-export registries. Enforce type-only imports (with autofix where supported), explicit `any`, and runtime dependency cycles; configure cycle detection to allow type-only cycles while reporting runtime cycles.

Use kebab-case selectors in colocated `Component.module.css` files and camelCase access in TSX, while preserving the plain-CSS/no-SCSS policy above. Verify the actual Next CSS Module pipeline and any generated typings map a selector such as `.navigation-item` to a nonempty `styles.navigationItem` class in a rendered development browser and production server; a passing typecheck or build alone is not proof. If the pipeline does not support it, use a tested selector-aware transformation or another verified compatible solution before generating code that relies on camelCase access. A transformation must safely cover project-owned module paths, local versus `:global` selectors, escaped identifiers, native nested selectors, selector lists, and kebab-to-camel collisions, run before CSS Modules, keep generated typings aligned, and be verified in development and production. Otherwise preserve the selector convention, report the explicit blocker, and do not claim the mapping works. Do not conceal the problem behind an unrestricted CSS-module index signature. Keep the one global stylesheet import in the framework-prescribed root layout.

Copy the packaged guide verbatim to the target repository/workspace root as `STYLEGUIDE.md`: keep exactly one canonical copy per project/workspace, never a per-package copy. If an existing root guide differs, preserve it, report the conflict, and ask before replacement. Explicitly reference the canonical guide in the existing contributor README and root `AGENTS.md`; create only a minimal root `AGENTS.md` when it is absent, and do not duplicate guide rules in either file. Do not link generated projects to this source skill. Verify the guide file and both references as part of documentation review.

Inspect existing guide locations before copying. If another style guide or package-local copy already exists, do not create a duplicate or delete it; ask how to reconcile or relocate it to the root and update references.

## 1. Inspect before choosing a path

1. Inspect the repository root and target directory before writing: hidden files, package manifests, lockfiles, package-manager settings, Node-version files, workspace definitions, `apps/` and `libs/`, source tree, Next config, TypeScript/ESLint/Prettier/Stylelint configs, scripts, CI, deployment files, and existing documentation. Read only paths discovered by that inspection.
2. Determine whether this is:
   - an existing Next application to adapt;
   - a new application inside an existing workspace;
   - a standalone new application; or
   - an incomplete/non-Next project whose safe migration needs an explicit user decision.
   Preserve working source, routes, package names, package-manager choice, scripts, lockfiles, CI, and deployment behavior. Do not relocate an established app merely to use `apps/`.
3. Identify the installed or selected Next.js major, React/React DOM requirements, Node constraints, router (`app/` or `pages/`), CSS solution, module format, and package-manager command. Keep a compatible existing framework version unless the user requests an upgrade. A new app must use current stable Next.js, React, and React DOM releases compatible with one another and with the selected Node version, determined from authoritative npm metadata and framework documentation at execution time.
4. Detect, rather than assume, a shared web-config package. Search the discovered workspace manifests and package exports for a package providing all of these public entry points:

   ```text
   eslint/next
   prettier
   stylelint
   typescript/next.json
   ```

   Also verify that the target package manager can resolve it as a workspace dependency and that its peer/engine ranges can support the target Next, React, ESLint, and TypeScript versions. Do not infer a package name or path from a directory name.
5. If a qualifying package exists, present its exact package name, location, exports, relevant version/peer constraints, and the proposed integration changes. **Ask for explicit confirmation before adding it, extending its configs, or changing dependencies.** Treat no answer, an ambiguous answer, a request for standalone configuration, an unavailable package, or a detected incompatibility as a decision to use local equivalent configuration. Do not wait indefinitely: complete the standalone path where it is safe. If adoption would conflict with existing project configuration, explain the specific conflict and use local equivalents unless the user explicitly chooses a reconciled shared-config migration.

## 2. Select one compatible toolchain

1. Retain the repository package manager and exact lockfile semantics. For a new standalone project, choose the package manager specified by the user; otherwise use the manager evidenced by the repository. Do not create mixed lockfiles or silently convert npm, pnpm, Yarn, or Bun projects.
2. Resolve exact latest stable versions at execution time from npm metadata and official Next.js, React, TypeScript, ESLint, typescript-eslint, Prettier, and Stylelint documentation. Exclude prereleases. Check Node engines and every relevant peer range before installing. Record actual selected versions in the manifest and lockfile, never in this skill.
3. Use **one normal `typescript` dependency version** for the app and all TypeScript-facing tools. Select the latest stable compiler supported by the selected `@typescript-eslint/*`, Next.js, and framework/plugin peer ranges. Enumerate stable versions across all majors in the supported range; do not assume an older major is the newest compatible one. Do not use TypeScript 6/7 compatibility aliases, a second compiler package, npm aliases masquerading as `typescript`, or a separate compiler solely for linting. If no single supported version satisfies the chosen latest framework/tool releases, explain the concrete ranges and ask for a version choice rather than force peers, suppress warnings, or install an unsupported combination.
4. Use latest stable ESLint, `@typescript-eslint` packages, Prettier, and Stylelint only when their engine and peer requirements are compatible with the selected Next/React stack. For Next linting, prefer the framework's currently documented configuration approach. If `eslint-config-next` has incompatible peers, compose the official `@next/eslint-plugin-next` recommended/Core Web Vitals config with compatible React, Hooks, TypeScript, and Prettier flat-config layers; scope it to the app and avoid duplicate plugin registration. Do not pretend this is byte-for-byte equivalent to the bundled config.
5. Add only tooling that the app uses. Add Stylelint and a `lint:styles` script only if the application contains supported CSS to lint. Do not add CSS preprocessors, Tailwind, test frameworks, databases, or deployment adapters unless requested or already used.

## 3. Create or adapt the application

### New application

1. Use the official `create-next-app` generator through the chosen package manager only after its options and generated files are understood. Place a workspace app in the discovered workspace app location, otherwise create a standalone root application. Choose TypeScript, ESLint, and the requested router/CSS choices. Do not accept an interactive default blindly or leave the generated sample screen as the deliverable.
2. Replace starter content with a small, coherent application that starts, typechecks, lints, builds, and renders meaningful browser UI. It must include a usable route/page, semantic markup, responsive styles, and accessible labels, headings, and controls appropriate to its actual function. Use only assets and data that are available locally or deliberately included; never depend on an invented API.
3. Use the Next routing convention selected by the app. For the App Router, keep routes/layouts server components by default and add `'use client'` only at an interaction boundary. Put state, event handlers, browser APIs, and interactive controls in client components. Keep secrets, filesystem access, database access, server-only libraries, and server data access in server components, route handlers, server actions, or another documented server boundary. Never import server-only code into a client component or expose secrets with `NEXT_PUBLIC_`. For the Pages Router, apply the analogous browser/server separation through page data APIs and client components.
4. Provide an intentional metadata/title description and a real global style entry. Do not make a static export assumption: preserve or choose rendering, image, environment-variable, and deployment settings only when justified by the app's requirements.

### Existing application

1. Inspect before changing framework-generated files, route layout, config, styling, dependencies, or scripts. Extend the existing router and styling system rather than adding a second one. Keep established app/server/client boundaries and only change files required by the request.
2. Merge scripts and configurations without deleting meaningful custom rules, TypeScript paths, Next plugins, aliases, CSS handling, or deployment settings. If existing configuration contradicts the safe configuration selected here, identify the precise conflict and request a decision before replacing it.
3. Never overwrite `.env*`, production configuration, user content, lockfiles from another package manager, or uncommitted work. Provide `.env.example` only for environment variables that the application actually reads, without secrets.

## 4. Configure the confirmed shared package or local equivalents

### Confirmed shared configuration

After explicit user confirmation only:

1. Add the verified workspace package as a direct dependency using the workspace protocol supported by the existing package manager. Ensure tool binaries remain declared/resolvable where the app invokes them; do not rely on hoisting.
2. Add or merge thin local config entry points. Use the package's `eslint/next` export for the Next app, its `prettier` export for formatting, its `stylelint` export only when CSS linting applies, and extend the verified `typescript/next.json` export from the app `tsconfig.json`. Keep Next-required generated type paths, includes, and any necessary local server/tooling overrides in the consumer config. Do not use the Vite TypeScript preset in a Next app.
3. Scope browser globals and browser-only rules to browser source. Configure Node globals/types only for Next server code and Node-run config/tool files; do not declare the entire project a browser environment. Preserve local overrides required by the existing app.

### Standalone local configuration

When no confirmed compatible shared package is used:

1. Add only local config files appropriate to the installed tool versions: a current ESLint flat config, Prettier config, optional Stylelint config, and `tsconfig.json`. Reuse existing configs where present and merge instead of creating parallel competing formats.
2. Make `tsconfig.json` strict and Next-compatible. Retain or add Next's required plugin and generated type inclusion, use `noEmit` for the application, and do not set library-oriented `rootDir`, `outDir`, declaration emit, references, or fabricated aliases. Keep `next-env.d.ts` framework-generated and included as required by the current Next version.
3. Configure ESLint for TypeScript, React, Hooks, Next rules, and Prettier compatibility using versions selected in step 2. Apply Next rules to application files and Node rules/globals only to config/build/server files. Exclude genuine generated/build output without hiding source from linting.
4. For a new Prettier configuration, use exactly `{ semi: true, singleQuote: true, trailingComma: 'all', tabWidth: 2, useTabs: false, printWidth: 100, bracketSpacing: true, arrowParens: 'avoid' }`; preserve an existing conflicting configuration and ask before reconciling it. Do not add import sorting or any rule not required here. If CSS is present, use the current Stylelint recommended standard rules and syntax-specific support only for CSS syntax actually used. Do not add a stylelint script that silently checks no styles.

### ESLint rule baseline

For standalone configs, compose `@eslint/js` `configs.recommended`, `typescript-eslint` `configs.recommended`, `@eslint-react/eslint-plugin` `configs.recommended`, and `eslint-plugin-react-hooks` `configs.flat.recommended`. Register `@next/eslint-plugin-next` once and merge its `configs.recommended.rules` and `configs["core-web-vitals"].rules` for this app. Set `settings.next.rootDir` correctly when linting in a monorepo. Place `eslint-config-prettier` last. Keep `@typescript-eslint/no-unused-vars`, `react-hooks/rules-of-hooks`, `react-hooks/exhaustive-deps`, `@next/next/no-img-element`, and `@next/next/no-html-link-for-pages` enabled. Explicitly set `@typescript-eslint/consistent-type-imports` to `['error', { prefer: 'type-imports', fixStyle: 'separate-type-imports' }]` and `@typescript-eslint/no-explicit-any` to `'error'`. Register `eslint-plugin-import-x` once, configure its TypeScript parser directly and a verified node resolver for relative TypeScript modules, and enable `import-x/no-cycle: 'error'` and `import-x/no-self-import: 'error'`; `import-x/no-cycle` ignores `import type` cycles. Probe any alias resolver before relying on it, and report CommonJS/dynamic-import analysis limitations rather than treating the rule as complete proof. Disable overlapping `@eslint-react/rules-of-hooks` and `@eslint-react/exhaustive-deps` when present to avoid duplicate diagnostics. Apply Hooks rules to `.js`, `.jsx`, `.ts`, and `.tsx`; type-aware TypeScript rules require a configured project service.

An enabled Next rule may be a warning rather than an error. Preserve the upstream severity unless the project chooses otherwise, and check both the diagnostic and exit status rather than assuming every Next violation fails CI.

For approved shared config, consume its `eslint/next` preset instead of registering the same plugins twice. Verify the effective config explicitly enables `@typescript-eslint/consistent-type-imports` with the type-import/autofix options above, `@typescript-eslint/no-explicit-any: 'error'`, plus `import-x/no-cycle: 'error'` and `import-x/no-self-import: 'error'` with the parser/resolver configuration above; add narrowly scoped local overrides only when one is missing or project needs require it. Its Prettier configuration must use `{ semi: true, singleQuote: true, trailingComma: 'all', tabWidth: 2, useTabs: false, printWidth: 100, bracketSpacing: true, arrowParens: 'avoid' }`; preserve a conflicting existing configuration and ask before reconciliation. Preserve existing rules; document overrides and rule coverage differences from `eslint-config-next`. Pin dependency versions and inspect `eslint --print-config` because upstream presets can change.

Next's `"use client"` cannot be reliably inferred from generic flat-config globs: don't assume all `app/` files execute in either environment. Use scoped rules when project filename or directory conventions actually identify server/client files; verify those scopes and report ambiguous cases. `no-undef` is disabled for TypeScript by the recommended preset, so it alone cannot enforce browser/server globals. Don't blanket-ban legitimate `process.env.NEXT_PUBLIC_*` access in client code.

## 5. Scripts, installation, and documentation

1. Preserve existing scripts and add or repair real scripts with failure-propagating commands appropriate to the selected package manager:
   - `dev` starts Next development mode;
   - `build` runs the actual Next production build;
   - `start` runs the built application;
   - `lint` runs the configured ESLint scope;
   - `typecheck` runs the single selected TypeScript compiler with `--noEmit`;
   - `format:check` checks formatting without writing;
   - `lint:styles` only when supported styles and Stylelint configuration exist.
   Do not make scripts no-ops, recursive workspace invocations, or scripts that depend on globally installed executables.
2. Install through the repository package manager and update its lockfile from the actual manifests. Do not hand-author lockfiles. In a workspace, run commands from the correct package context or use the package manager's supported filtering mechanism; do not accidentally lint/build every unrelated app.
3. Update existing project documentation or contributor documentation with the actual application directory, required Node/package-manager setup, install command, development/build/start commands, verification commands, selected configuration path (confirmed shared package or standalone), browser-versus-server boundary conventions, and required environment variables. Do not overwrite documentation or claim commands were run when they were not.

## 6. Meaningful smoke checks

After installation, use the selected local toolchain and package-manager context to run all applicable checks:

1. Confirm the resolved compiler is the one declared `typescript` version and that no TypeScript 6/7 alias or second compiler is used.
2. Run `typecheck` against actual component props and direct source imports, `lint`, `format:check`, optional `lint:styles`, and `build`. Fix issues in files changed for this work; do not mute rules, blanket-ignore source, or reformat unrelated files just to obtain a pass.
   Repeat `format:check` after the build: exclude generated output such as `.next/` or explicitly select source/config files without hiding source.
   Verify the resolved dependency graph has no unmet peer requirements, including transitive dependencies; confirm the chosen TypeScript is the newest stable version within all supported ranges. Check the resolved ESLint configuration enables React, Hooks, and Next rules using temporary violations, and remove those fixtures afterward.
3. Start both the development app and the built app with their real scripts, make a local request to the intended route, and confirm an HTTP success response and distinctive application UI. In rendered browser checks for both modes, verify a component using `.navigation-item` has a nonempty `styles.navigationItem` class and its expected computed style; build/typecheck success alone is insufficient. Exercise at least one actual client interaction while confirming server-rendered content works without moving server logic into the browser. Stop processes afterward.
4. For a shared-config integration, verify each app config resolves the confirmed package from the application directory. For standalone configuration, verify no unresolved external shared-config reference remains.
5. Report exact commands actually run, outcomes, selected versions, files changed, shared-config decision, and any network, environment, or tool blocker. A missing runtime or unavailable network is a blocker to report, not evidence that the app works.

## References

Use current official documentation when executing this skill: [Next.js installation](https://nextjs.org/docs/app/getting-started/installation), [Next.js ESLint configuration](https://nextjs.org/docs/app/api-reference/config/eslint), [Next.js server and client components](https://nextjs.org/docs/app/getting-started/server-and-client-components), [TypeScript tsconfig reference](https://www.typescriptlang.org/tsconfig/), [typescript-eslint](https://typescript-eslint.io/getting-started/), [ESLint flat configuration](https://eslint.org/docs/latest/use/configure/configuration-files), [Prettier configuration](https://prettier.io/docs/configuration), and [Stylelint configuration](https://stylelint.io/user-guide/configure/).
