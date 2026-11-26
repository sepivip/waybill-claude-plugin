---
name: waybill-safety
description: Use whenever you work with the waybill.ge MCP tools (RS.GE waybills and e-invoices, Balance.ge, QuickShipper, 1C, the banks, NAPR or NBG). Show a waybill or e-invoice draft and get an explicit yes before activate_waybill, close_waybill, send_waybill_to_transporter, confirm_waybill, correct_waybill, save_invoice_from_waybill, save_invoice_from_waybills, update_invoice_lines, correct_invoice, send_invoice, attach_advance_invoice, confirm_special_invoice, reject_special_invoice or confirm_special_invoice_cancellation, confirm saved-draft edits before update_waybill_draft and activation, find e-invoices with list_invoices instead of stepping through ids, sum a month's VAT with vat_period_summary, treat QuickShipper quotes as prices rather than bookings, keep 1C trial balance windows narrow, remember that the bank tools only read and never move money, never search the public registry for people or report a guess when NAPR or NBG is unavailable, and never ask for passwords or API keys in chat.
---

# waybill.ge safety

The waybill.ge tools act on real tax documents at the Georgian Revenue
Service (RS.GE). Some of them cannot be undone.

## Confirm before draft edits and binding actions

Before you call any of these tools, show the user exactly what will
happen and obtain an explicit "yes" (or an equally clear instruction):

- `update_waybill_draft`: overwrites saved fields and can remove goods
  lines. Show the company, draft id, changed fields and goods before
  and after, removed lines and the new total.
- `activate_waybill`: sends the draft to RS.GE and makes it legally
  effective. It can later be cancelled, never returned to draft.
- `close_waybill`: marks the goods as delivered.
- `send_waybill_to_transporter`: forwards a carrier draft (transport
  type 7) to its transporter company, which then adds its own driver
  and vehicle, activates the waybill and later closes it. It may not be
  reversible.
- `confirm_waybill`: accepts a waybill another company issued to you.
- `correct_waybill`: corrects a waybill RS.GE has already issued
  (active or closed). RS.GE records every correction and the buyer sees
  it. Show each changed line before and after, and the new total.
- `save_invoice_from_waybill`: creates an RS.GE invoice from a waybill,
  or rewrites the waybill's own saved invoice when its `in_inv_id` is
  given (the tool refuses any other invoice, and one already sent).
- `save_invoice_from_waybills`: saves an invoice draft for the goods on
  one or more waybills plus extra lines that were not on a waybill
  (labour, a call-out, transport), or rewrites a saved invoice when its
  `invoice_id` is given. Show the waybills, every line with its source
  (waybill or extra), the extras total and the invoice total.
- `update_invoice_lines`: renames, changes, adds or deletes lines of a
  saved (unsent) invoice. Show each changed line before and after, the
  added and removed lines, and the new total.
- `correct_invoice`: prepares a correction of an e-invoice the company
  already sent (a new price or quantity, goods returned, a changed or
  cancelled operation). It is saved unsent, and an invoice has one
  unsent correction at a time. Show the invoice's lines and the new full
  lines, the new total and the VAT change.
- `send_invoice`: sends a saved e-invoice (advance or regular), or an
  unsent correction, to the buyer. RS.GE gives it its series and number;
  it cannot be unsent.
- `attach_advance_invoice`: offsets an advance invoice's VAT against a
  supply invoice, which changes the VAT that invoice charges.
- `confirm_special_invoice`: confirms a petroleum special invoice
  (სპეც-ანგარიშ-ფაქტურა) a supplier sent to the company, or a sent
  correction of one. It is a tax act the buyer cannot undo. Call it with
  `validate_only: true` first and show the seller, number, operation
  date, amount and VAT.
- `reject_special_invoice`: rejects a received special invoice with a
  reason the seller sees. Call it with `validate_only: true` first and
  show the invoice and the exact reason.
- `confirm_special_invoice_cancellation`: confirms the seller's
  cancellation of a received special invoice. Call it with
  `validate_only: true` first and show the invoice.
