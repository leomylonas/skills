# TypeScript & React Project Style Guide

**Status:** Agreed project conventions  
**Applies to:** React single-page applications, Next.js applications, React UI/component libraries, and general-purpose TypeScript utility libraries.  
**Audience:** Developers and automated project-bootstrap/provisioning agents.

This document specifies the intended conventions for newly provisioned projects. Apply all applicable rules to each project type; React-only rules do not apply to non-React libraries. Framework-mandated filenames, exports, and directory conventions take precedence over these defaults where necessary. Avoid adding infrastructure or abstractions solely to satisfy an example in this guide.

## 1. Principles

1. **Organise by domain or function first**, not by a single global technical category.
2. **Use PascalCase consistently for directories and ordinary source filenames**, with explicitly stated exceptions (notably React hooks and framework-required names).
3. **Colocate implementation details**: components, CSS modules, tests, private hooks, and private child components belong together.
4. **Prefer explicit dependencies and imports** over re-export registries and barrel files.
5. **Preserve encapsulation and ownership boundaries**; do not make implementation details public just for convenience.
6. **Keep abstractions proportional to actual needs**: avoid premature sharing, excessive global state, unnecessary dependency-injection containers, and unnecessary wrapper directories.
7. **Enforce mechanical conventions through linting/TypeScript where practical**, including autofixes for type-only imports. The exact lint configuration is deliberately left to the provisioning agent.

## 2. Directory organisation and naming

### 2.1 Domain-first structure

Group code by business domain, feature, or coherent responsibility. Technical subdirectories such as `Components`, `Hooks`, `Services`, and `Types` may be used *within* a domain when warranted, but are not obligatory for small features.

```text
src/
  App.tsx
  Layout.tsx
  Global.css
  Features/
    Orders/
      Components/
        OrderList/
          OrderList.tsx
          OrderList.module.css
          OrderList.test.tsx
          OrderRow/
            OrderRow.tsx
        OrderStatusBadge/
          OrderStatusBadge.tsx
      Hooks/
        useOrders.ts
      Services/
        OrderService.ts
      Types/
        Order.ts
    Customers/
      Components/
        CustomerDetails/
          CustomerDetails.tsx
  Shared/
    Components/
      Navigation/
        Navigation.tsx
        Navigation.module.css
        NavigationItem/
          NavigationItem.tsx
      Button/
        Button.tsx
    Hooks/
      useMediaQuery.ts
    Utils/
      CurrencyUtils.ts
```

The example is illustrative, not a mandatory scaffold: create only directories that have a purpose. A `Shared` directory is for genuinely cross-domain code, not speculative future reuse.

### 2.2 Casing rules

| Item | Convention | Example |
| --- | --- | --- |
| Directories, including domain and grouping directories | PascalCase | `Features/Orders/Components/` |
| React component source files | PascalCase | `Navigation.tsx` |
| React component directory | PascalCase; same as component | `Navigation/Navigation.tsx` |
| Component tests | Match source basename | `Navigation.test.tsx` |
| Component CSS modules | Match source basename | `Navigation.module.css` |
| Ordinary TypeScript utility files | PascalCase | `CurrencyUtils.ts`, `FormatCurrency.ts` |
| Ordinary TypeScript class names | PascalCase | `CurrencyUtils` |
| Ordinary function names | camelCase | `formatCurrency` |
| React hook functions and filenames | camelCase, beginning with `use` | `useOrders.ts`, `useOrders()` |
| Type aliases, classes, props types | PascalCase | `NavigationProps`, `Order` |
| Traditional class-oriented interfaces | `I` + PascalCase | `ILogger` |
| CSS class selectors | kebab-case | `.navigation-item` |
| CSS Module property access in TypeScript | Exact kebab-case key | `styles['navigation-item']` |
| Framework-reserved files/paths | Follow framework convention | `app/page.tsx`, `app/layout.tsx` |

