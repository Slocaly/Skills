---
name: react-component-testing
description: Conventions for writing React component tests with Vitest, Testing Library, user-event and TanStack Query. Use when creating, editing, reviewing or debugging a `*.test.tsx` file, or when deciding whether and how to test a React component (queries, mocks, factories, async, providers).
---

# React component testing

How I test React components: behavior over implementation, real data layer cut at the service boundary, queries the way a user finds elements.

## Core principles

1. **Assert what the user sees and can do**, never internals (hooks called, state, component tree).
2. **Mock at the service boundary** (the function a `queryFn`/`mutationFn` calls), never query hooks, child components or child hooks.
3. **Query like a user**: role, then label, then text. `getByTestId` is never introduced.
4. **Interact with `user-event`**, await async UI with `findBy*` (not `waitFor`), wrap suspense renders in `act`.
5. **A test ships with clean output**: no warnings, no `act` noise.

## Read by task

Each reference has a TL;DR, the rationale, MUST/MUST NOT rules, ❌/✅ examples and exceptions. Read the group matching the task before writing or changing a test.

### Deciding what to test (read first when unsure)
- [Test components that carry logic](references/test-logic-only.md) — Every component with branching or behavior gets a test; purely presentational components do not.
- [Keep the default happy-dom environment](references/happy-dom-environment.md) — Tests run in happy-dom; a file opts into jsdom only when happy-dom cannot support what it needs.

### Writing a new test file (read all)
- [Colocate tests in __tests__](references/file-placement.md) — A component test lives in a `__tests__` folder next to the component and is named after it.
- [Render through the shared providers](references/render-with-providers.md) — Component tests render through `src/test/render-with-providers`, import `screen` from Testing Library, and hide repeated setup behind a local render helper.
- [Assert behavior, not implementation](references/behavior-over-implementation.md) — A component test asserts what the user sees and can do, never the internals that produce it.
- [Follow the Testing Library query priority](references/query-priority.md) — Query elements the way a user finds them: role first, then label, then text; `getByTestId` is a last resort you never introduce.
- [Mock services, not hooks or children](references/mock-services-not-hooks.md) — Cut the dependency at the service function the query calls, never at the query hook or the child component.

### Setting up data and mocks
- [Declare mocks above the tests](references/mocks-above-tests.md) — Module mock declarations sit at the top of the file, above the first `describe`; only per-scenario values live inside a test.
- [Build mock data with createTesting factories](references/testing-factories.md) — Mock entities come from a `createTesting*` factory in `src/test`, overriding only the fields the scenario cares about.
- [Use the shared test utilities](references/shared-test-utilities.md) — Permissions, feature flags, form context, and search params are controlled through the helpers in `src/test`, never by re-mocking what `setup.ts` already mocks.

### Interactions and async UI
These three form one cluster: async warnings usually come from skipping one of them.
- [Interact with user-event](references/user-event.md) — User interactions go through `@testing-library/user-event`, with `userEvent.setup()` called at the start of every test that interacts.
- [Await findBy instead of waitFor](references/find-by-over-wait-for.md) — An element that appears asynchronously is awaited with `findBy*`, not polled with `waitFor` around a `getBy*`.
- [Wrap suspense renders in act](references/act-on-suspense-render.md) — When the rendered tree uses a suspense query, `await act(() => render(...))` so the suspended update flushes before the assertions.

### Assertions
- [Assert visibility, not presence](references/assert-visibility.md) — Presence uses `getBy*` with `.toBeVisible()`; absence uses `queryBy*` with `.not.toBeInTheDocument()`.
- [Query the rendered string, not the i18n key](references/literal-translated-text.md) — Text queries and assertions use the literal string the user reads, never `t('namespace:key')`.

### Before shipping / debugging warnings
- [Ship tests with a clean output](references/no-warnings.md) — A test that emits a warning is not finished: fix the cause before it ships (often a missing `act`, `findBy*` or `userEvent.setup()`).

## Project-specific conventions

Several rules reference this project's setup (`src/test`, `render-with-providers`, `createTesting*` factories, globals mocked in `src/test/setup.ts`, happy-dom). Treat those paths and helper names as the reference implementation and adapt them to the project at hand; the principles above are project-independent.
