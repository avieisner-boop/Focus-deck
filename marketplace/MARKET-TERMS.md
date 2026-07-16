# Market Terms — Working Draft

> **Status: founder's working draft, NOT a legal document.** This is the plain-English rulebook
> for how the marketplace works and what every participant agrees to. It exists so that (a) the
> product gets built to enforce these rules mechanically, and (b) an attorney can convert it into
> real Terms of Service / participation agreements without guessing at intent. Have a lawyer
> review before any real transaction happens on the platform.
>
> Design goals these terms serve, in order:
> 1. **Protect the source.** A broker's supplier relationships and a seller's channel control are
>    the inventory of this marketplace. The terms make taking them illegitimate, and the product
>    makes taking them hard.
> 2. **Everyone enters willingly and informed.** No party is ever surprised by who they're
>    dealing with (masked, and they knew it), what they bought (grade/manifest acknowledged), or
>    what happens when something goes wrong (dispute rules shown before payment).
> 3. **Someone identifiable is always responsible.** Masked never means unaccountable — the
>    counterparty of record owns every order, and the platform holds verified identity behind
>    every alias.

---

## 1. Definitions

- **Platform** — the marketplace operator (working name TradeMesh; entity TBD).
- **Participant** — any business with an account: Seller, Broker, and/or Buyer (roles can stack).
- **Storefront Alias** — the masked trade identity a Participant transacts under. One legal
  entity may hold multiple aliases only with Platform approval (anti-abuse).
- **Channel** — a relationship through which listings become visible/purchasable: Seller→Buyer,
  Seller→Broker, or Broker→Buyer.
- **Edge Terms** — the per-relationship settings chosen by the granting party: communication
  mode (relay/direct), identity reveal, minimum volumes, price floors.
- **Counterparty of Record (CoR)** — the Participant who owns the buyer relationship on an
  order. The CoR is fully responsible to that buyer for fulfillment, accuracy, and resolution.
- **Source Listing / Derived Listing** — a Seller's original listing / a Broker's re-listing of
  it under their own alias, unit, and price.

## 2. Eligibility & verification

- **Businesses only.** This is a B2B marketplace. Participants represent they are purchasing and
  selling for business purposes, not personal/household use. No consumer transactions.
- Every Participant passes identity/business verification (KYB) before transacting. The Platform
  always knows the legal identity behind every alias, even when counterparties don't.
- Verification tier caps: new accounts have order-size and volume limits that grow with
  completed-transaction history.
- One entity, one account (aliases per §1). Creating accounts to evade suspension, ratings, or
  relationship revocation is prohibited.

## 3. Masked identity — informed consent

- Participants **knowingly agree to transact with masked counterparties.** The alias, its
  verification badge, and its ratings history are the disclosed basis of trust.
- The Platform guarantees behind every mask: a KYB-verified business, a ratings history that
  cannot be reset by re-registering, and recourse through escrow and disputes.
- **Reveal is the grantor's one-way choice.** No Participant may attempt to unmask another —
  including through shipping documents, payment records, packaging, social engineering, or
  data correlation. Discovering an identity by accident creates no right to use or share it.
- The Platform generates all cross-party documents (labels, invoices, POs, manifest covers)
  with the correct masking per hop, so no party has to leak to operate.

## 4. Source protection & anti-circumvention

This is the heart of the deal. By using the Platform, every Participant agrees:

- **Relationships introduced by the Platform belong on the Platform.** For any counterparty you
  first met through the Platform (masked or revealed), you will not solicit, negotiate, or
  transact off-platform for **24 months** after your last Platform transaction with them.