**Important:** Do not mix kebab-case feature folders with PascalCase component folders. Directory casing is consistently PascalCase except where a framework requires otherwise. Hook filenames are a deliberate exception to the ordinary PascalCase file rule because they match their hook function names. A utility file may be named `FormatCurrency.ts` even when it exports `formatCurrency`.

### 2.3 Framework precedence

Follow conventions required by the actual framework/toolchain, including Next.js special files such as `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, and `route.ts`. Framework-required default exports are also exceptions to the named-export preference. Do not rename reserved files or routes just to meet the general PascalCase policy.

## 3. Component structure and ownership

### 3.1 Component directories by default

A React component should **normally** have its own PascalCase directory containing a same-named `.tsx` file. Keep its CSS module, unit/component tests, private hooks, and private child components inside that directory.

```text
Navigation/
  Navigation.tsx
  Navigation.module.css
  Navigation.test.tsx
  useNavigation.ts
  NavigationItem/
    NavigationItem.tsx
    NavigationItem.module.css
  NavigationGroup/
    NavigationGroup.tsx
```

This is an architectural default, not a rule triggered only by the presence of CSS or tests. A navigation component warrants its own directory even if it starts as a single file, because it represents a coherent UI subsystem.

### 3.2 Narrow exception: lightweight composition components

A standalone `.tsx` file may be used for a lightweight application-level component whose main purpose is composition/delegation and which does not own meaningful internal UI implementation. Typical examples include `App.tsx` and a top-level `Layout.tsx`.

```text
src/
  App.tsx
  Layout.tsx
  Shared/
    Components/
      Header/
        Header.tsx
      Navigation/
        Navigation.tsx
      Footer/
        Footer.tsx
```

```tsx
// Layout.tsx — lightweight composition; no directory necessary
import type { ReactNode } from 'react';
import { Header } from './Shared/Components/Header/Header';
import { Navigation } from './Shared/Components/Navigation/Navigation';
import { Footer } from './Shared/Components/Footer/Footer';

type LayoutProps = {
  children: ReactNode;
};

export function Layout(props: LayoutProps) {
  const { children } = props;

  return (
    <>
      <Header />
      <Navigation />
      <main>{children}</main>
      <Footer />
    </>
  );
}
```

This exception is **not** a general exemption for any component that currently lacks a CSS module, test, or child component. Simple presentational and reusable UI components still normally get directories. Do not impose an arbitrary line-count threshold. If a standalone composition component grows into a substantive subsystem, give it a directory.

### 3.3 Nesting and reuse are based on ownership

- A child component used only as an implementation detail of a parent belongs beneath the owning component's directory.
- A component reused by multiple components in the *same feature* belongs at an appropriate feature-level location, not inside one consumer's private tree.
- A component genuinely shared across features belongs in an appropriate shared location.
- Rendering a component does **not** imply owning it. A layout can compose Header, Navigation, and Footer without nesting their implementations under Layout.
- Do not move a component into `Shared` merely because it might someday be reusable.

```text
Features/
  Orders/
    Components/
      OrderDetails/
        OrderDetails.tsx
        OrderActions/
          OrderActions.tsx     # private to OrderDetails
      OrderSummary/
        OrderSummary.tsx       # shared within Orders
  Customers/
    Components/
      CustomerDetails/
        CustomerDetails.tsx
Shared/
  Components/
    Button/
      Button.tsx               # genuine cross-feature use
```

Do not access another feature's private component internals merely to reuse a helper. Respect explicit feature/module boundaries and extract a suitable abstraction where needed.

## 4. React component declarations and props

Use **named function declarations** for React components. Declare props with a **type alias** named `<ComponentName>Props`, rather than an interface. Accept the entire object as `props: ComponentNameProps` and **destructure props on the first line inside the function body** (not in the parameter list). This is the standard pattern; do not interpret the policy as a ban on destructuring inside the function.

```tsx
import type { NavigationItemModel } from '../../Types/NavigationItemModel';
import { NavigationItem } from './NavigationItem/NavigationItem';
import styles from './Navigation.module.css';

type NavigationProps = {
  items: NavigationItemModel[];
  selectedId?: string;
};

