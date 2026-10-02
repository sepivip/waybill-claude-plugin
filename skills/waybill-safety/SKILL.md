---
name: waybill-safety
description: Use whenever you work with the waybill.ge MCP tools (RS.GE waybills, e-invoices or Balance.ge). Show a waybill or e-invoice draft and get an explicit yes before activate_waybill, close_waybill, confirm_waybill, save_invoice_from_waybill, send_invoice or attach_advance_invoice, and never ask for RS.GE or Balance.ge passwords in chat.
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

Never ask the user to type an RS.GE or Balance.ge password, service
user password or API key into the conversation, and do not repeat one
if they paste it. Credentials belong in the waybill.ge dashboard, under
Business integrations: https://waybill.ge/dashboard/integrations
(RS.GE and Balance.ge companies are connected and managed there).

If a tool reports missing or wrong credentials, point the user to the
dashboard page instead.

## Balance.ge is read-only

The Balance.ge tools only read data. Nothing is ever posted to or
changed in the user's ledger, so no confirmation is needed for them.

If `balance_list_companies` marks a company `needs_repair: true`, do
not pass its label to any other Balance tool and do not run
`balance_check_credentials` on it. Tell the user to disconnect it and
connect the right company again under Business integrations.