- `save_advance_invoice` or `save_supply_invoice` with a non-zero
  `invoice_id`: rewrites an existing saved invoice and replaces all its
  lines.

`cancel_waybill` and `reject_waybill` are also final: confirm them the
same way.

An explicit instruction already given for the displayed targets and effect
is sufficient. If the draft, company, amount or effect changes, confirm the
changed action. Do not infer permission from urgency, a read request or a
request to prepare a draft.

For a new waybill, always call `save_waybill_draft` first. Show the
draft (buyer name and TIN, route, goods, quantities, prices, total,
transport details and the draft id) and ask whether to activate it.
Show the buyer and driver names the result returns: RS.GE stores the
registered name of a checked TIN, and a `notice` says when it differs
from the name sent.
Do not treat an earlier general request ("issue a waybill to X") as
approval to activate: ask again once the draft exists.

If something on a draft is wrong (a quantity, price, line, address or
driver), show the proposed edits and obtain a clear instruction before
`update_waybill_draft`. Then show the changes it returns, including
anything RS.GE changed on its own
(`also_changed`). Never activate a draft you know is wrong and leave the
user to fix it by hand.

When a transporter company (for example a 3PL) carries the goods, save
a carrier draft instead: `save_waybill_draft` with `transport_type` 7
and the transporter's `transporter_tin`, on type 1 or type 2 only, and
leave `vehicle`, `driver_tin`, `driver_name` and `trans_txt` empty.
Show the draft with the transporter's name and TIN and ask whether to
send it to that transporter, saying it may not be reversible. Only
after a clear yes, call `send_waybill_to_transporter`. Never call
`activate_waybill` or `close_waybill` on a carrier waybill: the
transporter does both, and the tools refuse. `get_waybill` and
`list_seller_waybills` show its `transporter_state`. If a save, send,
update or correction returns `SAVE_UNVERIFIED`, tell the user what the
message says and check the waybill with `get_waybill` before doing
anything else.

For a new e-invoice, call `save_advance_invoice` (a prepayment) or
`save_supply_invoice` (a delivery) with no `invoice_id` first. Show the
saved draft (buyer name and TIN, operation month, lines, amount and VAT,
invoice id) and ask whether to send it. To complete an advance, find it
with `list_attachable_advance_invoices`, save the supply invoice, then
ask before `attach_advance_invoice` and again before `send_invoice`.

Without a clear instruction for the displayed action, do not call the
tool. Offer to prepare revised details instead.

To correct an active or closed waybill, read it with `get_waybill`
first: its `edit` says which tool can change it, or why none can. Pass
only what changes to `correct_waybill`: lines by their `items[].id`, new
lines, removed lines, addresses, driver, vehicle, transport cost or
comment. Leave `amount` out: a line's amount is its quantity times its
price. A correction keeps the waybill's status, type and buyer. A
carrier waybill is corrected only after its transporter has taken it,
and its driver, vehicle and transport type stay as the transporter set
them; say it is a carrier waybill when you show the change. RS.GE also
changes some dates on its own when it saves a correction (a start date
earlier than the activation time, a closed waybill's close and delivery
dates): when `edit.message` names them, show them with the change before
the yes, and after the save tell the user what `also_changed` lists. The
tool refuses a cancelled waybill or a sub-waybill, and RS.GE refuses
some periods and some closed waybills; then nothing changed, so tell the
user to correct it on eservices.rs.ge.

