# Module 14 — Frontend-Specific AI Review

*[← Previous: Verification](../03-workflow/06-verification.md) | [Next: AI for Testing →](./02-ai-for-testing.md)*

## Why this matters

A generic "review this code" pass will happily tell you a component is clean, well-typed, and handles errors — and still miss the fact that it just put server data (fetched from an API, owned by a cache, subject to going stale) into a global Zustand store where nothing will ever invalidate it. It will miss that a "modal" is a `div` with no focus trap, so a keyboard user tabs straight through it into the page behind. It will miss that a table renders three action buttons 800px off the right edge of a phone screen. None of these are bugs a general-purpose correctness review is built to find, because none of them are about whether the logic is right in isolation — they're about frontend-specific dimensions: where state actually lives, how the browser's rendering and focus model behaves, and what the same component looks like on a screen it was never checked against. This module gives you six review lenses to point at frontend code specifically, on top of the general verification habits from Module 13.

## Objective

Apply AI specifically to frontend engineering concerns — the dimensions that only exist because this is a browser UI and not a generic function: component lifecycle, state ownership, fetching semantics, render performance, accessibility, and layout across viewports. General code review checks whether logic is correct. This module is about the checks that are specific to building interfaces people actually look at, click on, and depend on across devices.

---

## 14.1 React / Vue

Hooks and composables are syntactically easy for AI to produce — the shape of `useEffect(() => {...}, [deps])` or a Vue `watchEffect` is extremely well-represented in training data, so a model can generate something that *looks* like a correct hook almost every time. The problem is that the dependency array and the reactivity system are not decorative — they are a contract about what the code depends on, and a model optimizing for "this looks like the effects I've seen" will produce one that lies about that contract just as fluently as one that's accurate. AI also cannot see your component tree while looking at a single file, so it has no way to know whether a piece of state it's about to introduce already lives one level up, one level down, or in a store elsewhere — which is exactly how state duplication and blurred component boundaries sneak in.

Review:

* Hooks/composables
* Dependency arrays
* Effects
* Reactive dependencies
* Unnecessary renders
* State duplication
* Component boundaries

**Worked example — dashboard widget.** Suppose you ask AI to add an auto-refresh behavior to a revenue widget on an internal dashboard, so it re-fetches every 30 seconds without a full page reload. It produces:

```tsx
function useAutoRefresh(fetchData: () => Promise<void>, intervalMs: number) {
  useEffect(() => {
    const id = setInterval(() => {
      fetchData();
    }, intervalMs);
    return () => clearInterval(id);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);
}

function RevenueWidget({ dateRange }: { dateRange: DateRange }) {
  const [data, setData] = useState<RevenuePoint[]>([]);

  const fetchData = async () => {
    const res = await getRevenue(dateRange);
    setData(res);
  };

  useAutoRefresh(fetchData, 30_000);

  return <Sparkline points={data} />;
}
```

This runs, refreshes every 30 seconds, and looks correct in a quick demo. The `eslint-disable-next-line` is the tell — AI silenced the warning instead of resolving it, which is itself worth flagging on sight: a disabled lint rule on generated code should read as "the model hit a warning it didn't know how to fix," not "this is fine." What actually happens: `fetchData` is created fresh on every render, but the effect's empty dependency array means the interval only ever captures the *first* render's `fetchData`, which closed over the *first* render's `dateRange`. If the user changes the dashboard's date range filter afterward, the widget keeps silently refreshing with the original range every 30 seconds — the UI shows the new range briefly after the initial fetch, then reverts to stale data on the next tick, with no error and no visual indication anything is wrong.

```text
Review this hook and the component using it.

Focus only on:
- dependency array correctness
- stale closures
- whether disabling exhaustive-deps here is masking a real bug
```

Bug class this catches: a polling or interval-driven effect that silently ignores prop or filter changes because its callback closed over first-render values — the widget appears to "auto-refresh" while actually refreshing the wrong data forever.

---

## 14.2 State Management

This is one of the most reliable places to catch AI making an architectural decision it never actually made — it just followed the path of least resistance. If a Zustand or Redux store is already open in the file or already imported elsewhere in the feature, a model asked to "store this data so other components can read it" will often just add it to that store, because syntactically that's the shortest path to "components can read it now." What it's not doing is asking whether that data is server state that already has an owner — your query cache — with its own invalidation, refetching, and staleness rules. The moment the same data exists in two places, they will eventually disagree, and the bug that results (a badge that doesn't match the list it's counting) has no stack trace pointing at the actual cause.

