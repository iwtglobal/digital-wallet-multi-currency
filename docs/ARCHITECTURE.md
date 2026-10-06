# Architecture — Digital Wallet Multi-Currency

High-level reference for evaluating a **multi-currency digital wallet**, e-wallet FX conversion, and multi-currency balances. Educational only; vendor designs vary.

## Logical Components

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ App / API /      │────►│ Rate service     │────►│ Quote store     │
│ partner channel  │     │ (ref rate+spread)│     │ (expiring)      │
└──────────────────┘     └──────────────────┘     └────────┬────────┘
                                                           │ accept
                                                           ▼
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ Treasury view    │◄────│ Currency pockets │◄────│ Posting engine  │
│ (net positions)  │     │ (per wallet)     │     │ (atomic legs)   │
└──────────────────┘     └──────────────────┘     └─────────────────┘
```

## Suggested Quote Record Fields

| Field | Meaning |
|-------|---------|
| `quote_id` | Unique quote reference, used idempotently on accept |
| `wallet_id` | Wallet the quote was issued for |
| `source_currency` / `target_currency` | ISO 4217 codes |
| `reference_rate` | Rate sourced from the provider, with source and timestamp |
| `customer_rate` | Rate offered after spread |
| `fee_minor` | Explicit fee in minor units, if any |
| `source_amount_minor` / `target_amount_minor` | Exact amounts shown to the customer |
| `expires_at` | Time after which the quote cannot be accepted |
| `status` | `issued` / `accepted` / `expired` / `rejected` |

## Suggested Ledger Line Fields

| Field | Meaning |
|-------|---------|
| `entry_id` | Journal entry grouping both legs of a conversion |
| `account_id` | Customer pocket, fee account, or FX position account |
| `currency` | ISO 4217 code; every line carries one |
| `amount_minor` | Signed integer in the currency's minor unit |
| `quote_id` | Link back to the accepted quote |

## Audit Events Worth Keeping

- Rate source updates, including stale or failed fetches  
- Every quote issued and its outcome  
- Conversion postings with both legs and fee lines  
- Manual rate overrides, with approver and reason  
- Pocket open, freeze, and close events  

## Failure Modes to Design For

- Only one leg of a conversion posted after a timeout  
- Quote accepted twice through a retried request  
- Quoting on a stale rate during a rate-feed outage  
- Assuming two decimal places for every currency  
- Treasury unaware of a growing net position in one currency  

## Related Reading

Live product reading: [evdsystem.com](https://evdsystem.com/), [About MoboGage](https://evdsystem.com/about-mobogage/).