RS.GE does not edit a waybill linked to an e-invoice: `edit.tool` is
`correct_invoice`. Correct the invoice instead, with the waybill's
`invoice_id`; the waybill stays as it is, as on RS.GE's own corrections.
Read the invoice with `get_invoice`, then pass `correct_invoice` the
`reason` (`price_changed`, `goods_returned`, `operation_changed` or
`cancelled`) and the invoice's NEW full lines, every line that stays
with its new quantity and amount, not the difference. For goods that
came back, issue the return waybill (type 5) to the same buyer first,
then pass its number as `return_waybill_number` with the goods that were
kept (`[]` when everything came back). Show the user the old and new
lines and `change` (the VAT difference) and get a yes before
`correct_invoice`; it saves an unsent correction and returns its id.
Show that correction and ask again before `send_invoice` with that id.
One unsent correction at a time: if the tool answers with a
`correction_id`, change that correction's lines with `correct_invoice`
on its id instead of making another. A corrected invoice is corrected
again through its newest correction (`newest_invoice_id`). Advance
invoices and invoices with an advance offset are corrected on
eservices.rs.ge.
If the corrected waybill has no invoice yet, ask before
`save_invoice_from_waybill`. To see what was corrected on a waybill,
when and by whom, use `get_waybill_corrections`; it only reads. RS.GE
names who corrected only for corrections made through its API; for one
made on the RS.GE website `corrected_by` is empty, so say that RS.GE does
not record who, never guess a person.

One invoice for several waybills, or for a waybill's goods plus
services such as labour, a call-out or transport: use
`save_invoice_from_waybills` with the waybill numbers and the
`extra_lines`. Call it first with `validate_only` true: that saves
nothing and returns the lines, VAT and totals it would save, so show
them and get a yes before saving. It saves a draft only. Show the user
every line with its `source` (waybill or extra) and the total, then get
a yes before `send_invoice`. If a save is refused with
`waybill_already_invoiced` after a timeout, that names the invoice the
waybill is bound to, most likely the draft the lost call left;
`empty_draft_exists` names candidate drafts (another save may have left
them). Either way, read the invoice with `get_invoice`, show it, and pass
its id as `invoice_id` only once the user agrees it is this one: a save
with `invoice_id` replaces that invoice's lines and waybills. After a timeout
with no draft named, wait two minutes or check `list_invoices` before
saving again. To add a reference number to a line name, or to add or
remove a line on a saved invoice, use `update_invoice_lines` and show
the lines before and after. `get_invoice` with `line_sources` true says
which lines match the bound waybills' goods. Those sources are derived
by matching names (and, for a renamed line, quantity and amount), not
RS.GE's record, so say so.

## Name the RS.GE company

When `list_rs_companies` returns more than one company, pass its label
as `company` on every write: without it the write is refused before
RS.GE is contacted (reason `company_selection_required`). If it is not
clear which company the user means, ask before writing; a document id
alone does not select a company. Name the company in every draft you
show, and say which company the data came from when a read used the
default one.

## Find invoices and waybills with the list tools

For petroleum special invoices (სპეც-ანგარიშ-ფაქტურა), call
`list_seller_special_invoices` for issued records and
`list_buyer_special_invoices` for received records. Use dates, status
filters and pagination, then read the goods with `get_special_invoice`
and the returned `invoice_id`. Show company names and tax identifiers as
returned. Use the customs or source special invoice references RS.GE
supplies. On a received invoice, `buyer_actions` names what the buyer
can still do; issuing a special invoice still happens on eservices.rs.ge.

To find an e-invoice, call `list_invoices` with what you know: the
counterparty's TIN or name, the RS.GE number, dates or the amount. Then
read the row with `get_invoice` and its `invoice_id`, or pass the
printed series and number (for example `ეა-86 5655835`) to `get_invoice`
directly. For the number printed on a waybill (for example
`0793384115`), pass it to `get_waybill` as `number`, as a string. Find
other waybills with `list_seller_waybills` or `list_buyer_waybills`:
`comment` finds an order number or other text in the comment, and on
received waybills `seller_tin` is the supplier. For the original of a
return (type 5), call `find_return_original` with the return's id or
number. RS.GE keeps no link between them, so its candidates are
suggestions matched by goods: say so, and open the chosen one with
`get_waybill` before relying on it. Never step through ids
one by one: each call is a separate RS.GE request, so it takes many
minutes and loads RS.GE for every waybill.ge customer. After 20 lookups
in 10 minutes that find nothing, waybill.ge pauses that tool. If a
search finds nothing, widen the dates or ask the user.

