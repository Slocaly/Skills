# Do not wrap Testing Library calls in act

> _Testing Library already wraps its utilities in `act`: an `act` warning is fixed by awaiting what the user would wait for, never by adding `act` around the test._

## TL;DR

- `render`, `userEvent`, `fireEvent`, `findBy*`, `waitFor` and `waitForElementToBeRemoved` already run inside `act`. Do not wrap them again.
- A "not wrapped in act(...)" warning means an update landed after the test moved on: await its visible consequence with `findBy*` or `waitForElementToBeRemoved`.
- Manual `act` is for what Testing Library does not drive: calling a hook's `result.current`, advancing fake timers, updating an external store.

## Why

Wrapping a `render` or an interaction in `act` makes the warning disappear, but it does not say what the test is waiting for. React flushes whatever is pending when the callback ends, so the test passes as long as the async work happens to settle in time, and the next reader cannot tell which update mattered. The real fix is almost always one `await screen.findBy*` on the element the test asserts on: it waits for the same update, documents it, and fails with the rendered DOM when the update never comes. Kent C. Dodds lists this as a common mistake in [Common mistakes with React Testing Library](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library#wrapping-things-in-act-unnecessarily).

## Rules

- `render`, `userEvent` interactions, `fireEvent`, `findBy*`, `waitFor` and `waitForElementToBeRemoved` **MUST NOT** be wrapped in `act`.
- An `act` warning **MUST** be fixed by awaiting the async consequence of the update, through `findBy*` or `waitForElementToBeRemoved`.
- A render helper **MUST NOT** use `act` to flush pending work on behalf of its callers.
- `act` **MAY** be used when the update is not triggered through Testing Library: calling a function exposed by `renderHook` on `result.current`, advancing fake timers, or updating an external store directly.

## Example

**❌ Avoid: act around render to silence the warning**

```tsx
const renderInvitationForm = async () => {
  await act(() => render(<InvitationForm />));
};

it("prefills the legal category", async () => {
  await renderInvitationForm();

  expect(screen.getByRole("combobox", { name: "Forme juridique" })).toHaveValue("Société par actions simplifiée");
});
```

Nothing says which update the test waits for, and the helper hides it from every caller.

**✅ Do this: await what the assertion needs**

```tsx
it("prefills the legal category", async () => {
  render(<InvitationForm />);

  expect(await screen.findByRole("combobox", { name: "Forme juridique" })).toHaveValue(
    "Société par actions simplifiée",
  );
});
```

**✅ Do this: act for a hook call or a timer**

```ts
const { result } = renderHook(() => useCounter());

act(() => result.current.increment());
act(() => vi.advanceTimersByTime(1000));

expect(result.current.count).toBe(1);
```

## Exceptions

- **Hook tests, fake timers and external stores**: the `MAY` clause above. Testing Library does not trigger these updates, so `act` is the only way to flush them.