export function Navigation(props: NavigationProps) {
  const { items, selectedId } = props;

  return (
    <nav className={styles['navigation-container']}>
      {items.map(item => (
        <NavigationItem
          key={item.id}
          item={item}
          selected={item.id === selectedId}
        />
      ))}
    </nav>
  );
}
```

Avoid default exports except where a framework explicitly requires them. A Next.js App Router page, for example, may use `export default function Page()` because Next.js requires a default-exported page component.

```tsx
// app/orders/page.tsx — framework-required default export
import { OrderList } from '../../Features/Orders/Components/OrderList/OrderList';

export default function Page() {
  return <OrderList />;
}
```

## 5. Hooks and business logic

- Keep hooks private to a component colocated with that component.
- Place hooks shared by multiple components in a feature-level `Hooks/` directory.
- Put genuinely cross-feature hooks in `Shared/Hooks/` (or an equivalent shared package).
- Extract substantial React-specific state/effect coordination into hooks when doing so improves clarity or reuse.
- Keep non-React business logic independent of React where practical, particularly in TypeScript utility libraries.
- Hook functions and filenames use their normal `useSomething` camelCase spelling.

```text
Features/
  Orders/
    Components/
      OrderSearch/
        OrderSearch.tsx
        useOrderSearch.ts      # private to OrderSearch
    Hooks/
      useOrders.ts            # shared within Orders
```

```ts
// Features/Orders/Components/OrderSearch/useOrderSearch.ts
import { useMemo, useState } from 'react';
import type { Order } from '../../Types/Order';

export function useOrderSearch(orders: Order[]) {
  const [query, setQuery] = useState('');
  const filteredOrders = useMemo(
    () => orders.filter(order => order.reference.includes(query)),
    [orders, query],
  );

  return { query, setQuery, filteredOrders };
}
```

## 6. TypeScript types, interfaces, and safety

### 6.1 Prefer `type`

Use `type` for object shapes, props, DTOs, domain models, unions, intersections, and most other TypeScript declarations. Reserve `interface` for **traditional class-oriented contracts**, particularly contracts implemented by classes; such interfaces use an `I` prefix.

```ts
// Types/Order.ts
export type Order = {
  id: string;
  reference: string;
  status: 'pending' | 'paid' | 'cancelled';
};

export type OrderResponseDto = {
  order_id: string;
  status: string;
};

// Services/ILogger.ts
export interface ILogger {
  log(message: string): void;
}

// Services/ConsoleLogger.ts
import type { ILogger } from './ILogger';

export class ConsoleLogger implements ILogger {
  public log(message: string): void {
    console.log(message);
  }
}
```

Keep DTOs, domain models, and view models distinct when their responsibilities or representations differ; do not create needless duplicates when they do not.

### 6.2 Type-only imports are mandatory

Use `import type` for dependencies used only in type positions. Configure ESLint to enforce and automatically fix this where possible. Prefer compiler/linter settings that preserve clear runtime/type boundaries.

```ts
import type { Order } from '../Types/Order';
import { OrderService } from '../Services/OrderService';
```

### 6.3 Strictness and `any`

- Enable TypeScript strict checking.
- Prefer precise types and `unknown` where values genuinely require validation or narrowing.
- Explicit `any` is **lint-disallowed by default**, but **technically permitted** with a narrowly scoped lint suppression in exceptional circumstances.
- Use type-aware unsafe-`any` lint rules where practical to catch unsafe propagation from third-party types, not only explicit `any` syntax.
- Do not disable the `any` rule globally or add broad file-wide suppressions to bypass the policy.
- A suppression should be as narrow as possible and explain why the exception is necessary.

```ts
// Exceptional compatibility boundary with an untyped legacy API:
// eslint-disable-next-line @typescript-eslint/no-explicit-any -- Legacy callback API supplies an untyped payload; validate before use.
export function receiveLegacyPayload(payload: any): unknown {
  return payload;
}
```

The example demonstrates the suppression mechanism, **not** a recommendation to use `any` at integration boundaries. Prefer `unknown` and validation when possible.

### 6.4 Other TypeScript conventions

- Prefer string literal unions or `as const` objects to enums where suitable.
- Define types close to their owner; extract shared types only when genuinely shared.
- Do not use `I` prefixes for ordinary object type aliases or props.

## 7. Imports, exports, and module boundaries

### 7.1 No barrel exports

Do **not** create `index.ts`/`index.tsx` files to aggregate or re-export sibling modules. Import from the **defining source module**, not a barrel. This avoids maintaining export registries, unnecessary indirection, and some dependency-graph complications.

```tsx
// Preferred: explicit source import
import { Navigation } from '../../Shared/Components/Navigation/Navigation';