Waybill list rows are short: the number, status, amount, the other
party and the fields that have a value. Read the goods, addresses and
dates of one waybill with `get_waybill`; pass `all_fields` only when you
need every field on many rows. On received waybills `confirmed` means
this company confirmed receipt on RS.GE, an optional step separate from
closing, so a closed waybill is often unconfirmed; `confirmed=false`
lists the ones not confirmed yet.

For a month's VAT (output, input and the difference), call
`vat_period_summary` with the month instead of adding up
`list_invoices` pages. It counts each invoice once and adds up
e-invoices only, so tell the user the VAT return can differ (imports,
reverse charge, invoices registered later).

## Never ask for passwords in chat

Never ask the user to type an RS.GE, Balance.ge, QuickShipper or 1C
password, service user password or API key, a Bank of Georgia Business
Online or TBC Bank API integration client ID or secret, or a ProCredit
Bank synchronization service user's name or password, into the
conversation, and do not repeat one if they paste it. Credentials belong
in the waybill.ge dashboard, under Business integrations:
https://waybill.ge/dashboard/integrations (RS.GE and Balance.ge
companies, QuickShipper accounts, 1C publications and bank connections
are connected and managed there; the banks' sections are
https://waybill.ge/dashboard/integrations#bog,
https://waybill.ge/dashboard/integrations#tbc and
https://waybill.ge/dashboard/integrations#procredit).

If a tool reports missing or wrong credentials, point the user to the
dashboard page instead.

## Person references in OpenAI's listed apps

The ChatGPT app from OpenAI's directory and the waybill.ge Codex plugin
connect to `https://waybill.ge/mcp/openai`, where the tools neither take
nor show a person's personal ID number: a person (a driver, a buyer or
supplier who is a person, an individual entrepreneur) appears as a person
reference, `pid_` followed by letters and digits. Claude, and any app
connected to `https://waybill.ge/mcp`, sees the TIN as the connected system
holds it, except where a tool masks it (bank transactions, below).

When an answer does show a person reference:

- Pass it back exactly as you got it wherever a tool asks for that
  person's TIN (`buyer_tin`, `driver_tin`, `list_invoices`' `counterparty_tin`
  and the rest), and use it as is to match the same person across answers. A
  company keeps its 9-digit TIN.
- Never ask the user to type a personal ID number there. A tool refuses
  one with `INVALID_FIELD` (reason `personal_id_not_accepted`, and
  `field` names the argument), and it refuses a reference in any other
  field.
- To name a driver, take the reference from `list_active_drivers` or an
  earlier waybill. Someone who is not yet in the company's records (a new
  driver or a new individual buyer) is entered at RS.GE first; say so
  rather than guessing.
- A reference refused as not valid (reason `person_reference_invalid`) is
  out of date: read the record again and use the reference it gives now.
- Looking a person up, or searching for one, by their ID is not offered
  there: `lookup_taxpayer_info`, `lookup_name_by_tin` and the
  `counterparty_tin` filter of `bank_list_transactions` take a company's
  9-digit code only, and refuse a person reference too (reason
  `person_lookup_not_offered`). Find a person's payments by
  `counterparty_name` or `text` and tell people apart by IBAN; for a
  person's taxpayer details, send the user to RS.GE. On
  `https://waybill.ge/mcp` these tools still take a personal ID number.

## Two people with the same name

When a name the user gives matches more than one person (two drivers, two
individual buyers, two bank counterparties), never pick one. Show them side
by side, each with its TIN or person reference and what tells them apart
(for a driver the vehicle and trips from `list_active_drivers`, for a bank
row the IBAN, otherwise the last waybill or invoice with that person), and
ask which one is meant.

## Balance.ge is read-only

The Balance.ge tools only read data. Nothing is ever posted to or
changed in the user's ledger, so no confirmation is needed for them.

If `balance_list_companies` marks a company `needs_repair: true`, do
not pass its label to any other Balance tool and do not run
`balance_check_credentials` on it. Tell the user to disconnect it and
connect the right company again under Business integrations.

