# Await the loaded UI after a suspense render

> _When the rendered tree uses a suspense query, the first assertion awaits the loaded UI so the suspended data resolves before anything is checked._

## TL;DR

- Tree contains a `use<Entity>SuspenseQuery` (in the component or any parent)? The first assertion is `await screen.findBy*` on the loaded UI.
- When the test asserts an absence, first `await waitForElementToBeRemoved` on the suspense fallback, then `queryBy*`.
- No `act` around the render (see [no-manual-act](no-manual-act.md)).

## Why

A suspense query throws a promise on first render and React resolves it after the synchronous test body has moved on. A synchronous `getBy*` then runs against the fallback, and React logs "A suspended resource finished loading inside a test, but the event was not wrapped in act(...)". `findBy*` and `waitForElementToBeRemoved` run inside `act` and retry until the data has resolved, which removes both the race and the warning. Wrapping the render in `act` hides the same race without saying what the test waits for.

## Rules

- When the component under test, or any parent in the rendered tree, uses a suspense query hook, the first assertion **MUST** await the loaded UI with `findBy*`.
- A test asserting that an element is absent from a suspense tree **MUST** first `await waitForElementToBeRemoved` on the fallback, then use `queryBy*`.
- A suspense render **MUST NOT** be followed by a synchronous `getBy*` or `queryBy*` before the loaded UI has been awaited.
- The `render` call **MUST NOT** be wrapped in `act`.

## Example

**❌ Avoid: synchronous assertion on a suspense tree**

```tsx
it("shows data", () => {
  render(<MyComponent />);

  expect(screen.getByText("Données chargées")).toBeVisible();
});
```

The assertion runs against the fallback, and the output carries an `act` warning.

**✅ Do this: await the loaded UI**

```tsx
it("shows data", async () => {
  render(<MyComponent />);

  expect(await screen.findByText("Données chargées")).toBeVisible();
});

it("hides the action when nothing was loaded", async () => {
  render(<MyComponent />);
  await waitForElementToBeRemoved(() => screen.queryByText("suspense boundary..."));

  expect(screen.queryByRole("button", { name: "Supprimer" })).not.toBeInTheDocument();
});
```

## Exceptions

- **Non-suspense queries** (`use<Entity>Query`) expose loading state instead of suspending: assert the loading UI, then use `findBy*` for the loaded UI.
- **Asserting the fallback itself**: assert it synchronously right after `render`, before awaiting anything.
