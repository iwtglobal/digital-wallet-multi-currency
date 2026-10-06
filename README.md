# Digital Wallet Multi-Currency

An educational guide to the **multi-currency digital wallet**: holding balances in more than one currency, converting between them (e-wallet FX conversion), and keeping the ledger correct while rates move. Written for fintechs, telecom operators, and payment teams evaluating platforms in the [EVD System](https://evdsystem.com/) and [MoboGage](https://evdsystem.com/about-mobogage/) family.

---

## What Is a Multi-Currency Digital Wallet?

A **multi-currency digital wallet** is an e-wallet in which one customer account can hold several balances, each in its own currency, instead of a single balance in one currency. A customer might keep money in their home currency, receive a payment in another, and convert between the two inside the wallet.

The important design idea is that each currency is a **separate sub-balance (a "pocket")** with its own ledger entries. The platform never adds amounts in different currencies together. Converting money is not an edit to a balance; it is a pair of ledger postings, a debit in one currency and a credit in another, linked by a quoted exchange rate.

### Where multi-currency wallets show up

- **Cross-border remittance** — receive in the sender's currency, cash out or spend in the local one  
- **Travel and expatriate wallets** — hold a second currency for spending abroad  
- **Merchant and business wallets** — collect in several currencies and settle in one  
- **Regional platforms** — one wallet product serving several neighbouring markets  

---

## Architecture Overview: A Conversion Inside the Wallet

```
Customer requests conversion (app / API)
        │
        ▼
Rate service returns a quote (rate, spread, fee, expiry)
        │
        ▼
Customer accepts quote before expiry
        │
        ▼
Checks: balance, limits, currency allowed for this KYC tier
        │
        ├──► Pass ──► Atomic double posting:
        │               debit  source pocket   (currency A)
        │               credit target pocket   (currency B)
        │               record FX gain / fee lines
        │
        └──► Fail ──► Decline with reason code (quote expired, limit, balance)
```

### Core layers

| Layer | Responsibility |
|-------|----------------|
| **Currency pockets** | One sub-ledger per currency per wallet, with its own balance |
| **Rate service** | Pulls reference rates, applies spread, issues time-limited quotes |
| **Quote store** | Keeps every quote shown, accepted, or expired, for audit |
| **Posting engine** | Writes linked debit/credit pairs in a single atomic transaction |
| **Treasury position** | Platform's net exposure per currency after customer conversions |
| **Reporting** | Balances, conversions, fees, and positions per currency |

See [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) for data fields and failure modes.

---

## How Multi-Currency Balances Work, Step by Step

### 1. Open a pocket per currency

A wallet starts with its home currency. Additional pockets are opened on demand (or automatically on first receipt), subject to which currencies the operator and the customer's KYC tier allow.

### 2. Quote before converting

The customer sees the rate, any fee, and the exact amount they will receive *before* confirming. The quote has a short expiry so the platform is not exposed to rate movement on stale prices.

### 3. Post both legs atomically

The debit in currency A and the credit in currency B must succeed or fail together. A conversion where only one leg posts creates or destroys money.

### 4. Store amounts in minor units with the currency code

Every amount is stored as an integer in minor units (for example, cents) plus an ISO 4217 currency code. Different currencies have different numbers of decimal places, so the platform should read the exponent from the currency definition rather than assume two.

### 5. Handle incoming payments in a foreign currency

When money arrives in a currency the wallet does not hold, the policy must be explicit: open a new pocket, auto-convert to the home currency at a disclosed rate, or reject.

### 6. Reconcile per currency

Customer pockets, settlement accounts, and the treasury position are reconciled separately for each currency. A total "in home-currency terms" is a reporting view, not a ledger balance.

---

## Key Concepts

- **Pocket / sub-balance** — the per-currency balance inside one wallet  
- **Reference rate vs customer rate** — the market rate the platform sources, and the rate offered after spread  
- **Spread and fee** — two different ways of charging for conversion; disclose both clearly  
- **Quote expiry** — the window in which a shown rate is guaranteed  
- **Rounding rule** — which side absorbs rounding differences, defined once and applied consistently  
- **Currency position** — the platform's net holding per currency, which treasury must fund or hedge  

---

## Implementation Considerations

- **Never mix currencies in one balance column** — keep currency on every ledger line  
- **Limits per currency or in a base currency** — decide and document how KYC limits apply across pockets  
- **Refunds of converted transactions** — define whether a refund returns the original currency amount or re-converts at a new rate  
- **Rate source outages** — stop issuing quotes rather than quoting on stale rates  
- **Regulatory scope** — holding and converting foreign currency may require specific licensing; this varies by country  
- **Display vs value** — showing a balance converted for display must not change the stored value  

Platforms such as EVD System / MoboGage provide digital wallets alongside mobile money, eTopup, and voucher distribution, so the same ledger discipline applies when a wallet pays for airtime, gift cards, or bills.

---

## FAQ

**What is a multi-currency digital wallet?**  
An e-wallet where one account holds separate balances in several currencies and can convert between them through linked ledger postings.

**How does FX conversion work in an e-wallet?**  
The platform issues a time-limited quote (rate plus any fee). When the customer accepts, it debits the source currency pocket and credits the target pocket in one atomic transaction.

**Why store each currency in its own pocket?**  
Because amounts in different currencies cannot be added. Separate pockets keep balances, limits, and reconciliation correct per currency.

**What happens if the exchange rate changes during checkout?**  
If the quote has not expired, the quoted rate is honoured. If it has expired, the customer must accept a fresh quote.

**How does this relate to MoboGage / EVD System?**  
MoboGage offers digital wallet and mobile money platforms within the EVD System family. See the [EVD System home](https://evdsystem.com/) and [About MoboGage](https://evdsystem.com/about-mobogage/) when evaluating product fit.

---

## Further Reading / Related Industry Resources

- [EVD System home](https://evdsystem.com/) — platform overview for digital wallets, eTopup, and voucher distribution  
- [About MoboGage](https://evdsystem.com/about-mobogage/) — company and product family context  
- Related repo: [prepaid-ewallet-ledger](https://github.com/iwtglobal/prepaid-ewallet-ledger) — ledger fundamentals for prepaid e-wallets  

---

## License

Documentation in this repository is provided under the MIT License. See [LICENSE](./LICENSE).

*Educational material only. Not financial, legal, or regulatory advice; foreign currency rules vary by country. MoboGage and EVD System refer to offerings on evdsystem.com.*