Ask:

```text
Local state?
Server state?
Global state?
URL state?
```

AI can easily put server state into a global store when it does not belong there.

**Worked example — checkout cart.** Say AI is asked to make the cart item count available to a header badge component that lives outside the cart page's component tree. It reaches for the existing Zustand store already used for UI state elsewhere in the app:

```tsx
const useCartStore = create<CartState>((set) => ({
  items: [],
  setItems: (items) => set({ items }),
}));

function CartPage() {
  const { data: cartItems } = useCartQuery(); // React Query — the real source of truth
  const setItems = useCartStore((s) => s.setItems);

  useEffect(() => {
    if (cartItems) setItems(cartItems);
  }, [cartItems, setItems]);

  const storeItems = useCartStore((s) => s.items);
  return <CartBadge count={storeItems.length} />;
}
```

This works the first time the cart page loads. But now there are two copies of the cart: the React Query cache (correct, invalidated automatically on mutation) and the Zustand store (a snapshot, synced only by that one `useEffect`). When the user removes the last item, the `removeFromCart` mutation invalidates and refetches the query — but the header badge, which reads `useCartStore` directly and is rendered outside `CartPage`, still shows the old count until something re-triggers that sync effect. The bug ships as: "I emptied my cart but the header still says 3 items," and it has nothing to do with the removal logic itself, which is correct — it's a state-ownership bug.

```text
Review this state management.

Classify every piece of state as:
- local
- server
- global
- URL

Flag anything that duplicates server state into the global store.
```

Bug class this catches: server-owned data (cart items) duplicated into a global client store, so the two copies drift after any mutation and the UI reads whichever one happens to be stale.

---

## 14.3 Data Fetching

Fetching bugs are temporal — they depend on the order and timing of events, not on what any single render looks like — which makes them close to invisible to a model reasoning over a static diff. AI reliably produces a `fetch` call that works for the one sequence it was implicitly asked about (open the panel once, wait, get data), and just as reliably omits cancellation, dedupe, and stale-response guards, because none of those exist in the "happy path" shape of the request. The result is code that is correct in the demo and wrong the moment a real user does something faster than the demo assumed — reopens a panel quickly, types two characters before a request resolves, or is on a spotty connection where retries and cancellations start to interleave.

Review:

* Cache behavior
* Stale data
* Race conditions
* Duplicate requests
* Retry behavior
* Cancellation
* Loading state
* Error state

**Worked example — notifications panel.** Suppose AI implements the data-loading side of a notifications bell dropdown:

```tsx
function useNotifications(isOpen: boolean) {
  const [notifications, setNotifications] = useState<Notification[]>([]);

  useEffect(() => {
    if (!isOpen) return;
    fetchNotifications().then(setNotifications);
  }, [isOpen]);

  return notifications;
}
```

Opened once, this is fine. But nothing here cancels the in-flight request if `isOpen` flips again before it resolves. If a user clicks the bell, changes their mind and closes it, then reopens it a moment later — a completely ordinary interaction, not an edge case — two requests are now in flight. There is no guarantee the first one resolves first. If the *older* request happens to resolve *after* the newer one (slower connection, server load, anything), its `.then(setNotifications)` fires last and silently overwrites the correct, fresh data with the stale response from the first fetch — with nothing in the UI to indicate it happened. The user sees notifications that are already read, or missing one that just arrived, and has no reason to suspect the panel is lying to them.

```text
Review this data fetching only for:
- race conditions
- request cancellation
- duplicate requests

Assume the panel can be opened and closed rapidly.
```

Bug class this catches: an out-of-order response overwriting correct data because nothing tracks which request is the most recent one — fixed by an `AbortController` or an "ignore stale response" flag keyed to the request that fired it, not just a dependency-array re-run.

---

## 14.4 Performance

Performance problems in frontend code are almost always about *how often* something runs, not whether it's correct when it runs — and "how often" is precisely the dimension a model reading a diff has the hardest time reasoning about, because it requires simulating render cascades across parent and child components, not just reading one function top to bottom. AI-generated list and table code in particular tends to be correct-but-naive: it filters and formats inline in the render path, without asking whether that computation needs to happen on every keystroke of an unrelated input, or whether 2,000 DOM rows really need to exist at once just because 2,000 items exist in memory.

```text
Analyze potential performance issues.

Focus on:
- unnecessary renders
- expensive calculations
- large lists
- memoization
- bundle size
- network requests
- image loading
```

