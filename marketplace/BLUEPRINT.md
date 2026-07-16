# TradeMesh — Broker-Network Marketplace Blueprint

> A gated wholesale marketplace where **sellers**, **brokers**, and **buyers** trade goods and
> services behind masked identities, and where a well-connected broker can build a storefront
> out of *relationships* instead of inventory. Think Upwork's gated communication + Airbnb's
> masked identities + Amazon's multi-offer listings — for B2B/wholesale.
>
> This doc is the full product/technical design. A working single-file prototype lives next to
> it at `marketplace/index.html` — open it in any browser to click through every mechanic below.

---

## 1. The core idea in one paragraph

Sellers list goods/services but **nobody can see or buy anything without being approved** by
whoever owns that sales channel. Sellers approve brokers; brokers re-list the sellers' goods
under their own storefront (their own unit sizes, their own prices, their own minimums) without
ever holding inventory. Buyers only see listings from channels that approved them — and to a
buyer, a broker looks exactly like any other seller. Identities are masked (storefront aliases,
Airbnb-style) until the relationship owner chooses to reveal. All communication runs through the
platform, filtered and flagged (Upwork-style) so relationships can't leak off-platform. Whoever
the buyer's relationship is with — seller-direct or broker — is the **counterparty of record**:
they control order routing and they're on the hook for problems. The platform provides the rails:
payments, escrow, invoicing, messaging, ratings, ads.

## 2. Actors & roles

