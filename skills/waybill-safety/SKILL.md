---
name: waybill-safety
description: Use whenever you work with the waybill.ge MCP tools (RS.GE waybills, e-invoices, Balance.ge, QuickShipper, 1C (OneSoft), Bank of Georgia, the NAPR public registry or NBG exchange rates). Show a waybill or e-invoice draft and get an explicit yes before activate_waybill, close_waybill, confirm_waybill, save_invoice_from_waybill, send_invoice or attach_advance_invoice, treat QuickShipper quotes as prices rather than bookings, keep 1C trial balance windows narrow, remember that the Bank of Georgia tools only read and never move money, never search the public registry for people or report a guess when NAPR or NBG is unavailable, and never ask for RS.GE, Balance.ge, QuickShipper or 1C passwords or Bank of Georgia API keys in chat.
---

# waybill.ge safety

The waybill.ge tools act on real tax documents at the Georgian Revenue
Service (RS.GE). Some of them cannot be undone.

## Confirm before binding actions

Before you call any of these tools, show the user exactly what will
happen and wait for an explicit "yes" (or an equally clear instruction)
in their latest message:

- `activate_waybill`: sends the draft to RS.GE and makes it legally
  effective. It can later be cancelled, never returned to draft.
- `close_waybill`: marks the goods as delivered.
- `confirm_waybill`: accepts a waybill another company issued to you.
- `save_invoice_from_waybill`: creates an RS.GE invoice from a waybill,
  or changes an existing one when an invoice id is given.
- `send_invoice`: sends a saved e-invoice (advance or regular) to the
  buyer. RS.GE gives it its series and number; it cannot be unsent.
- `attach_advance_invoice`: offsets an advance invoice's VAT against a
  supply invoice, which changes the VAT that invoice charges.
- `save_advance_invoice` or `save_supply_invoice` with a non-zero
  `invoice_id`: rewrites an existing saved invoice and replaces all its
  lines.

`cancel_waybill` and `reject_waybill` are also final: confirm them the
same way.

For a new waybill, always call `save_waybill_draft` first. Show the
draft (buyer name and TIN, route, goods, quantities, prices, total,
transport details and the draft id) and ask whether to activate it.
Do not treat an earlier general request ("issue a waybill to X") as
approval to activate: ask again once the draft exists.

For a new e-invoice, call `save_advance_invoice` (a prepayment) or
`save_supply_invoice` (a delivery) with no `invoice_id` first. Show the
saved draft (buyer name and TIN, operation month, lines, amount and VAT,
invoice id) and ask whether to send it. To complete an advance, find it
with `list_attachable_advance_invoices`, save the supply invoice, then
ask before `attach_advance_invoice` and again before `send_invoice`.

If the user says anything other than a clear yes, do not call the tool.
Offer to change the draft instead.

## Never ask for passwords in chat

Never ask the user to type an RS.GE, Balance.ge, QuickShipper or 1C
password, service user password or API key, or a Bank of Georgia
Business Online client ID or secret, into the conversation, and do not
repeat one if they paste it. Credentials belong in the waybill.ge
dashboard, under Business integrations:
https://waybill.ge/dashboard/integrations (RS.GE and Balance.ge companies,
QuickShipper accounts, 1C publications and Bank of Georgia connections
are connected and managed there; the bank's section is
https://waybill.ge/dashboard/integrations#bog).

If a tool reports missing or wrong credentials, point the user to the
dashboard page instead.

## Balance.ge is read-only

The Balance.ge tools only read data. Nothing is ever posted to or
changed in the user's ledger, so no confirmation is needed for them.

If `balance_list_companies` marks a company `needs_repair: true`, do
not pass its label to any other Balance tool and do not run
`balance_check_credentials` on it. Tell the user to disconnect it and
connect the right company again under Business integrations.

## Bank of Georgia is read-only

The `bank_*` tools only read the balances and statements of the Bank of
Georgia accounts the user selected in the dashboard. They never move
money: nothing is paid, transferred or signed at the bank, and no tool
can do it, so never offer to. No confirmation is needed for them.

- The user never pastes Business Online keys in chat. The client ID and
  secret of the Business Online API client go on the dashboard connect
  page (https://waybill.ge/dashboard/integrations#bog, then Bank of
  Georgia, Connect), which has the step-by-step guide. If the user pastes
  one anyway, do not repeat it; point them to that page.