// Avoid: directory/barrel import
// import { Navigation } from '../../Shared/Components';
```

The reason is **not** that barrel exports always defeat tree shaking: modern ESM bundlers can often tree-shake re-exports. The stronger reasons are explicitness, encapsulation, reduced maintenance, and clearer dependency graphs.

For the library variants covered here, **libraries are consumed through direct source imports**; do not create a public package entry point or export barrel merely out of habit. If a framework or required toolchain mandates a special entry file, follow that requirement without using it as a reason to introduce general-purpose barrels.

### 7.2 Named exports and relative imports

- Prefer named exports; avoid default exports unless a framework/toolchain requires them.
- Prefer **relative imports for references within the same package**, including imports between features and shared code.
- Do not mandate an import ordering convention. No automatic grouping or sorting is required by this policy.
- Use `import type` for type-only references.
- Keep module dependency boundaries explicit; never bypass encapsulation just because a relative path is possible.

```ts
import type { Order } from '../Types/Order';
import { CurrencyUtils } from '../../../Shared/Utils/CurrencyUtils';
```

### 7.3 Dependency cycles

- **Runtime circular dependencies are forbidden.**
- **Type-only circular imports are permitted**, but should still be avoided where practical. Prefer extracting a shared abstraction or contract when it improves the dependency graph.
- Do not force an abstraction solely to remove a harmless type-only cycle when the abstraction would be more confusing than the dependency.

### 7.4 Dependency injection and encapsulation

Apply dependency-injection concepts where they improve testability, substitution, and coupling. **No DI container/provider is required**. Constructor parameters, ordinary function parameters, and React props are valid ways to inject dependencies. Avoid gratuitous interfaces and service abstractions.

```ts
// Services/ILogger.ts
export interface ILogger {
  log(message: string): void;
}

// Services/OrderService.ts
import type { ILogger } from './ILogger';
import type { Order } from '../Types/Order';

export class OrderService {
  public constructor(private readonly logger: ILogger) {}

  public getDisplayReference(order: Order): string {
    this.logger.log(`Formatting order ${order.id}`);
    return order.reference;
  }
}
```

In React, simple prop injection is often sufficient:

```tsx
type OrderActionsProps = {
  orderId: string;
  onCancel: (orderId: string) => void;
};

export function OrderActions(props: OrderActionsProps) {
  const { orderId, onCancel } = props;

  return <button onClick={() => onCancel(orderId)}>Cancel</button>;
}
```

## 8. State ownership and data fetching (React projects)

Use the smallest appropriate scope for state:

1. Prefer local component state (`useState`/`useReducer`) for local concerns.
2. Lift state to the nearest common owner when multiple components need it.
3. Use React Context for suitable shared subtree concerns.
4. Introduce dedicated application-wide client-state management only when justified.
5. Treat **server state** separately from client UI state: caching, invalidation, loading, and errors belong to an appropriate data-fetching strategy.

Avoid copying server data into client state without a reason. Do not provision a global state library merely because the project uses React.

```tsx
// Local state example; no global store necessary
import { useState } from 'react';

type OrderFilterProps = {
  onChange: (query: string) => void;
};

