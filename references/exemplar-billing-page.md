> An exemplar of the shape and register. The product is invented; the
> specificity is the point. Every file, string and number in a real brief
> comes from the map of the real codebase.

# Brief: one Billing page

## Thesis

Billing is split across three places that were built at different times.
`app/(app)/settings/billing/page.tsx` renders a plan card with a
hand-rolled `<div role="button">` for the upgrade, `PlanPicker.tsx` opens a
modal built from a fixed `<div>` and a `useEffect` that adds an Escape
listener, and `InvoiceTable.tsx` is a raw `<table>` with inline
`style={{ color: '#6b7280' }}` cells and a `window.confirm` before
downloading a PDF. The page says `Stripe customer` and `subscription_id`
to the person, and `Active` in green even when the card on file expired
last month. Nothing uses the `components/ui` library the rest of the app
moved to in March; the page has its own `Button`, its own `Badge`, and a
`formatMoney` that disagrees with `lib/format.ts` on rounding.

Make it **one billing page**: a status line that says what the account is
right now, the plan and the card as rows, the invoices as a table that
reads like the rest of the app, and one dialog for every change. A person
should be able to tell in two seconds whether they are paid up, what they
pay, and what to do if they are not.

## Approach

Rebuild the page on `components/ui`; delete the local copies. The data
layer stays exactly as it is: `lib/billing/queries.ts` (`useSubscription`,
`useInvoices`, `usePaymentMethod`), the route handlers under
`app/api/billing/*` (`GET subscription`, `POST change-plan`, `POST portal`,
`GET invoices/:id/pdf`) and the Stripe webhook in
`app/api/webhooks/stripe/route.ts`. The page's own `Button`, `Badge`,
`Modal` and `formatMoney` are deleted once nothing imports them;
`lib/format.ts` is the one money formatter. Add to `components/ui` only
when a piece is needed twice; a billing-only component stays in
`app/(app)/settings/billing/`.

Rejected: sending people to Stripe's hosted portal for everything. The
portal handles cards and cancellation well and stays for those two, but a
plan change through it loses the in-app confirmation of what the price
becomes today, which is the question people actually have.

## Scope

**The status line.** At the top, under the page title `Billing`: a dot and
one sentence. `Paid through 14 October` when the subscription is active
and the next charge is scheduled; `Payment failed on 2 October — update
your card` in the danger colour when the last invoice is `open` after a
failed attempt; `Trial ends in 6 days` during a trial; `On the free plan`
otherwise. The sentence comes from `useSubscription` alone; no second
request.

**The plan row.** A `ListPanel` with one row: the plan name, the price as
`$24 / month` from `lib/format.ts`, and a `Change plan` button
(`Button variant="outline" size="sm"`). On the free plan the button reads
`Upgrade`. During a trial the row adds a `Trial` `Badge`.

**The card row.** The same panel, second row: the brand and last four as
`Visa · 4242`, the expiry, and `Update card` as a link-style button that
opens the Stripe portal in a new tab. When the card has expired the expiry
is in the danger colour and the row carries the same sentence as the
status line, once, so a person sees it wherever they look first.

**Changing the plan.** `Change plan` opens a `Dialog` `size="md"`:
title `Change your plan`, the plans as rows with a drawn radio
(`ChoiceRow`), each with the name, the price, and one line on what it adds.
Under the rows, one sentence that changes with the selection: `You'll pay
$48 today, prorated, then $48 a month from 14 October.` or `Your plan
changes on 14 October; nothing is charged today.` The footer is `Cancel`
(`Button variant="ghost"`) and `Change plan` (the CTA), disabled until a
different plan is selected. Sending calls `POST /api/billing/change-plan`;
success closes the dialog and `toast.success('Plan changed')`; failure
keeps the dialog open with the server's message in a `Callout tone="danger"`
above the footer.

**Invoices.** The registry `Table`: `Date`, `Amount`, `Status`, and a
trailing `Download` icon button. Dates from `Intl.DateTimeFormat`, amounts
right-aligned with `tabular-nums`, status as a `Badge`: `Paid`, `Open`,
`Void`, `Refunded`. Twenty rows a page with the registry pager; empty state
`No invoices yet` with one line on when the first one arrives. Download
opens the PDF route in a new tab with no confirmation.

**Cancelling.** A `Cancel subscription` link at the bottom of the page,
tertiary text, opens the portal; the page does not build its own
cancellation flow.

**Narrow columns.** Under 640px the rows stack their trailing controls
under the label; the table hides the `Status` column and shows the status
as a dot before the amount.

## Non-negotiables

1. **Registry only.** `Button`, `Badge`, `Dialog`, `ChoiceRow`, `Table`,
   `Callout`, `toast`, `ListPanel` from `components/ui`; no raw
   `<table>`, `<button>` or `<div role="button">`; no inline `style`.
2. **One money formatter.** Every amount goes through `lib/format.ts`;
   the page's `formatMoney` is deleted.
3. **Chroma is meaning.** Green only for `Paid`; the danger colour only
   for a failed payment, an expired card, or `Open` past due; nothing else
   on the page is tinted.
4. **Copy is for the person.** No `Stripe`, `customer`, `subscription_id`,
   `price_id`, `webhook` in the UI. Buttons say what happens: `Change
   plan`, `Update card`, `Download`.
5. **The proration sentence is the server's number.** The dialog shows the
   amount from `POST /api/billing/change-plan?preview=1`, never a client
   calculation.
6. **Destructive actions confirm somewhere.** Cancellation lives in the
   portal, which confirms; the page never cancels in one click.
7. **Data layer untouched.** No new route handlers, no changes to the
   webhook, the queries or their cache keys.
8. **Both states of every row are drawn.** Trial, free, active, past due,
   expired card: each has a fixture and a screenshot.
9. **Existing tests keep passing.** `billing.spec.tsx` is extended for the
   status line and the dialog's preview sentence, not rewritten.

## What to cover

A fixture route at `/dev/billing`, listed with the other fixtures, that
renders the page from `lib/billing/fixtures.ts` in five states: free,
trial, active, past due, expired card, plus the dialog open with a plan
selected. Both themes, 1280px and 375px.

## Verification

- `grep -rn "style={{\|window.confirm\|<table" app/(app)/settings/billing`
  returns nothing.
- `grep -rn "Stripe\|subscription_id\|price_id" app/(app)/settings/billing`
  returns nothing outside `lib/`.
- The status sentence for each fixture state matches the copy above, word
  for word.
- Selecting a plan in the dialog issues one preview request and the
  sentence updates with the server's amount.
- `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build`; stage; ask
  before committing. `docs/design.md` gains a "Billing" paragraph: the
  status line, the rows, the dialog, where cancellation lives.

One decision before I start: the portal keeps card updates and
cancellation, and the page owns plan changes. Say so if you'd rather the
page own all three, which adds a card form and a cancellation dialog to
the scope.