**Worked example — order history page.** Suppose AI wires up a search box on an order history page with roughly 2,000 rows, filtered client-side:

```tsx
function OrderHistoryPage() {
  const [filters, setFilters] = useState({ keyword: "", status: "all" });
  const orders = useOrders(); // ~2,000 orders, loaded up front

  return (
    <div>
      <SearchInput
        value={filters.keyword}
        onChange={(keyword) => setFilters({ ...filters, keyword })}
      />
      <OrderTable orders={orders} filters={filters} />
    </div>
  );
}

function OrderTable({ orders, filters }: Props) {
  const visible = orders.filter((o) => o.customerName.includes(filters.keyword));
  return (
    <table>
      <tbody>
        {visible.map((o) => (
          <OrderRow key={o.id} order={o} total={formatCurrency(computeOrderTotal(o))} />
        ))}
      </tbody>
    </table>
  );
}
```

Functionally this filters correctly. Performance-wise, every keystroke in the search box re-renders `OrderHistoryPage`, which passes a brand-new `filters` object down, which re-runs the `.filter()` over all 2,000 orders and recomputes `computeOrderTotal` for every row that survives the filter — none of it memoized, none of it debounced, and all 2,000 potential `<OrderRow>` elements are real DOM nodes with no virtualization. On a fast dev machine with 20 seeded rows this feels instant. In production with real order volume, typing in the search box visibly lags a keystroke or two behind, and no one on the review caught it because nothing about the code looks wrong — it's just doing far more work than it needs to, every time.

```text
Analyze potential performance issues.

Focus only on:
- unnecessary renders
- expensive calculations
- large lists
- memoization
```

Bug class this catches: an O(n) filter and per-row calculation re-run on every keystroke with no memoization and no list virtualization, producing input lag that only shows up at realistic data volume.

---

## 14.5 Accessibility

Accessibility bugs are structurally invisible to a code review that only reads text and to a sighted engineer clicking through a UI with a mouse — the DOM can look and behave correctly for the interaction pattern everyone actually tests, while being completely broken for keyboard-only or screen-reader users. AI is trained almost entirely on code as text, not on the experience of operating that code with `Tab`, `Escape`, and a screen reader, so it can produce a dialog that is pixel-perfect and passes a visual review while having no `role`, no focus trap, and no announcement of its own state changes — because nothing in "build a confirmation dialog" as a prompt surfaces those requirements unless you ask for them explicitly.

Review:

* Semantic HTML
* Keyboard navigation
* Focus management
* ARIA
* Screen reader behavior
* Dialog behavior
* Form labels

**Worked example — delete-account confirmation.** Suppose AI builds the "type DELETE to confirm" dialog for the account-deletion flow on a settings page:

```tsx
function DeleteAccountDialog({ isOpen, onClose, onConfirm }: Props) {
  const [confirmText, setConfirmText] = useState("");
  if (!isOpen) return null;

  return (
    <div className="dialog-overlay">
      <div className="dialog">
        <h2>Delete your account?</h2>
        <input value={confirmText} onChange={(e) => setConfirmText(e.target.value)} />
        <button onClick={onClose}>Cancel</button>
        <button onClick={() => confirmText === "DELETE" && onConfirm()}>Delete</button>
      </div>
    </div>
  );
}
```

Visually, this is a correct-looking modal: overlay, heading, input, two buttons. Operate it without a mouse and several things fall apart at once. There is no `role="dialog"` or `aria-modal="true"`, so a screen reader has no idea this content is a modal rather than more page — it will happily keep reading the page behind it. Nothing moves focus into the dialog when it opens, and nothing traps focus inside it, so pressing `Tab` a few times walks straight past the dialog into whatever is behind the overlay, even though the overlay visually blocks clicking there. `Escape` does nothing. The `<input>` has no associated `<label>`, so a screen reader announces it as an unlabeled text field instead of "type DELETE to confirm." And when the typed text doesn't match "DELETE," the Delete button just silently stays inert — there is no `aria-live` region announcing why, so a screen reader user has no feedback loop telling them what's wrong.

```text
Review this dialog only for accessibility.

Focus on:
- focus management
- keyboard navigation
- ARIA roles
- screen reader announcements
- form labels
```

Bug class this catches: a "modal" that is only modal in appearance — keyboard and screen-reader users can tab straight through it into the page behind, can't dismiss it with `Escape`, and get no feedback when their input is invalid.

