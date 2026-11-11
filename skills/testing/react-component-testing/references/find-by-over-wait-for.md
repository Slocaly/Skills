# Await findBy instead of waitFor

> _An element that appears asynchronously is awaited with `findBy*`, not polled with `waitFor` around a `getBy*`._

## TL;DR

- Asynchronous element → `expect(await screen.findByRole(...)).toBeVisible()`.
- Do not wrap `getBy*` in `waitFor`: `findBy*` already retries until it appears or times out.
- Keep `waitFor` for conditions that are not "an element showed up".
- Async work on mount with nothing new on screen (a validation, a prefill)? Still `await findBy*` on the element you assert on.

## Why

`findBy*` is `waitFor` plus `getBy*`, so the wrapper adds a layer and loses something: when it times out, `waitFor` reports the last thrown error from inside the callback, which is noisier than the "unable to find an element with role X" message `findBy*` produces, complete with the rendered DOM. The wrapper also invites putting several assertions or a side effect inside the callback, where they get retried repeatedly and produce failures that make no sense.

## Rules

- An element that appears asynchronously **MUST** be queried with `await findBy*`.
- `waitFor` **MUST NOT** wrap a `getBy*` query whose only purpose is waiting for the element to appear.
- A `waitFor` callback **MUST NOT** contain side effects, only assertions.
- Asserting that something never appears **MUST** use `queryBy*` after the awaited state change, not a timeout.
- When the component runs async work on mount that renders no new element (a form `trigger()`, a prefill from storage), the test's first assertion **MUST** be an `await findBy*` on the element it checks, even if that element is already on screen.
- When that async work does change something visible (an error message, an enabled button), the test **SHOULD** await that change instead.

## Example

**❌ Avoid: waitFor around getBy**

```tsx
await waitFor(() => {
  expect(screen.getByRole("option", { name: "Alsace Vosges" })).toBeInTheDocument();
});
```

**✅ Do this: findBy**

```tsx
expect(await screen.findByRole("option", { name: "Alsace Vosges" })).toBeVisible();
```

**❌ Avoid: synchronous assertion while a validation runs on mount**

```tsx
render(<InvitationForm />);

expect(screen.getByText("Certaines informations ont été préremplies")).toBeVisible();
```

The form calls `trigger()` in an effect; the validation resolves after the test ends and React logs an `act` warning.

**✅ Do this: findBy on the asserted element**

```tsx
render(<InvitationForm />);

expect(await screen.findByText("Certaines informations ont été préremplies")).toBeVisible();
```

`findBy*` resolves on the first match, but it runs inside `act` and lets pending work settle before returning, which flushes the validation.

## Exceptions

- **Waiting on something other than an element**: a mock called, a URL changed, an element removed (`waitForElementToBeRemoved`). `waitFor` is correct there.
- **Waiting for an element to reach a state** it did not have on appearance (a button becoming enabled) is a legitimate `waitFor` on an already-found element.