- Pass a label from `bank_list_connections` as `connection` when the
  user has more than one. `bank_list_transactions` and
  `bank_statement_summary` read one selected account over at most 31
  days. Transaction amounts are positive; `direction` gives the sign.
- The statement summary never covers today. When it returns
  `covered_to` and a `note`, use `bank_list_transactions` for the
  missing days. `truncated: true` means the list is incomplete; follow
  its `hint`.
- `REAUTH_REQUIRED` means Bank of Georgia no longer accepts the saved API
  key: the user updates it under Business integrations. A connection with
  `accounts: []` cannot be read until the user adds an account there.
- Disconnecting deletes the key from waybill.ge but does not revoke it at
  the bank. To end it there, the user deletes the API client in Business
  Online.

## QuickShipper is read-only

The QuickShipper tools only read. `quickshipper_get_delivery_quote`
prices a route; it does not book a courier, and nothing is created,
changed or cancelled at QuickShipper, so no confirmation is needed.

- A quote needs both street addresses with their cities and both
  points' coordinates. If you do not have the coordinates, ask the user
  for a map pin or the coordinates; do not guess them. Prices are live
  and change, so say a quote is not a booking.
- `quickshipper_list_orders` takes Tbilisi calendar dates (YYYY-MM-DD,
  both included) and one status. To count orders, read `total` instead
  of paging through every page.
- If a QuickShipper connection needs a new login (`REAUTH_REQUIRED`),
  point the user to Business integrations to update it.

## 1C (OneSoft) is read-only

The `onesoft_*` tools only read the user's own 1C publication. Nothing
is created, changed, posted or unposted in 1C, so no confirmation is
needed. Pass a label from `onesoft_list_connections` as `connection`
when the user has more than one.

- `onesoft_trial_balance` can take minutes: 1C adds up the whole ledger
  for the window first. Ask for the narrowest window that answers the
  question (a week where you can), tell the user it may take a while, and
  do not retry a slow call in a loop. `balanced: false` is a finding
  about the ledger, not an error; report it with the figures.
- `onesoft_list_contracts` needs one counterparty's `ref_key` from
  `onesoft_list_clients` or `onesoft_list_vendors`. Its rows may be
  trading points or delivery routes rather than signed agreements.
- In 1C balances `advances` is negative and `net` is `debt + advances`;
  never subtract advances from debt.
- If `onesoft_check_connection` reports `NOT_PUBLISHED` or
  `CONTENT_EMPTY`, a 1C administrator has a one-time step to do: point
  them to https://waybill.ge/dashboard/integrations/onesoft/setup. For
  `REAUTH_REQUIRED`, the user updates the 1C login under Business
  integrations.

## The public registry and NBG rates are read-only

The `napr_company_lookup`, `napr_lien_check`, `napr_property_lookup` and
`nbg_exchange_rates` tools only read public sources: Georgia's National
Agency of Public Registry (NAPR) and the National Bank of Georgia (NBG).
They need no connection or company label, and no confirmation.

- They look up companies and property, never people. They take a legal
  entity's 9-digit identification code (the same number as its RS.GE
  TIN), part of its registered name in Georgian script, a cadastral code
  or an application's registration number, and they refuse personal ID
  numbers. Do not offer to search for a person.
- Look up one counterparty at a time; these tools are not for bulk
  checks.
- `SERVICE_UNAVAILABLE` from these tools means the public source did not
  answer, is paused after a captcha, or the account made too many
  registry lookups in a minute. Say so and offer to try again later
  (`retry_after_seconds` when given). Never report a status, "no liens"
  or a rate you did not receive.
- An NBG `rate` is GEL per `quantity` units (AMD per 1000, JPY per 100),
  so use `rate_per_unit` for one unit. On a weekend or holiday
  `valid_from` shows the earlier set in force. These are official
  reference rates, not a bank's buy or sell rates; attribute them to the
  National Bank of Georgia.

## When a tool call fails

- `SERVICE_UNAVAILABLE` with `retryable: true` (or an HTTP 503):
  waybill.ge could not reach its own database for a moment, and nothing
  was sent to RS.GE or any connected service. Retry once after a few
  seconds (`retry_after_seconds` when given). Do not ask the user to
  reconnect.
- `INTERNAL_ERROR`: an unexpected error on waybill.ge. Give the user its
  `request_id` for support. Before repeating a binding action (activate,
  close, confirm, send), check the document's current state: the first
  attempt may already have reached RS.GE.
- Ask the user to reconnect waybill.ge only when the host reports an
  authentication failure (HTTP 401), never for the errors above.
