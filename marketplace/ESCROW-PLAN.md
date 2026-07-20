# Escrow Plan — Stripe-Compliant Flow of Funds

> How money moves through QuietLot so that buyers are protected, sellers and brokers get paid
> automatically, and the platform stays inside Stripe's rules and outside money-transmitter /
> escrow-agent licensing. ⚠️ This whole document is the thing counsel reviews (LAWYER-LETTER
> items 3–4) — nothing here is legal advice, and no real payment runs until counsel signs off.

## 1. Principles

1. **We are a payments facilitator, not a bank.** Funds are held only between two defined
   events — payment and delivery confirmation — never "on deposit."
2. **Stripe is the regulated party.** All funds live in Stripe's system (platform balance /
   connected accounts); card and bank data never touch our servers.
3. **Every hold has a clock.** Nothing is held indefinitely; every state has an automatic exit.
4. **The ledger is ours.** Stripe moves the money; our append-only ledger explains every cent
   (it already exists in the prototype and the data model).

## 2. The flow, hop by hop

Actors: Buyer B, Broker K (counterparty of record), Seller S. Direct orders are the same minus
the middle hop.

```
1. ORDER      B pays total via Stripe (card or ACH) → PaymentIntent on the PLATFORM account,
              transfer_group = order chain id. No instant payout to anyone.
2. ROUTED     K's upstream PO to S is recorded; no money moves yet beyond B's charge.
3. DELIVERED  B confirms delivery (or auto-confirm N business days after verified delivery
              — default 3 — if no dispute is opened).
4. RELEASE    Two transfers fire from the platform balance, same transfer_group:
              → S's connected account: seller cost basis (the upstream PO amount)
              → K's connected account: K's margin (retail total − cost − platform fee)
              Platform fee stays on the platform account (application fee).
5. PAYOUT     Standard Stripe payout schedule from each connected account to its bank.
```

- **Pattern:** Separate Charges & Transfers (charge on platform, later transfers to connected
  accounts) — Stripe's supported pattern for exactly this "release on event" marketplace flow.
- **ACH first for large orders:** wholesale-sized totals default to ACH debit (cheaper,
  chargeback-light); cards allowed under a configurable ceiling. ACH settlement lag (~4 business
  days) is communicated in the order timeline before the seller ships.
- **Connected accounts:** Stripe Express for S and K (Stripe handles KYB, we keep the UX).
  Payout-capable roles require MFA (SECURITY.md).

## 3. Hold windows & the clock

| State | Held where | Exits how | Max duration |
|---|---|---|---|
| Paid, awaiting fulfillment | Platform balance | Ship + deliver, or cancel → full refund | Cancel auto-fires if not shipped in X days (per listing terms, default 10 business days) |
| Delivered, awaiting confirmation | Platform balance | Buyer confirms, or auto-confirm after N=3 business days | N days |
| Disputed | Platform balance (frozen for this order) | Dispute ladder outcome → release, partial refund, or refund | Target 14 days; hard cap 30 |

The caps matter legally: a defined, short, event-driven hold is payments facilitation; an
open-ended hold starts to look like custody/escrow. ⚠️ Counsel confirms the caps.

## 4. Disputes, refunds & chargebacks

- **Dispute opened before release:** funds for that order freeze in the platform balance.
  Resolution follows MARKET-TERMS §8 (acknowledgment record + manifest + audit log). Outcomes:
  full release, negotiated partial (split adjusted), or refund to B.
- **Multi-hop refunds:** each hop unwinds independently — B is made whole from the downstream
  hop first; K recovers from S on the upstream hop per their own dispute. The platform never
  advances its own money; it re-routes what's held. Where the upstream hop already released
  (e.g., broker fulfilled from held stock), K's future payouts carry a negative balance offset.
- **Card chargebacks:** evidence pack is auto-assembled (acknowledgment screen record, manifest,
  delivery confirmation, message log). Our B2B model + acknowledgment flow is built to win
  these; ratio target <0.9% (SETUP.md §2).
- **Risk reserve:** platform holds a small rolling reserve on new high-volume accounts
  (e.g., 5% for 30 days) until history accrues — standard marketplace practice, disclosed in
  terms. ⚠️ Confirm disclosure language.

## 5. What we say vs. what we don't

- Until counsel approves the word "escrow," all UI/marketing copy uses **"funds held until
  delivery is confirmed"** / **"payment protection."** (LAWYER-LETTER item 4. The prototype
  demo says "escrow" today — it's a demo; sweep the copy before launch either way.)
- We never describe the platform balance as an "account," "wallet balance," or anything
  deposit-like for participants; participants see *orders and payouts*, not stored value.
  (The demo's "Wallet" view gets renamed to **"Payments"** in the MVP for exactly this reason.)

## 6. Graduation path

| Trigger | Move |
|---|---|
| Lot sizes routinely > card-comfortable (wires, $50k+) | Add wire-in flow; consider Stripe Treasury or a licensed partner for holding |
| Hold windows need to lengthen (freight, inspection periods) | Licensed escrow partner (Escrow.com API / Trustap / Balance) for those order types only |
| Net-terms demand (net-30/60) | Financing partner or B2B BNPL (Balance, Slope, Resolve) — never extend credit off our own balance sheet |

## 7. Build checklist (MVP)

- [ ] Stripe platform account, Connect enabled, business profile = B2B marketplace (SETUP.md §2)
- [ ] Express onboarding for sellers/brokers; block listing activation until `charges_enabled`
- [ ] PaymentIntent + transfer_group per order chain; ACH + card
- [ ] Delivery-confirmation webhook → transfer release job (idempotent, ledger-first)
- [ ] Auto-confirm timer + dispute-freeze flag
- [ ] Ledger table mirrors every Stripe object (charge, transfer, refund, payout) 1:1
- [ ] Reconciliation job: ledger vs. Stripe balance daily; alert on drift
- [ ] Test-mode end-to-end: order → route → deliver → split release → refund path → chargeback sim