export function OrderFilter(props: OrderFilterProps) {
  const { onChange } = props;
  const [query, setQuery] = useState('');

  function handleChange(value: string) {
    setQuery(value);
    onChange(value);
  }

  return (
    <input
      value={query}
      onChange={event => handleChange(event.target.value)}
      aria-label="Filter orders"
    />
  );
}
```

## 9. CSS and styling (React projects)

### 9.1 CSS Modules only for application/component styles

- Use CSS Modules for application and component styling.
- Permit **one top-level global CSS stylesheet import** for application-wide baseline/global styling (such as resets, root tokens, and foundational element rules).
- Additional imports of **vendor-provided stylesheets** are allowed when required by dependencies.
- Colocate a component's CSS module with that component, using matching PascalCase filename.
- Use **kebab-case selectors in CSS** and access their exact keys in TypeScript, for example `styles['navigation-item']`.
- Use class-specific TypeScript declarations or equivalent tooling so valid keys typecheck and misspelled keys fail. A generic string index signature does not detect typos. Keep declarations synchronized with their CSS, including when a stylesheet is renamed or deleted.
- Prefer native CSS nesting.
- Inline styles are allowed **only when genuinely necessary**, such as a dynamic value derived from state or props; do not use them for ordinary static presentation.
- A parent may override child styling or layout when needed, but prefer child-owned styling and explicit styling APIs where practical.

```css
/* Navigation.module.css */
.navigation-container {
  display: flex;
  gap: 0.5rem;

  .navigation-item {
    padding: 0.5rem;

    &:hover {
      background-color: var(--surface-hover);
    }
  }
}
```

```tsx
import styles from './Navigation.module.css';

type NavigationProps = {
  children: React.ReactNode;
};

export function Navigation(props: NavigationProps) {
  const { children } = props;

  return <nav className={styles['navigation-container']}>{children}</nav>;
}
```

**Tooling:** Vite and Next.js/Turbopack can use the original CSS Module keys without a custom naming transformation. Verify that the selected type tooling resolves the same exact keys at compile time and that styles apply at runtime; generated typings alone do not prove the runtime mapping.

### 9.2 Global and vendor styles

```tsx
// Application bootstrap (illustrative)
import './Global.css';
import 'some-vendor-package/dist/styles.css'; // only when needed
```

A Next.js app may need its global CSS import in the framework-prescribed root layout rather than a generic SPA entry file. This is a framework-specific exception.

### 9.3 Dynamic inline values

```tsx
import type { CSSProperties } from 'react';
import styles from './ProgressBar.module.css';

type ProgressBarProps = {
  progress: number;
};

export function ProgressBar(props: ProgressBarProps) {
  const { progress } = props;
  const boundedProgress = Math.max(0, Math.min(100, progress));

  return (
    <div
      className={styles['progress-bar']}
      style={{ '--progress': `${boundedProgress}%` } as CSSProperties}
    />
  );
}
```

```css
/* ProgressBar.module.css */
.progress-bar {
  width: var(--progress);
}
```

This is an appropriate narrow use of inline styles: the changing value comes from props; the presentation rules remain in the CSS module.

## 10. Testing

- Colocate unit and React component tests with their implementation.
- Use `*.test.ts` and `*.test.tsx` filenames matching the source basename.
- Default to **Vitest** for unit tests and **React Testing Library** for React component tests where compatible with the project/toolchain.
- Test **observable behaviour**, not private implementation details.
- Prefer mocking external boundaries (network, storage, clock, injected services) rather than mocking internal helpers or component implementation.
- Use snapshots selectively; do not make snapshots the default testing strategy.
- Keep application-level end-to-end tests in a separate E2E area, rather than inside individual component directories.
- Keep test fixtures/helpers close to the tests that own them; promote them only when genuinely shared.
- A component may still deserve its own directory even before tests are written. Directory policy must not discourage testing.

```tsx
// OrderFilter.test.tsx
import { fireEvent, render, screen } from '@testing-library/react';
import { expect, test, vi } from 'vitest';
import { OrderFilter } from './OrderFilter';

