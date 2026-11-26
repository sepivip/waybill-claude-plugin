---
name: waybill-setup
description: Help users sign in to waybill.ge, connect RS.GE, Balance.ge, QuickShipper, a 1C (OneSoft) publication, Bank of Georgia, TBC Bank or ProCredit Bank accounts, and resolve missing credentials or company connections when using the waybill.ge connector or plugin. The NAPR registry and NBG rate lookups need no connection.
---

# Set up waybill.ge

waybill.ge connects a business's RS.GE waybills and e-invoices, its
read-only Balance.ge accounting, its read-only QuickShipper deliveries,
its read-only 1C (OneSoft) accounting and its read-only Bank of Georgia,
TBC Bank and ProCredit Bank accounts. Each system needs its own connection; having RS.GE credentials
does not establish a Balance.ge, QuickShipper, 1C or bank connection. The
public registry (NAPR) and National Bank of Georgia rate tools read public
sources and need no connection, only the waybill.ge sign-in.

## Sign in and connect a system

1. Use the host's authentication flow for the waybill MCP connection.
   The browser opens waybill.ge for sign-in and OAuth consent.
2. Send users who need to connect a financial system to
   [Business integrations](https://waybill.ge/dashboard/integrations).
   Enter RS.GE service-user, Balance.ge, QuickShipper or 1C integration-user
   credentials there (for QuickShipper, waybill.ge keeps only the login
   token QuickShipper issues, encrypted). For Bank of Georgia, the user
   creates a Business Online API client (Client Credentials Flow) for
   waybill.ge and enters its client ID and secret, with the accounts AI
   apps may read, under Bank of Georgia, Connect. That page has the
   step-by-step guide; point to it rather than repeating the steps. For
   TBC Bank, a user who can manage API integrations in TBC Business
   Internet Banking creates an integration for waybill.ge that may only
   view transaction history and account statements, and the user enters
   its client ID and secret on the
   [TBC connect page](https://waybill.ge/dashboard/integrations/tbc/connect).
   Find my accounts on that page suggests and checks the accounts with
   movements in the 30 days up to yesterday that the key can read. The
   list can be incomplete: when nothing was found, one IBAN can be typed
   on the connect page instead; otherwise the user connects first and adds
   the other accounts by IBAN on the connection's page. The [TBC setup guide](https://waybill.ge/dashboard/integrations/tbc/setup)
   walks through it; point to the guide rather than repeating the steps. For
   ProCredit Bank, a ProCredit Internet Bank administrator adds a
   synchronization service user with view-only rights to the accounts,
   and the user enters its name and password on the
   [ProCredit connect page](https://waybill.ge/dashboard/integrations/procredit/connect).
   The [ProCredit setup guide](https://waybill.ge/dashboard/integrations/procredit/setup)
   walks through it; point to the guide rather than repeating the steps.
   Credentials are stored encrypted, and disconnecting deletes them from
   waybill.ge.
3. [Connector setup](https://waybill.ge/dashboard/connector) provides
   connection instructions. Claude, Claude Code and Cursor connect to
   `https://waybill.ge/mcp`, with or without the waybill.ge plugin.
4. Once authenticated, use `list_rs_companies` for RS.GE,
   `balance_list_companies` for Balance.ge,
   `quickshipper_list_connections` for QuickShipper,
   `onesoft_list_connections` for 1C or `bank_list_connections` for
   the banks as relevant to the request.
   `whoami` reports the selected RS.GE company. Use the returned labels,
   and carry the user's selected company into subsequent calls.

Never request, repeat or pass financial-system passwords, API keys or
OAuth tokens in chat. waybill.ge never needs them in chat.

## Connection problems

- Missing or rejected financial credentials: direct the user to Business
  integrations to connect or repair that system. Do not try guessed credentials.
- `needs_repair: true` on a Balance company: do not use it or call
  `balance_check_credentials` on it. Direct the user to disconnect and
  reconnect the correct company in Business integrations.
- `REAUTH_REQUIRED` on a QuickShipper or 1C connection: direct the user to
  update its login in Business integrations.
- `NOT_PUBLISHED` or `CONTENT_EMPTY` from `onesoft_check_connection`: the
  1C publication needs a one-time step by its 1C administrator, described in
  the [1C setup guide](https://waybill.ge/dashboard/integrations/onesoft/setup).
- `NEEDS_RECONNECT` on a bank connection (sent with `legacy_code`
  `REAUTH_REQUIRED` when the key was refused): the bank no longer accepts
  the saved API key or service user (for ProCredit, often a changed
  password), or the connection was disconnected. Direct the user to update
  it or connect again under
  [Business integrations](https://waybill.ge/dashboard/integrations#bog)
  (for TBC, [its section](https://waybill.ge/dashboard/integrations#tbc);
  for ProCredit, [its section](https://waybill.ge/dashboard/integrations#procredit)).
- A bank connection with `accounts: []`: the user adds an account
  to it in the dashboard before any bank tool can read it. A currency or
  IBAN opened at the bank later is read only after the user adds it with
  Add accounts on that connection's page; Check again only re-checks the
  saved accounts.
- `PERMISSION_DENIED` from a bank tool: bank access is not available to the
  account, or the bank refused the API key for an account. For Bank of
  Georgia, the user checks the Business Online user's API rights, as the
  connect page guide describes. For TBC Bank, the user checks that the
  integration may view transaction history and statements, and that the
  Business Internet Banking user linked to it is active and sees the
  account.
- Disconnecting a bank connection deletes its credentials from waybill.ge
  but does not revoke them at the bank; to end it there, the user deletes
  the API client in Business Online (Bank of Georgia), deletes the
  integration on the API Integration page of TBC Business Internet
  Banking (TBC Bank), or blocks or deletes the service user in ProCredit
  Internet Bank, under My profile, Users (ProCredit Bank).
- An expired MCP session: use the host's reconnect/authenticate flow. Ask
  the user to reconnect only when the host reports an authentication failure
  (HTTP 401), never for the errors below.
- `SERVICE_UNAVAILABLE` with `retryable: true` (also as the body of an HTTP 503): waybill.ge
  could not reach its own database for a moment and nothing was sent to
  RS.GE or any connected service. Retry once after a few seconds
  (`retry_after_seconds` when given); do not ask the user to reconnect.
  Any other `SERVICE_UNAVAILABLE`, or an HTTP 503 without that body, after
  a write (activate, close, confirm, send, correct, save) may come after
  RS.GE got the request: check the document's current state before
  repeating it, as for `INTERNAL_ERROR`.
- `SERVICE_UNAVAILABLE` from a `napr_` or `nbg_` tool: the public registry
  or the National Bank of Georgia did not answer, or is paused after a
  captcha. From a `napr_` tool it can also mean the account made too many
  registry lookups in the last minute. Nothing needs reconnecting. Say the
  source is temporarily unavailable and try again later
  (`retry_after_seconds` when given); never present a guessed status, lien
  list or rate.
- `INTERNAL_ERROR`: an unexpected error on waybill.ge. Give the user its
  `request_id` for support, and before repeating a write (activate, close,
  confirm, send), check the document's current state first.
- An unavailable company: list accessible companies and explain the result;
  do not substitute another company's data without the user's instruction.
- A quota or entitlement error: report the returned limit and remaining
  allowance when available. Do not promise access beyond the account's allowance.
  `QUOTA_EXCEEDED` means the actions for the current period are used up. On
  the Free plan the period is the calendar month and it resets on the 1st,
  Tbilisi time (`resets_on`); on a paid plan it is the billing period, which
  ends at `period_end`. Use the date the error returns; never assume the 1st
  for a paid account. The error carries the message in English and Georgian
  (`message_ka`); show the user's language. `whoami`, `check_credentials`,
  `get_quota_status` and `get_egress_ip` never count and still work then.
- `RATE_LIMITED` from `check_credentials` or `get_egress_ip`: wait
  `retry_after_seconds` before trying again. Do not retry in a loop.
- `RATE_LIMITED` with reason `lookup_misses` from `get_invoice`,
  `get_waybill`, `find_return_original` or `get_waybill_corrections`:
  too many ids found nothing. Stop trying ids; find the
  document with the list tool the message names (`list_invoices`,
  `list_seller_waybills` or `list_buyer_waybills`), or ask the user.

Balance.ge business data, QuickShipper deliveries, 1C accounting and bank
accounts stay read-only. Credential checks may update
connection-verification metadata on waybill.ge, but do not post to the
ledger or act at the bank.
For help, use [Support](https://waybill.ge/contact).
