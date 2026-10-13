---
name: waybill-safety
description: Use whenever you work with the waybill.ge MCP tools (RS.GE waybills, e-invoices, Balance.ge or QuickShipper). Show a waybill or e-invoice draft and get an explicit yes before activate_waybill, close_waybill, confirm_waybill, save_invoice_from_waybill, send_invoice or attach_advance_invoice, treat QuickShipper quotes as prices rather than bookings, and never ask for RS.GE, Balance.ge or QuickShipper passwords in chat.
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

Never ask the user to type an RS.GE, Balance.ge or QuickShipper
password, service user password or API key into the conversation, and do
not repeat one if they paste it. Credentials belong in the waybill.ge
dashboard, under Business integrations:
https://waybill.ge/dashboard/integrations (RS.GE and Balance.ge companies
and QuickShipper accounts are connected and managed there).

If a tool reports missing or wrong credentials, point the user to the
dashboard page instead.

## Balance.ge is read-only

The Balance.ge tools only read data. Nothing is ever posted to or
changed in the user's ledger, so no confirmation is needed for them.

If `balance_list_companies` marks a company `needs_repair: true`, do
not pass its label to any other Balance tool and do not run
`balance_check_credentials` on it. Tell the user to disconnect it and
connect the right company again under Business integrations.

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
