# waybill.ge for Claude Code

The financial connector for businesses in Georgia. This plugin connects
Claude Code to [waybill.ge](https://waybill.ge), so you can issue and
track RS.GE e-waybills and e-invoices and ask read-only questions about
your Balance.ge accounting in plain language.

## What it does

- **RS.GE e-waybills:** look up counterparties by TIN, draft waybills,
  activate them after you confirm, list issued and received waybills
  with totals, close delivered waybills, confirm or reject incoming
  ones, and create invoices from waybills.
- **RS.GE e-invoices:** issue an advance invoice for a prepayment or a
  regular invoice without a waybill, send it to the buyer after you
  confirm, and complete an advance by offsetting its VAT against the
  final invoice.
- **Balance.ge accounting (read-only):** clients and vendors, items,
  stock levels, prices, who owes you and whom you owe, the general
  ledger and monthly cash flow. Nothing is ever posted to your ledger.

The plugin adds one remote MCP server, `waybill`, at
`https://waybill.ge/mcp`, and a small `waybill-safety` skill. The skill
has Claude show you a waybill or e-invoice draft and wait for an
explicit yes before any binding RS.GE action (activate, close, confirm,
create invoice, send an e-invoice, offset an advance), and never ask for
your RS.GE or Balance.ge passwords in chat.

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
   if you use it, your Balance.ge companies. These passwords are stored
   encrypted on waybill.ge and never reach Claude.

New accounts start with a free trial. Plans and limits:
[waybill.ge](https://waybill.ge/#pricing).

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

## Links

- Documentation: <https://waybill.ge/developers>
- Privacy policy: <https://waybill.ge/privacy>
- Support: [hello@waybill.ge](mailto:hello@waybill.ge)

## License

[MIT](LICENSE).

waybill.ge is not affiliated with rs.ge or Balance.ge.