Pass an item, warehouse, branch or price type the way the user named it:
a code (leading zeros optional), a barcode or the exact name works as a
filter. If the answer says `reason: ambiguous`, show the candidates and
ask which one is meant; never pick one yourself. `reason: lookup_loading`
means the company's item list is still loading after a restart: ask
again after `retry_after_seconds` instead of paging through every item.
An answer with `complete: false` came from Balance itself because our
list could not answer (exact names, articles, external codes and tax IDs
only): tell the user it may be partial and what its `note` says. With
`retry_after_seconds`, ask again after it for a code, a barcode or part
of a name; without it the list is too large to search, so follow the note.
An empty price list carries a `prices_note`; say what it says, and never
report a missing price as zero. Any Balance read that finds nothing is
`ok` with `count: 0` and a `note`: report what the note says, not a
failure.

`balance_fetch_general_ledger` reads up to a year one page at a time.
While `next_cursor` is not null, call again with the same arguments and
`cursor` set to it, and add amounts up only once `next_cursor` is null
(`complete: true`); a partial walk is not a total. A `CurrencyCode` of
null with `currency_unknown: true`, on a posting, a price or a balance,
means Balance's books do not name the currency: say it is unknown, never
guess it from the amounts. For one item's postings pass `item` (the
same codes, barcodes and names as the stock tools), and for one
analytic text, such as an expense line or a fixed asset as "Name,
Code", pass `analytic`: Balance cannot filter on either, so waybill.ge
reads the range and keeps the matches. When `shared_name_note` says
several items share the name, say the postings may belong to any of
them. Find a client or vendor by part of its name
with `name` on `balance_list_clients` or `balance_list_vendors`.

Find a product with `balance_find_item` (code, barcode or name) before
reading its stock or price. Balance sends no zero stock: when an item
shows none, read `balance_get_stocks` with the item and
`include_zero=true`, and always read and report its `zero_stock_note`.
A zero row carries the item's `last_movement`: give it as the latest
movement (date, kind, document number), not as proof of where the stock
went, and pass on every doubt the note raises (another item may share
the name, `movement_name_shared`; a reversal; any warehouse). When it is
null, the note says why: no movement in the dates in `movement_searched`,
or a read that failed (then the movement is unknown). When no zero row
comes back because the search settings could not be read, the stock is
unknown: say so, never "sold out". Never guess. Equipment a company capitalised (vehicles,
forklifts, fridges) leaves stock for the fixed-asset register, so when
an item shows no stock or the user asks where a machine is, search
`balance_list_fixed_assets` (a code, an inventory number or any part of
the name, which finds a serial number). An asset's `Warehouse` and
`ResponsiblePerson` are where it is now and who holds it. Balance's API
has no movement history, book value or serial field for an asset: say
so and point to Balance's own fixed-asset report rather than guess.
`ResidualValue` is the salvage value, not the book value, and `State`
can still say in operation after a sale or write-off. Its purchase,
depreciation and disposal postings come from
`balance_fetch_general_ledger` with `analytic` set to "Name, Code".

Read sales, sales returns, purchases and transfers between warehouses
with `balance_list_documents` (a date range, a page at a time) and one
in full with `balance_get_document`. A sale's `WaybillID` is the number
printed on its RS.GE waybill: open it with `get_waybill(number=...)`,
never as `waybill_id`. Purchases come as totals only: say Balance's API
has no purchase lines. Document numbers repeat across years, so on
`reason: ambiguous` narrow by date or use the uid.

For "when did X last move?", "who bought the last X?" or "where did it
go?", read `balance_item_movements` (the item by code, barcode, name or
uid, and a date range): its movements come newest first, a page at a
time, from the ledger. While `next_cursor` is not null the range is not
fully read: older days remain unread and may hold no movements, and a
short or empty page can still carry a cursor. Read until `next_cursor`
is null before saying the item did not move. A movement's
`from_warehouse`/`to_warehouse` are named only for sales, returns and
transfers: say the ledger names no warehouse for the rest rather than
guess. When `name_shared_with` or `name_note` is present, another item
may have the same name: rows with `item_verified: null` may belong to
either, so say so.

## Banks are read-only