test('notifies the consumer when the query changes', () => {
  const onChange = vi.fn();
  render(<OrderFilter onChange={onChange} />);

  fireEvent.change(screen.getByRole('textbox', { name: 'Filter orders' }), {
    target: { value: 'ABC' },
  });

  expect(onChange).toHaveBeenCalledWith('ABC');
});
```

For a standalone TypeScript utility library, colocated `.test.ts` files and Vitest remain applicable, while React Testing Library and React-specific test guidance do not.

## 11. Utility modules and non-React TypeScript libraries

The same domain-first organisation, PascalCase directories/files, `type`-first declarations, named exports, explicit relative source imports, no barrels, strictness, test colocation, and dependency-boundary rules apply to ordinary TypeScript libraries.

A preferred utility pattern is a **PascalCase class containing static methods**, though standalone functions are allowed. Do not require utility classes when a function is materially simpler or more appropriate.

```text
src/
  Currency/
    CurrencyUtils.ts
    CurrencyUtils.test.ts
    Types/
      CurrencyCode.ts
  Dates/
    DateUtils.ts
    DateUtils.test.ts
```

```ts
// Currency/CurrencyUtils.ts
export class CurrencyUtils {
  public static format(amount: number, currency: string): string {
    return new Intl.NumberFormat('en-AU', {
      style: 'currency',
      currency,
    }).format(amount);
  }
}
```

```ts
// Consumer: direct source import, not an index barrel
import { CurrencyUtils } from '../Currency/CurrencyUtils';

