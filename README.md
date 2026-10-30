# waybill.ge for Claude Code

The financial connector for businesses in Georgia. This plugin connects
Claude Code to [waybill.ge](https://waybill.ge), so you can issue and
track RS.GE e-waybills and e-invoices, ask read-only questions about
your Balance.ge or 1C (OneSoft) accounting, your QuickShipper deliveries
and your Bank of Georgia accounts, and check companies in the public
registry and the official NBG exchange rates, in plain language.

## What it does

- **RS.GE e-waybills:** look up counterparties by TIN, draft waybills,
  activate them after you confirm, find issued and received waybills by
  buyer or supplier, date, amount or text in the comment (such as an
  order number), with totals, or read one by the number printed on it.
  Close delivered waybills, confirm or reject incoming ones, and create
  invoices from waybills. When a transporter company carries the goods,
  draft a carrier waybill and send it to the transporter after you
  confirm; the transporter adds its own driver and vehicle. For a
  return, suggest the waybill it sends goods back from, matched by goods
  (RS.GE keeps no link between them).
- **RS.GE e-invoices:** find invoices you issued or received by
  counterparty, date, RS.GE number or amount, issue an advance invoice
  for a prepayment or a regular invoice without a waybill, send it to
  the buyer after you confirm, and complete an advance by offsetting its
  VAT against the final invoice.
- **Balance.ge accounting (read-only):** clients and vendors, items,
  stock levels, prices, who owes you and whom you owe, the general
  ledger and monthly cash flow. Nothing is ever posted to your ledger.
- **QuickShipper deliveries (read-only):** live delivery quotes for a
  route, your recent orders by date or status, and one order's status
  history and tracking link. Nothing is booked, changed or cancelled.
- **1C accounting (read-only):** your own 1C publication (for example one
  hosted by OneSoft): items, customers and suppliers, a counterparty's
  contracts, stock, prices, receivables and payables, cash, exchange
  rates, one document's lines, account turnovers and the trial balance.
  Nothing is created, changed or posted in 1C.
- **Bank of Georgia accounts (read-only):** current balances,
  transactions filtered by direction, counterparty, amount or payment
  purpose, and period totals with daily balances, for the accounts you
  choose. waybill.ge cannot make payments or transfers, or sign anything
  at the bank.
- **Public registry and NBG rates (read-only):** check a Georgian
  company in the National Agency of Public Registry (NAPR): its status
  and legal form, the liens recorded against it, and the registration
  applications for a property by cadastral code. Get the National Bank
  of Georgia's official exchange rates for any day. These read public
  sources and need no connection. They look up companies and property,
  not people.

The plugin adds one remote MCP server, `waybill`, at
`https://waybill.ge/mcp`, and a small `waybill-safety` skill. The skill
has Claude show you a waybill or e-invoice draft and wait for an
explicit yes before any binding RS.GE action (activate, close, send to a
transporter, confirm, create invoice, send an e-invoice, offset an
advance), treat QuickShipper
quotes as prices rather than bookings, keep slow 1C trial balance windows
narrow, treat the Bank of Georgia tools as read-only, never search the
public registry for people or report a guess when a public source is
down, and never ask for your RS.GE, Balance.ge, QuickShipper or 1C
passwords or your Bank of Georgia API key in chat.

## Install

In Claude Code:

```
/plugin marketplace add sepivip/waybill-claude-plugin
/plugin install waybill@waybill-ge
```

Or from a terminal:

```
claude plugin marketplace add sepivip/waybill-claude-plugin
claude plugin install waybill@waybill-ge
```

## Sign in

Sign-in is OAuth. The plugin contains no API keys, tokens or passwords.

1. After installing, run `/mcp` in Claude Code, select `waybill` and
   choose **Authenticate**. You can also run `claude mcp login plugin:waybill:waybill`
   from a terminal.
2. Your browser opens waybill.ge. Sign in with your email (a one-time
   code) and approve access.
3. In the waybill.ge dashboard, connect your RS.GE service user and,
   if you use them, your Balance.ge companies, QuickShipper accounts,
   1C publications (with a dedicated read-only 1C user; the
   [1C setup guide](https://waybill.ge/dashboard/integrations/onesoft/setup)
   lists the one-time steps inside 1C) and Bank of Georgia accounts.
   For the bank, you create a Business Online API client (Client
   Credentials Flow) for waybill.ge and choose the accounts Claude may
   read; the Bank of Georgia section of
   [Business integrations](https://waybill.ge/dashboard/integrations#bog)
   opens the connect page with the step-by-step guide. These credentials
   are stored encrypted on waybill.ge (for QuickShipper only the login
   token QuickShipper issues) and never reach Claude. Disconnecting a
   bank connection deletes its API key from waybill.ge.

New accounts start on the Free plan: 300 actions a month, every tool,
no end date. Plans and limits:
[waybill.ge](https://waybill.ge/#pricing). How many Bank of Georgia
connections you can add depends on your plan.

When waybill.ge's tools change, the server asks Claude Code to fetch the
tool list again. If the tools still look outdated after an update (an old
parameter default or description), run `/mcp`, select `waybill` and
reconnect, or start a new session.

## Example prompts

1. **Check RS.GE credentials**

   > Check that my RS.GE connection still works.

2. **Taxpayer lookup, then draft a waybill Tbilisi to Batumi**

   > Look up TIN 111111111. If they are an active VAT payer, draft a
   > general transport (type 2) waybill to them from our warehouse at
   > 12 Aghmashenebeli Ave, Tbilisi to their store at 5 Rustaveli St,
   > Batumi: 10 pcs of mineral water at 12.50 each, by car AA-000-DEMO,
   > driver personal ID 01001000001.

   Claude saves a draft, shows it to you and waits for your yes before
   activating it.

3. **Seller waybills for the last 7 days with totals**

   > Show the waybills we issued in the last 7 days, with the total
   > amount and a breakdown by status.

4. **Balance client balances and cash flow this month**

   > In Balance, who owes us money right now, and what is our cash flow
   > so far this month?

5. **Advance invoice for a prepayment**

   > Issue an advance invoice to TIN 111111111 for September: 1180 GEL
   > including VAT, "advance for consulting services".

   Claude saves the invoice as a draft, shows it to you and sends it to
   the buyer only after your yes.

6. **QuickShipper orders this month**

   > How many QuickShipper orders did we send this month, and how many
   > were cancelled?

   Claude reads the counts from QuickShipper; nothing is booked or changed.

7. **A customer's contracts and debt in 1C**

   > In 1C, which contracts does our customer Vake Market have, and how
   > much do they owe us on each?

   Claude finds the customer, lists its contracts and reads the
   receivable per contract. Nothing is posted in 1C.

8. **Bank of Georgia balances and last week's money in and out**

   > What is the balance on our Bank of Georgia accounts, and how much
   > came in and went out of the GEL account last week?

   Claude reads the balances and the GEL account's statement summary.
   Nothing is paid, transferred or signed at the bank.

9. **Check a new customer in the public registry, and today's USD rate**

   > Before I invoice a new customer, check in the public registry that
   > their company is active and has no liens, and give me today's
   > official NBG rate for USD.

   Claude looks the company up by its 9-digit identification code (the
   same number as its TIN), checks its liens and reads the National Bank
   of Georgia's rate. No connection is needed for these lookups.

10. **Find an e-invoice without its number**

    > Find the e-invoice we received at the end of August for 210.70 GEL,
    > and show its lines.

    Claude searches the received invoices in one call and reads the
    match. Nothing is changed in RS.GE.

## Links

- Documentation: <https://waybill.ge/developers>
- Privacy policy: <https://waybill.ge/privacy>
- Support: [hello@waybill.ge](mailto:hello@waybill.ge)

## License

[MIT](LICENSE).

waybill.ge is not affiliated with rs.ge, Balance.ge, QuickShipper, OneSoft,
1C, Bank of Georgia, the National Agency of Public Registry or the National
Bank of Georgia.
