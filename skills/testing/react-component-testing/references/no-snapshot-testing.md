# Never use snapshot testing for components

> _A component test asserts explicit, user-visible expectations; `toMatchSnapshot` and `toMatchInlineSnapshot` are never used on rendered output._

## TL;DR

- Never call `toMatchSnapshot()` or `toMatchInlineSnapshot()` on a rendered component, container, or `asFragment()`.
- Assert the specific things the user sees and can do (`getByRole`, `toBeVisible`, `toHaveTextContent`, ...).
- A snapshot is not a substitute for deciding what the test is about.

## Why

A snapshot records everything the component renders and fails on any change, relevant or not. It breaks on every restyle, class rename or markup tweak, so people regenerate it with `-u` without reading the diff, and from then on it approves whatever the component does, bugs included. It also never states the intent: a reader cannot tell which part of the output matters. It couples the test to the DOM structure (the opposite of [behavior over implementation](behavior-over-implementation.md)) and produces large, noisy diffs in review.

## Rules

- A component test **MUST NOT** use `toMatchSnapshot`, `toMatchInlineSnapshot`, or `toMatchFileSnapshot`.
- A component test **MUST NOT** snapshot `container`, `asFragment()`, `baseElement` or `prettyDOM` output.
- A test **MUST** assert explicit expectations on what the user sees and can do.
- A failing snapshot **MUST NOT** be fixed by updating it; replace it with explicit assertions.

## Example

**❌ Avoid: snapshotting the rendered output**

```tsx
it("renders the invoice card", () => {
  const { asFragment } = render(<InvoiceCard invoice={createTestingInvoice()} />);

  expect(asFragment()).toMatchSnapshot();
});
```

This passes today and says nothing about what matters. It fails when a class changes and keeps passing when the amount is wrong, once someone runs `-u`.

**✅ Do this: assert the behavior that matters**

```tsx
it("shows the invoice amount and a pay action when unpaid", () => {
  render(<InvoiceCard invoice={createTestingInvoice({ status: "unpaid", amount: "120,00 €" })} />);

  expect(screen.getByText("120,00 €")).toBeVisible();
  expect(screen.getByRole("button", { name: "Payer" })).toBeVisible();
});
```

## Exceptions

None for component tests. Visual regression is a different tool (screenshot comparison in a browser, e.g. Storybook/Playwright), not a component unit test.