---

## 14.6 Responsive UI

A model reasoning about a component has no viewport — it reasons about the JSX or template as an abstract tree, not as pixels rendered at 375px versus 1440px, so unless a prompt says "check this on mobile," nothing pushes it to consider a width narrower than whatever implicit desktop layout dominates its training data. Fixed pixel widths, multi-column grids sized for a monitor, and wide tables all tend to be internally consistent and "correct" at that implicit width, which is exactly why the resulting layout bug is invisible in a code review — you cannot see a horizontal scrollbar, or a button rendered off-screen, by reading a diff.

Check:

```text
Mobile
Tablet
Desktop
Long content
Empty state
Error state
Loading state
```

**Worked example — admin orders table.** Suppose AI builds a wide data table for an internal admin dashboard, with per-row action buttons:

```tsx
function OrdersTable({ orders }: Props) {
  return (
    <table style={{ width: "1200px" }}>
      <thead>{/* customer, email, date, status, total, actions */}</thead>
      <tbody>
        {orders.map((o) => (
          <tr key={o.id}>
            <td>{o.customerName}</td>
            <td>{o.email}</td>
            <td>{o.date}</td>
            <td>{o.status}</td>
            <td>{o.total}</td>
            <td>
              <button>View</button>
              <button>Refund</button>
              <button>Cancel</button>
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

At desktop width this renders exactly as intended. There is no wrapping `overflow-x: auto` container and no responsive column strategy, so on a 375px phone screen the fixed 1200px table simply overflows: the "Refund" and "Cancel" buttons render roughly 800 pixels past the right edge of the visible viewport, reachable only if the whole page happens to scroll horizontally — which most mobile layouts explicitly suppress. Support staff trying to process a refund from a phone cannot find the button at all; nothing crashes, nothing errors, the feature is just physically unreachable on that screen size. Checking "long content" catches a second, related issue: a customer name long enough to wrap pushes the row height up unevenly and misaligns it against adjacent columns, because nothing constrains the cell's text handling.

```text
Check this component at:
Mobile
Tablet
Desktop
Long content
```

Bug class this catches: a fixed-width table with no horizontal scroll container and no responsive column strategy, making critical actions physically unreachable on a phone-sized viewport even though the code itself has no logic error.

---

## Key Takeaways

* A general "is this correct" review does not automatically check frontend-specific dimensions — state ownership, fetch timing, render frequency, accessibility, and viewport behavior each need their own explicit pass.
* State duplication is the most common architectural mistake AI makes silently: it will put server-owned data into a global store just because the store is already open in the file, not because that's where the data belongs.
* Fetching bugs (races, duplicate requests, stale responses) are temporal — they only appear across a sequence of user actions, so a static read of the code will not surface them; you have to prompt for them explicitly, or trigger the sequence yourself.
* Performance and accessibility problems both share a common trait: the code looks completely normal and the happy-path demo works, but the failure only appears at scale (thousands of rows) or under a different mode of interaction (keyboard, screen reader) that the review never exercised.
* Responsive bugs are invisible in a diff by definition — you cannot see a horizontal scrollbar or an off-screen button in source code, so this is one dimension where you must actually render the component at multiple widths, not just ask AI to reason about it in the abstract.
* Each of the six review lenses in this module works best as a narrow, explicit prompt ("review only for accessibility," "review only for race conditions") rather than folded into one generic review — the same lesson from Module 12, applied to frontend-specific failure modes.

## Try It Yourself

1. Pick a component in your codebase that reads from both a data-fetching hook (React Query, SWR, or equivalent) and a global store (Zustand, Redux, Context). Run the 14.2 prompt against it — classify every piece of state it touches as local, server, global, or URL — and check whether any server-owned data has been copied into the global store. If you find one, trace what would happen the next time that data mutates: does the global copy get invalidated, or does it silently go stale?
2. Take one modal, drawer, or dropdown you already ship in production and operate it with your mouse put away: open it with the keyboard, tab through it, try `Escape`, and turn on your OS screen reader (VoiceOver, NVDA, or your platform's equivalent) for two minutes. Note every gap against the 14.5 checklist (focus management, keyboard navigation, ARIA, screen reader announcements, form labels), then run the accessibility review prompt against the component's source and compare what the checklist and the prompt each caught that the other missed.

---

*[← Previous: Verification](../03-workflow/06-verification.md) | [Next: AI for Testing →](./02-ai-for-testing.md)*