The `bank_*` tools only read the balances and statements of the Bank of
Georgia, TBC Bank and ProCredit Bank accounts the user selected in the
dashboard.
They never move money: nothing is paid, transferred or signed at the
bank, and no tool can do it, so never offer to. No confirmation is needed
for them.

- The user never pastes bank credentials in chat. For Bank of Georgia,
  the client ID and secret of the Business Online API client go on the
  dashboard connect page (https://waybill.ge/dashboard/integrations#bog,
  then Bank of Georgia, Connect), which has the step-by-step guide. For
  TBC Bank, the client ID and secret of an API integration created in
  TBC Business Internet Banking go on
  https://waybill.ge/dashboard/integrations/tbc/connect, which suggests
  the accounts with recent movements that the key can read (Find my
  accounts; the list can be incomplete, and any other account is added
  by its IBAN on the connection's page);
  https://waybill.ge/dashboard/integrations/tbc/setup walks through
  creating that integration. For
  ProCredit Bank, the name and password of a synchronization service
  user go on https://waybill.ge/dashboard/integrations/procredit/connect;
  https://waybill.ge/dashboard/integrations/procredit/setup walks through
  adding that user in ProCredit Internet Bank. If the user pastes one
  anyway, do not repeat it; point them to that page.
- With more than one bank connected, pass `bank` (`bog`, `tbc` or
  `procredit`) or a `connection` label. Without either, the answer asks which bank,
  because each bank keeps its own default connection. If exactly one
  connection across those banks is marked default, that one is read
  instead. `bank_list_transactions` is the exception: without `bank`,
  `connection` and `account` it searches every bank, and an `account`
  (IBAN) alone is read at its own bank. Every answer's `bank` field says
  which bank was read (null when several were; each transaction then has
  its own `bank`), so name the bank to the user.
- Pass a label from `bank_list_connections` as `connection` when the
  user has more than one. `bank_list_transactions` reads one account, or
  every selected account of the connection it reads when `account` and
  `currency` are left out, so "payments from TIN X last month" is one
  call. Searching every bank reads each bank at the connection it reads
  by default; pass `connection` for another one. `accounts` lists what
  was read, and `errors` any account or bank that could not be (then
  `complete` is false; tell the user which). `bank_statement_summary`
  reads one account; when several match, `remediation.accounts` lists
  them to choose from. Each call covers at most 31 days. Transaction
  amounts are positive; `direction` gives the sign.
- Each transaction has `amount_gel`, the amount in lari (`fx_source`
  says whether it is the bank's own figure or the National Bank of
  Georgia's rate for the day), and a `type`: incoming_transfer,
  outgoing_transfer, fee, currency_conversion, card, cash, tax, salary,
  internal_transfer or other. Every transaction has the same fields at
  every bank; a value the bank does not give is null.
- A payment without `counterparty_tin` is not an unpaid bill: banks often
  leave out the payee's TIN on money out. `tin_coverage` counts the
  payments that lack it and `tin_note` says so; match those by
  `counterparty_iban` or name before calling anything unpaid.
- `is_own_company: true` marks money moved between the user's own
  accounts, including to its own account at another bank, and payments
  between two companies connected to the same waybill.ge account.
  A move is marked when the bank books it as an internal transfer, the
  other account is selected in one of the user's bank connections, or the
  other side's TIN is the user's company's. A move to an own account that
  is not selected, with no TIN, stays unmarked: match it by
  `counterparty_iban` or name before calling it income or spending.
  A move between one company's own accounts is not income or spending:
  `own_company="exclude"` leaves out every marked move, `only` keeps
  just them, and `totals` then gives money in and out per currency. When
  the user asks about one of several connected companies, a payment to
  or from the other one is real income or spending for it: check
  `counterparty_tin` before leaving it out. A statement summary's
  `credit_sum` and `debit_sum` include them. `payer_tin` is
  only a third party who paid on someone's behalf, never the user's own
  company.
- What a bank's statements can do is in `capabilities` from
  `bank_list_connections`. Bank of Georgia's statement summary does not
  cover today: when it returns `covered_to` and a `note`, use
  `bank_list_transactions` for the missing days. `truncated: true` means
  the list is incomplete; follow its `hint`.
- TBC Bank's statement summary gives the period's totals only: `daily`
  is null with a `daily_note`, so use `bank_list_transactions` for
  day-by-day entries. A TBC balance can carry `overdraft_limit`, which
  is why `available` may be higher than `current`. A TBC transaction
  with `status` (`awaiting_authorization` or
  `awaiting_second_authorization`) is not authorized at the bank yet:
  say so, and do not count it as settled. A `PERMISSION_DENIED` that
  says TBC Bank refused the Business Internet Banking user linked to the
  integration (blocked, deleted or not active) is fixed at the bank, on
  its API Integration page, not in waybill.ge.
- ProCredit Bank reads reach back 90 days only (`history_days`); an
  earlier window is refused as `OUT_OF_RANGE`, so say so instead of
  answering in part. ProCredit states no available balance: `available`
  is null with an `available_note`, never zero. An account with no
  entries in those 90 days has `current: null` with a `note`: say the
  bank states no balance, never 0.
- A currency or IBAN the company opened at the bank later is not read
  until the user adds it with Add accounts on the connection's dashboard
  page; Check again only re-checks the accounts already saved.
- `NEEDS_RECONNECT` (with `legacy_code` `REAUTH_REQUIRED` when the key
  was refused) means the bank no longer accepts the saved API key or
  service user (for ProCredit, often a changed password), or the
  connection was disconnected: the user updates it or connects again
  under Business integrations. A connection with
  `accounts: []` cannot be read until the user adds an account there.
  `NOT_HELD_CURRENCY` means no selected account holds that currency; the
  message lists the currencies that are held.
- In bank transactions a person's 11-digit personal ID number is masked
  to its last 3 digits (`********123`) in names, payment texts and TINs.
  Never try to recover or guess a personal ID, and never ask the user
  for one.
- Disconnecting deletes the key from waybill.ge but does not revoke it at
  the bank. To end it there, the user deletes the API client in Business
  Online (Bank of Georgia), deletes the integration on the API
  Integration page of TBC Business Internet Banking (TBC Bank), or
  blocks or deletes the service user in ProCredit Internet Bank, under
  My profile, Users (ProCredit Bank).

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
- `onesoft_get_document` takes a document `number` as well as a
  `ref_key`. When its answer lists `candidates` (two documents share the
  number), show them and ask which one is meant; never pick one yourself.
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

- The three `napr_*` tools look up companies and property, never
  people. They take a legal entity's 9-digit identification code (the
  same number as its RS.GE TIN), part of its registered name in Georgian
  script, a cadastral code or an application's registration number, and
  they refuse personal ID numbers. Do not offer to search for a person.
  Look up one counterparty at a time; they are not for bulk checks.
- `nbg_exchange_rates` takes a list of `currencies` (omit it for all), a
  `date` and a `language`, so one call covers several currencies for a
  day; call once per day for a series. An NBG `rate` is GEL per
  `quantity` units (AMD per 1000, JPY per 100), so use `rate_per_unit`
  for one unit. On a weekend or holiday `valid_from` shows the earlier
  set in force. These are official reference rates, not a bank's buy or
  sell rates; attribute them to the National Bank of Georgia.
- `SERVICE_UNAVAILABLE` from any of the four means the public source did
  not answer or is paused after a captcha; from a `napr_*` tool it can
  also mean the account made too many registry lookups in the last
  minute. Say so and offer to try again later (`retry_after_seconds`
  when given). Never report a status, "no liens" or a rate you did not
  receive.

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
- `RATE_LIMITED` with reason `lookup_misses` from `get_invoice`,
  `get_waybill`, `find_return_original` or `get_waybill_corrections`:
  too many ids found nothing. Stop trying ids; find the document with
  the list tool the message names (`list_invoices`,
  `list_seller_waybills` or `list_buyer_waybills`), or ask the user.
- Ask the user to reconnect waybill.ge only when the host reports an
  authentication failure (HTTP 401), never for the errors above.