const displayAmount = CurrencyUtils.format(12.5, 'AUD');
```

An equally valid exception where appropriate:

```ts
// Currency/FormatCurrency.ts
export function formatCurrency(amount: number, currency: string): string {
  return new Intl.NumberFormat('en-AU', {
    style: 'currency',
    currency,
  }).format(amount);
}
```

The file remains PascalCase even though the standalone function is camelCase.

## 12. Project-type applicability

| Rule family | React SPA | Next.js | React UI library | TS utility library |
| --- | --- | --- | --- | --- |
| Domain-first organisation, PascalCase directories | Yes | Yes, except reserved framework paths | Yes | Yes |
| Component folders/props/hooks | Yes | Yes | Yes | Not applicable |
| CSS Modules and styling rules | Yes | Yes, subject to framework CSS handling | Yes | Not applicable unless UI is involved |
| TypeScript, type imports, `any` lint policy | Yes | Yes | Yes | Yes |
| Explicit relative imports, named exports, no barrels | Yes | Yes, with mandatory framework exceptions | Yes | Yes |
| Direct source consumption; no package entrypoint required | N/A | N/A | Yes | Yes |
| State hierarchy and server-state separation | Yes | Yes | Where applicable | Not applicable |
| Accessible UI and reporting observed barriers | Yes | Yes | Yes | Where a library produces user-facing output |
| Responsive UI and viewport verification | Yes | Yes | Yes | Where a library produces user-facing output |
| Colocated unit tests, boundary-focused mocking | Yes | Yes | Yes | Yes |
| Application E2E test separation | Yes | Yes | Where applicable | Not generally applicable |

## 13. Provisioning and enforcement guidance

When bootstrapping a project:

1. Apply these conventions as the **default generated structure and example code**, not merely as documentation.
2. Scaffold only meaningful directories; do not precreate empty `Components`, `Hooks`, `Services`, or `Shared` trees without a need.
3. Generate React components using named functions, `type` props, `props: PropsType`, and first-line body destructuring.
4. Generate direct relative imports to defining source files; avoid barrel `index.ts` files and unnecessary package export registries.
5. Configure TypeScript strictness and lint enforcement for applicable mechanical rules. In particular, enforce/autofix type-only imports, flag explicit `any` (with narrowly documented suppressions), and prevent runtime circular dependencies.
6. Ensure CSS Module types expose exact selector keys, reject misspellings and stay synchronized with their source stylesheets.
7. Keep required framework files, names, exports, and CSS import locations intact.
8. Include the style guide in the provisioned project so maintainers and coding agents can follow it. The exact destination/integration mechanism is up to the provisioning skill.
9. Treat architectural choices such as ownership, appropriate sharing, DI, and encapsulation as review guidelines where they cannot be fully lint-enforced.
10. Do not invent additional mandatory tooling, import ordering, public entry points, global state libraries, DI containers, or unrelated policies that this guide does not require.

## 14. Accessibility

Accessibility is a default requirement, not an optional enhancement. When implementing a requested feature or changing user-facing output, choose an accessible implementation from the outset; do not wait for a separate accessibility request. Aim for the applicable [WCAG 2.2 AA success criteria](https://www.w3.org/TR/WCAG22/) and current platform guidance, while recognising that automated checks alone cannot establish conformance.

- Prefer semantic HTML and native controls with correct names, roles and states. Use ARIA only when native semantics are insufficient, and keep it accurate as state changes.
- Make functionality operable by keyboard, with a logical focus order, visible focus, appropriate focus management, and no keyboard traps. Do not make a pointer, hover, colour or animation the only way to understand or operate something.
- Provide meaningful labels and instructions for inputs, accessible names for controls, useful text alternatives for informative images, and error messages that identify the affected field and explain how to correct the problem.
- Use clear document structure and headings, readable contrast, scalable and responsive layouts, and respect reduced-motion preferences. Announce meaningful dynamic updates when necessary without overwhelming assistive technology.
- Preserve or improve accessibility when changing existing UI. Exercise relevant keyboard, focus, labels, state transitions and error flows in a real browser; use available automated accessibility checks as a supplement, not a substitute for manual and assistive-technology verification. Report checks actually performed and any verification limitations; never claim WCAG conformance without a suitable assessment.
- If you notice an existing accessibility barrier, **call it out to the developer**, including its location, user impact and a practical remedy. Fix issues within the requested change when appropriate; do not silently make unrelated or breaking changes. Report unresolved barriers rather than ignoring them.

## 15. Responsive design

Responsive design is required for all user-facing interfaces and components, not a later enhancement. Design and implement the requested behavior so it remains usable across narrow and wide viewports, different input methods and text sizes; preserve this behavior when modifying existing UI.

- Let content and layout adapt to available space using flexible sizing, wrapping and appropriate CSS breakpoints or container queries. Choose breakpoints based on the content, not assumptions about specific devices.
- Keep text readable and controls usable without clipping, overlap or unintended page-level horizontal scrolling. Preserve access to navigation, form feedback and actions at narrow widths. Content such as large data tables may need deliberate local scrolling when its two-dimensional layout is essential.
- Respect browser zoom and text enlargement; don't disable zoom or rely on a fixed viewport. Keep keyboard and touch interaction practical and retain visible focus and accessible names at all sizes.
- When UI behavior or layout changes, verify the actual rendered surface at representative narrow and wide viewport sizes and at intermediate widths where the layout changes. Exercise relevant navigation and interactions, inspect overflow, and check zoom/text enlargement where appropriate. For reusable components, verify in a realistic consuming layout. Automated viewport checks can supplement visual and interaction checks.
- Report the viewport sizes and checks actually performed, as well as any blocker or unverified condition. Call out observed responsive defects in existing code with their location, impact and a practical remedy; avoid unrelated or breaking changes without approval.

## 16. Out of scope for this guide

This guide does not prescribe detailed performance, error-boundary, loading/empty-state, or memoisation policies. It also does not prescribe a complete ESLint rule list, a specific DI framework, a specific global state library, or mandatory import ordering. Those may be addressed separately.

---

**Summary:** Domain-first structure; PascalCase directories and ordinary filenames; hook-name exception; component folders by default with a narrow lightweight-composition exception; nested private children; colocated tests/styles/hooks; named functions and exports; `type`-first TypeScript with `I`-prefixed class contracts; props destructured on the first line of the function body; strict typing with lint-controlled `any` exceptions; CSS Modules with exact kebab-case keyed TypeScript access; explicit relative source imports without barrels; no runtime dependency cycles; thoughtful encapsulation and lightweight dependency injection; accessible, responsive implementations and reporting observed barriers; framework conventions override where required.