- **Legitimate exit exists.** Parties who want to go direct may do so by paying the published
  **conversion fee** (modeled on Upwork's; exact schedule TBD) — after which the restriction
  lifts for that pair. Nobody is trapped; circumvention is just never free.
- All transaction-related communication happens on-platform. Messages are automatically
  filtered for contact information (emails, phone numbers, links, handles); filtered content is
  flagged. Pattern of circumvention attempts → warnings → suspension → ban with forfeiture of
  pending sponsored-placement fees (never of escrowed goods payments, which follow §7).
- A Broker's supplier list, a Seller's broker/buyer lists, and all relationship-graph data are
  confidential. No scraping, exporting, or sharing of other Participants' relationship data.
- Sellers approving a Broker grant a **resale right, not a relationship transfer**: the Broker
  may re-list per the Edge Terms, and may not represent the goods as their own manufacture, may
  not exceed authorization scope (e.g., price floors, territories if set), and loses derived
  listings automatically if the approval is revoked.

## 5. Willing & responsible entry into every transaction

Every order requires, **before payment**, an affirmative acknowledgment screen showing:

1. exactly what is being bought: product, **unit of sale and what it contains**, quantity, and
   total (unit math shown — "10 cases × 12 bottles = 120 bottles");
2. **condition grade** (New / Shelf-pull / Customer returns / Salvage) and whether the lot is
   manifested — with the manifest attached if so;
3. freight/FOB terms — who arranges and pays shipping, and where risk transfers;
4. the **dispute window and final-sale rules** that apply to this grade (§8);
5. who the counterparty of record is (by alias) and that identities are masked;
6. that the buyer is purchasing for business purposes.

No acknowledgment, no order. The acknowledgment record is part of the order's audit trail and
is what dispute resolution is judged against. MOQs and channel minimums are enforced by the
product — an order below a channel's minimum cannot be placed, only negotiated.

## 6. Counterparty of record — responsibility chain

- The CoR on each hop is fully responsible to their buyer: listing accuracy, routing choice,
  fulfillment, and dispute resolution — **regardless of who actually ships**. Choosing blind
  drop-ship does not delegate responsibility.
- Each hop is its own contract: Buyer↔Broker and Broker↔Seller disputes are resolved
  independently. A broker left holding a bad upstream lot pursues the Seller on the upstream
  hop; their buyer is made whole on the downstream hop first.
- Brokers choose their routing policy (auto-route / manual / hold) and own its consequences.

## 7. Money

- All payments flow through Platform escrow. Funds release to the CoR on buyer delivery
  confirmation, or automatically **N days after verified delivery** (default 3 business days)
  if the buyer neither confirms nor disputes.
- Platform fee: **[X]% of the retail hop** (target 3–5%), disclosed on every invoice before
  order placement. Wholesale (upstream PO) hops: no fee at launch (revisit with liquidity).
- Payouts split automatically per hop. Every party sees its own economics and only its own.
- Off-platform payment for platform-introduced transactions is circumvention (§4).
- Taxes (sales tax, resale certificates, import duties) are each Participant's own
  responsibility; the Platform collects resale certificate documentation from Buyers at
  onboarding where applicable.

## 8. Listing accuracy, grades & disputes

- Manifests and condition grades are **binding representations** by the listing owner.
- Default dispute windows by grade (CoR may extend, never shorten):
  - **New / Shelf-pull**: 5 business days from delivery for not-as-described claims.
  - **Customer returns**: 3 business days; variance within industry-standard tolerance
    disclosed on the listing (e.g., ±10% of manifest) is not disputable.
  - **Salvage / unmanifested**: final sale; disputes only for wrong-lot or quantity shortage.
- Dispute ladder: direct resolution between the parties (masked, on-platform, time-boxed) →
  Platform arbitration on the record (acknowledgment screen, manifest, photos, audit log,
  messages) → escrow disposition. The Platform's decision on escrow is final within the
  Platform; nothing waives statutory rights that can't be waived.
- Fraud (counterfeit goods, forged manifests, stolen property) → immediate ban, escrow freeze,
  cooperation with law enforcement.

## 9. Ratings integrity

- Only completed orders can rate; ratings are double-blind (simultaneous reveal or 14 days).
- Ratings attach to the alias and survive it — an alias cannot be abandoned to escape history.
- Retaliatory ratings, rating extortion ("5 stars or I dispute"), and coordinated manipulation
  are prohibited and reversible by the Platform on evidence.

## 10. Prohibited & restricted

- Prohibited: counterfeit or trademark-infringing goods, recalled products, stolen property,
  hazardous materials without certification, and any goods illegal to resell.
- Restricted (license required, provided at listing time): alcohol, tobacco, pharmaceuticals,
  medical devices, weapons, food requiring cold chain (certification of handling).
- The Platform may delist and hold escrow on any listing pending verification.

## 11. Platform's own rules

- The Platform is a marketplace facilitator, not a party to the goods contracts, not the seller
  of record, and holds no title to goods (attorney: confirm marketplace-facilitator tax
  obligations per state).
- The relationship graph, ratings data, and audit logs are Platform property; Participants get
  their own transaction data out anytime (their side only).
- Fee changes: 30 days' notice, never retroactive to open orders.
- Suspension/termination: for cause per these terms, with pending orders completed or refunded
  through escrow — enforcement never strands a counterparty's money or goods.
- Liability: Platform liability capped at fees earned on the transaction in question
  (attorney: standard caps, disclaimers, arbitration clause, governing law).

## 12. For the attorney (known gaps to close)

1. Entity + governing law/venue once the entity exists (see formation plan).
2. Convert §4's 24-month non-circumvention + conversion fee into enforceable covenant language
   (jurisdiction-dependent; Upwork/Fiverr precedents).
3. Marketplace facilitator sales-tax obligations by state; resale certificate handling.
4. Arbitration clause + class waiver enforceability (B2B makes this easier).
5. Payment/escrow structure: confirm operating as Stripe Connect platform keeps us out of
   money-transmitter licensing (destination charges + separate transfers pattern).
6. Data: privacy policy, relationship-graph data ownership vs. participant data rights.
7. Insurance: platform E&O; whether to require participant liability coverage above a volume tier.