Roles are **capabilities, not account types** — one account can hold several (a seller can also
broker other sellers' goods; a buyer can graduate to broker).

| Role | What they do | Who lets them in |
|---|---|---|
| **Seller / Provider** | Owns canonical products/services. Creates source listings. Approves brokers & buyers. Sets terms per relationship (comm mode, minimums, price floors, identity reveal). | Platform KYC/verification |
| **Broker** | Gets approved by sellers → gains access to their source catalog → creates *derived listings* under their own storefront (own unit, price, MOQ). Middleman for communication (or grants direct-with-consent). Counterparty of record to their buyers. | Sellers approve per-relationship |
| **Buyer / Consumer** | Sees only listings from channels that approved them. Orders, negotiates, pays, rates. | Invited/approved by sellers or brokers only |
| **Platform (us)** | Payments, escrow, invoicing, messaging + masking, ratings, ads, disputes, verification. | — |

Key asymmetry: **only sellers and brokers can invite buyers.** There is no public browse — the
network *is* the product.

## 3. The relationship graph (the heart of the system)

Everything hangs off directional, term-carrying **approval edges**:

```
Seller ──approves──▶ Broker      (broker gains source-catalog access + resell right)
Seller ──approves──▶ Buyer      (buyer sees seller's direct listings)
Broker ──approves──▶ Buyer      (buyer sees broker's derived listings)
```

Each edge carries its own **terms**, set by the grantor:

- **Communication mode**: `relay` (grantor's middleman relays every message) or
  `direct` (parties may message directly, still platform-filtered).
- **Identity reveal**: masked (default) or revealed. Reveal is a one-way door and always the
  grantor's choice.
- **Minimum volume / order size** (each channel sets its own — seller may say "pallets only",
  broker may break that same pallet into cases).
- **Price floor** (optional): seller can forbid brokers from listing below X to protect the
  brand/channel.
- **Status lifecycle**: `pending → approved → suspended/revoked`. Revoking a seller→broker edge
  automatically deactivates all of that broker's derived listings from that seller.

### Identity masking

Every account has a **storefront alias** ("Golden Harvest Wholesale ★4.8 · Verified") that is
the *only* identity counterparties see. Ratings, reviews, and verification badges attach to the
alias, so trust accrues without exposing the legal identity. Real identity is revealed only when
the relationship grantor flips the reveal flag (e.g., after the first successful order, or for
direct fulfillment/shipping needs — see §7 on blind shipping).

To a layman buyer, **a broker is indistinguishable from a seller.** That's deliberate — it's what
makes the broker's relationship capital sellable.

## 4. Products, listings & the derivation chain

Three layers, and getting this separation right is what makes the whole model work:

1. **Product** (canonical): owned by the seller. Name, description, images, category, base unit,
   spec sheets. One product, many listings.
2. **Source listing** (seller's channel): unit of sale (e.g., *pallet of 720*), price/price
   tiers, MOQ, available stock, negotiability, active flag.
3. **Derived listing** (broker's channel): points at a source listing. The broker chooses their
   own unit (*case of 12*), own price (their markup), own MOQ, own images/copy if they want.
   Stock is computed from source stock × unit conversion. Provenance (`derived from listing X`)
   is stored but **never shown to buyers** — only the seller and broker see the chain.

**Multi-offer view (the Amazon buy-box moment):** when a buyer is approved on several channels
selling the *same underlying product*, they see the offers grouped: "Organic Olive Oil —
3 offers: pallet of 720 @ $2,880 from Golden Harvest · case of 12 @ $69 from Nexus Sourcing ·
case of 12 @ $66 from Pacific Trade." Same lot, different units, different counterparties, and
the buyer picks. Two brokers reselling the same seller's lot compete on price/terms/service.

Derived listings can chain (broker-of-broker) — cap chain depth at 2 initially to keep margin
stacks and dispute chains sane.

## 5. Discovery without leaking

Buyers see three tiers in their marketplace:

1. **Unlocked listings** — full detail, can order/negotiate (channels that approved them).
2. **Locked teasers** — category, unit of sale, rating of the storefront, *blurred* everything
   else, with a **Request access** button. This is how the network grows without becoming
   public: you can tell *something* is there, not *what* or *who*.
3. **Nothing** — sellers can opt out of teaser visibility entirely (fully dark listings).

Access requests land in the channel owner's Requests queue with the requester's (masked) trade
profile: category, order-volume history on platform, rating as a buyer. Approve / decline /
approve-with-terms.

## 6. Communication: filtered, flagged, relayed

- All messaging is on-platform, per-thread, tied to a listing/negotiation/order.
- **Content filter** strips emails, phone numbers, URLs, and social handles, replaces them with
  `[contact info removed]`, and flags the message for review (Upwork model). Repeated
  circumvention attempts → strikes → suspension.
- **Relay mode** (default for broker channels): buyer messages "the storefront" → message lands
  in the broker's relay inbox → broker answers themselves *or* forwards upstream to the seller →
  seller's answer relays back down. The buyer never sees the seam.
- **Direct-with-consent**: broker (or seller) can open a direct thread between buyer and seller
  while keeping identities masked. The broker's economics are protected by the platform (orders
  still route through the broker's channel), not by information asymmetry alone.

## 7. Orders, routing & responsibility

**The rule: whoever owns the buyer relationship is the counterparty of record.** They control
routing, they collect the buyer's payment, they own the dispute.

Order lifecycle on a broker channel:

```
Buyer places order (sees only broker storefront)
  → Broker routing policy fires:
      • AUTO   — platform instantly creates the upstream PO to the seller at the broker's cost basis
      • MANUAL — broker reviews, then routes: forward to seller / fulfill from own stock / hold
  → Seller confirms upstream PO → fulfills
      • Blind drop-ship: seller ships with the BROKER's storefront branding on the label
        (platform generates the label so the seller never learns the buyer & vice versa)
      • Or ship to broker, broker re-ships (broker's choice, broker's cost)
  → Delivery confirmation → escrow release → split payout
```

Direct seller→buyer orders are the degenerate case: one hop, no upstream PO.

**Provenance is role-scoped:** the seller sees `my listing → broker X → (masked buyer)`, the
broker sees the full chain, the buyer sees only their counterparty.

## 8. Money: escrow, invoicing, splits

- Buyer pays the platform (card/ACH/wire) → funds held in **escrow** → released on delivery
  confirmation (or milestone completion for services).
- **Automatic split** on release: seller's cost basis → seller; broker margin → broker;
  platform fee (take rate, e.g. 3–5% + payment costs) → platform. One payment in, N payouts out
  (Stripe Connect destination-charges pattern).
- **Invoicing**: platform generates invoices/POs at every hop (buyer↔broker, broker↔seller) with
  the correct masked/unmasked identities per document.
- **Negotiation**: offer → counter → accept binds price for that order (or creates a private
  price tier for that buyer on that channel). All negotiation happens per-channel, so a broker
  negotiating down their buyer doesn't touch the seller's price.
- Later: net-30 terms, trade credit, financing partners (§12).

## 9. Ratings & reputation

- **Double-blind, order-triggered** (both sides rate before either sees the other's review, or
  after 14 days). Only completed orders can rate — no drive-by reviews.
- Ratings attach to the **storefront alias**, and are **role-scoped**: an account has separate
  scores *as seller*, *as broker*, *as buyer*. A broker's seller-facing score ("routes orders
  fast, pays upstream on time") is what earns them more seller approvals; their buyer-facing
  score is what makes their storefront sell.
- Dimensions, not just stars: accuracy of listing, communication, fulfillment speed, payment
  reliability (for buyers).
- Ratings feed the **access-request trade profile** (§5) — reputation is the currency that gets
  you approved into more of the network.

## 10. Ads & promotion (within the walls)

Ads never breach the approval graph — you can only advertise to people already allowed to see
you:

- **Sponsored listings**: pay to pin a listing at the top of the marketplace for buyers already
  in your channel. (CPC or flat-fee-per-week to start; auctions later.)
- **Category boosts** in locked-teaser space: a seller can pay to make their *teaser* more
  prominent to unapproved buyers ("someone in your category sells this — request access").
- **Storefront features**: featured placement in the broker directory sellers browse when
  deciding whom to approve.
- House rails: platform email digests ("3 new listings in your network"), re-engagement.

## 11. Trust, safety & platform protection

- **KYB/KYC verification** at signup (business registration, bank account) → "Verified" badge.
- **Anti-circumvention**: comm filtering (§6) + the escrow/payment lock-in + a contact-reveal
  fee option (like Upwork's conversion fee) for relationships that want to go direct legitimately.
- **Disputes**: counterparty of record owns it; platform arbitrates with the full provenance
  chain + message history; escrow holds until resolution.
- **Graduated exposure**: new accounts get order-size caps that grow with history.
- **Audit log** on every relationship change, price change, and routing decision — disputes in a
  masked-identity system live and die on the audit trail.

## 12. Expansion paths (roughly in order)

1. **Services vertical** (the Upwork half): milestone escrow, SOWs, deliverable review flow —
   same relationship graph, different fulfillment object.
2. **Logistics integrations**: rate-shopping, label generation (enables true blind drop-ship),
   freight for pallets, tracking webhooks.
3. **Financing**: net-30/60 terms underwritten by order history; invoice factoring for sellers;
   the platform knows both sides' cashflow — that's a lending moat.
4. **Sample orders**: below-MOQ one-off orders at sample pricing to bootstrap trust.
5. **RFQs / reverse auctions**: buyers post demand into channels they're approved on; sellers
   and brokers bid.
6. **Group buying**: brokers aggregate many small buyers to hit a seller's pallet minimum —
   this is the broker's superpower formalized as a feature.
7. **Private label / white-label storefronts**: broker gets their own branded subdomain; the
   network becomes invisible infrastructure.
8. **API/EDI** for sellers' ERPs (inventory sync, order push) — stickiness with serious sellers.
9. **AI matchmaking**: recommend broker↔seller approvals from category/volume/rating fit;
   auto-draft derived listings; negotiation assistants.
10. **Analytics**: sell-through dashboards for sellers ("which brokers move my product"),
    margin analytics for brokers, spend analytics for buyers.
11. **Tiered subscriptions**: free + take-rate to start; Pro tiers (lower take rate, more
    sponsored slots, API access, multiple storefront aliases) once liquidity exists.

## 13. Data model (entities → real schema later)

```
Account          id, legalName, verified, kycStatus, createdAt
User             id, accountId, email, role-in-org
Storefront       id, accountId, alias, tagline, avatar, revealPolicy      ← identity mask
RoleGrant        accountId, role (seller|broker|buyer)
Relationship     id, grantorId, granteeId, kind (broker|buyer), status,
                 terms { commMode, reveal, minVolume, priceFloor }, invitedBy, auditTrail[]
Product          id, ownerId, name, desc, category, images[], specs
Listing          id, channelOwnerId, productId, sourceListingId?,          ← null = source listing
                 unit, unitsPerPack, moq, price, priceTiers[], stock, active,
                 negotiable, teaserVisible, sponsored
Negotiation      id, listingId, buyerId, status, history[{by, qty, price, note, at}]
Order            id, buyerId, channelOwnerId, listingId, qty, unitPrice, total,
                 status (placed|routed|confirmed|shipped|delivered|disputed|closed),
                 upstreamOrderId?, provenance[], routingMode, timeline[]
Invoice / PO     id, orderId, fromId, toId, amount, maskingProfile, status
LedgerEntry      id, orderId, party, type (escrow_in|payout|fee|margin), amount
Thread           id, contextType (listing|negotiation|order), contextId,
                 parties[], relayVia?, messages[{from, raw, filtered, flagged, at}]
Review           id, orderId, raterId, targetStorefrontId, asRole, stars, dims{}, text
AdCampaign       id, accountId, listingId, type (sponsored|teaser_boost|feature),
                 budget, impressions, clicks, status
Invite           code, issuerId, kind, usedById?, expiresAt
Dispute          id, orderId, openedBy, status, resolution, escrowHold
AuditEvent       id, actorId, entity, action, before/after, at
```

## 14. Tech roadmap

| Phase | What | Stack |
|---|---|---|
| **0 — Prototype (done, in this folder)** | Clickable single-file simulation of every mechanic; use it to pressure-test the model with real sellers/brokers before writing a backend. | `index.html`, localStorage, role-switcher |
| **1 — MVP** | Real accounts, one category, manual KYC, Stripe Connect escrow+splits, relationship graph, source+derived listings, relay messaging with regex filtering, invoicing, double-blind ratings. | Next.js + Postgres (**row-level security is the natural fit for the approval graph**) + Stripe Connect + Resend/Twilio |
| **2 — Liquidity** | Access-request marketplace, teasers, sponsored listings, negotiation, sample orders, mobile PWA. | + Redis, background jobs, S3 image pipeline |
| **3 — Moats** | Logistics, financing, API/EDI, AI matchmaking, subscriptions. | + partner integrations |

**Why RLS matters:** every query in this product is "what is this account allowed to see through
the relationship graph" — encoding that once, at the database layer, prevents the entire class
of identity-leak bugs that would kill trust in a masked marketplace.

## 15. Open questions to settle with real users

- Take rate: flat % vs. per-role (charge the broker's margin, not the seller's base?).
- Should sellers see the broker's resale price? (Price floor says yes at least partially.)
- Reveal economics: free after N orders vs. paid conversion fee.
- Who eats payment processing on multi-hop splits.
- Category to launch with (food/bev wholesale? closeouts/liquidation? — liquidation lots fit the
  broker model unusually well).

---

*Prototype: open `marketplace/index.html` (or the GitHub Pages URL once merged) and use the
"Acting as" switcher to walk the same order through buyer → broker → seller.*
