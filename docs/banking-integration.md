# Bank connection for the Ledger & Tax panel

Status: not implemented yet — planning notes only, captured 2026-08-08.

## Approach

Connect via a regulated UK Open Banking aggregator (TrueLayer, GoCardless/Nordigen,
or Yapily) rather than building bank connectivity directly. The aggregator holds the
FCA authorization and handles the actual secure link to the bank: the user is
redirected to their own bank's login page, authenticates there, and only an OAuth
token comes back to us. We never see or store the bank password.

This means we are not building bank-grade security infrastructure from scratch —
that part is already built and regulated by the aggregator. Our work is mostly
integration: calling their API, handling the OAuth redirect/callback, and taking
proper care of the two things that do land in our system.

## Vault / secrets handling

What we're responsible for once the aggregator hands us a token:

- API keys and OAuth access/refresh tokens stored encrypted at rest, not in plain
  config or source control
- Secrets loaded from environment variables / a secrets manager, never hardcoded
- No tokens or credentials written to logs
- Scoped, short-lived tokens refreshed through the aggregator's flow rather than
  stored indefinitely

## Transaction handling

What we're responsible for once transaction data comes back from the aggregator:

- Transaction data transmitted over HTTPS only
- Stored data access-restricted to what the app needs to display (monthly
  takings/costs for the ledger panel), not broader account data
- Basic UK GDPR handling since this is a real person's financial data
- No transaction content in logs or error reports

## Not yet decided

- Which aggregator (TrueLayer vs GoCardless/Nordigen vs Yapily)
- Where the backend lives (this repo is currently a static single-page demo with
  no server — a backend would need to be scaffolded separately)
- Sandbox/test account first, before anything touches a real bank connection
