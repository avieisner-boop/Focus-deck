# TradeMesh — Broker-Network Marketplace Blueprint

> A gated wholesale marketplace where **sellers**, **brokers**, and **buyers** trade goods and
> services behind masked identities, and where a well-connected broker can build a storefront
> out of *relationships* instead of inventory. Think Upwork's gated communication + Airbnb's
> masked identities + Amazon's multi-offer listings — for B2B/wholesale.
>
> This doc is the full product/technical design. A working single-file prototype lives next to
> it at `marketplace/index.html` — open it in any browser to click through every mechanic below.

---

## 0. The pitch in one line

**Turn inquiry-based, relationship-hidden markets into selectable catalogs — without exposing
the relationships.** In broker-driven markets today (wholesale, closeouts, freight, lending,
insurance) a buyer *inquires* and waits while a broker shops their private contacts opaquely.
Here the broker's relationships are pre-provisioned onto a shelf: the buyer **selects** from
live, priced, in-stock options — and the broker's source stays protected the whole time. That
"select, don't inquire" inversion is the product; everything below exists to make it safe for
the people whose relationships are the inventory.

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

**Full-catalog syndication ("provide all through broker"):** instead of the broker deriving
listings one by one, a seller can grant a broker *catalog-wide* syndication on the relationship
edge. The broker sets a markup rule once (e.g., cost + 30%, or a per-category table) and every
current and future source listing auto-derives into their storefront — new seller inventory
appears on the broker's shelf the moment it's listed, still masked, still routed through the
broker as counterparty. This is what makes a well-connected broker's storefront feel *fully
stocked* on day one, and it's why buyers can select instead of inquire: the broker's
relationships are provisioned as a live catalog, not a rolodex they query on demand.

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
- **Recruiting ads flow one tier down** (see §13): sellers advertise masked to brokers ("join my
  channel"), brokers advertise to buyers. Never past a discovery shield.
- **Category boosts** in locked-teaser space: a seller can pay to make their *teaser* more
  prominent to unapproved buyers ("someone in your category sells this — request access").
- **Storefront features**: featured placement in the broker directory sellers browse when
  deciding whom to approve.
- House rails: platform email digests ("3 new listings in your network"), re-engagement.

### Blind ad routing (promotion without exposure)

The ad system inherits the identity guarantees — an advertiser can promote hard without ever
leaking who they are:

- **Platform-served creative only.** The platform hosts every ad asset and renders every
  creative under the storefront alias. No advertiser URLs, pixels, UTM tags, or third-party
  trackers — the same contact-stripping filter that guards messages guards ad copy.
- **Click routing dies at the platform's door.** Every click resolves through a platform
  redirect to an on-platform destination (listing, teaser, or gated invite page). There is
  nothing in the ad, the link, or the analytics that identifies the business behind the alias.
- **Identity-free attribution.** Advertisers see impressions/clicks/orders per alias-served
  campaign; viewers are counted, never identified to the advertiser. Neither side of an ad can
  deanonymize the other.
- **Teaser boosts respect the graph.** Promoting to buyers outside your channel shows category,
  unit, alias + rating, and a Request Access button — nothing more — and **discovery shields
  still apply** (§13): a boost never surfaces a supplier to a broker's shielded buyers.
- **Off-platform acquisition (Google/Meta) lands on gated pages.** External ads never link to a
  storefront; they land on invite/request-access pages that show only category + alias. The
  platform is the advertiser of record externally, so no seller's name ever appears in an ad
  network's transparency library.
- Ad billing is separate from trade settlement (card-on-file subscription), so ad spend never
  touches escrow.

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

## 13. Broker protection & channel precedence (the disintermediation firewall)

Premise: in wholesale, **availability, MOQ flexibility, and trust beat price within reason** —
so the platform protects intermediation by construction instead of racing every deal to the
cheapest hop. These rules make cutting out the middleman structurally unattractive.

### Discovery shielding

- A broker's suppliers are **invisible to that broker's buyers by default** — no teasers, no
  directory entries, no access requests. If a buyer purchases through broker B and B sources
  from seller S, the buyer cannot discover S exists on the platform.
- The broker can lift the shield per supplier ("allow my buyers to discover this supplier") —
  the broker's call alone, never the buyer's or the seller's.
- Multiple brokers, mixed settings → **the most protective setting wins**: shielded if *any* of
  the buyer's brokers sources from that seller and keeps the shield on.
- Shielding gates **discovery only** — it never hides or breaks relationships that already
  exist. A buyer who was direct with a seller before joining a broker stays direct.

### Going direct is seller-initiated only

- Where a shield applies, the buyer has no request path to the supplier. The only way a direct
  seller↔buyer relationship forms there is a **seller-initiated invite**, which the buyer may
  accept or decline. (The seller weighs one direct buyer against the broker relationship that
  moves volume across many buyers — that tension is intentional.)
- Recruiting ads flow one tier down, never past a shield: sellers advertise (masked) to brokers
  to join their channels; brokers advertise to buyers. Sellers do not advertise direct-to-buyer
  around a broker's shield.

### Band partition when direct + broker coexist

When a buyer holds both a direct seller channel and broker channels for the same product:

- Quantity bands partition at the seller's unit/MOQ: **the seller's band is the seller's**
  (1 pallet and up), **below it belongs to brokers** (1 case up to just under a pallet). The
  buyer sees both, labeled, and picks by quantity need.
- **Relationship precedence beats price**: inside the seller's band, broker offers are
  suppressed *even when cheaper*. There is no "cheaper through my broker" arbitrage against a
  seller who chose to go direct; the seller's price-floor terms already bound the other
  direction.
- Open negotiations and orders are grandfathered at their existing terms when a direct invite
  lands mid-deal.

### Item removal & re-add at seller discretion

- A buyer who prefers broker options for a specific item may **remove that item from their
  direct channel** — the direct offer disappears and broker offers repopulate uncapped (any
  broker, not just the original one).
- Getting the item back on the direct channel is **at the seller's discretion** (request →
  approve/decline). Dropping direct is easy; regaining it is not guaranteed — which makes
  dropping a broker a real decision, not a toggle to flip for one cheap order.

### Fee schedule favors intermediated flow

- **Broker volume tiers**: the platform fee on a broker's retail hop steps down with delivered
  volume (launch schedule: 3% → 2.5% at $10k → 2% at $50k) — productive brokers keep more
  margin the more they move.
- **Seller-side discount**: any seller-facing fees are lower on broker-routed volume than on
  direct volume — selling through brokers is the cheaper way to sell.
- Net effect: the platform's own economics reward keeping brokers in the deal, reinforcing the
  anti-circumvention terms instead of fighting them.

## 14. Broker verticals — one graph, many markets

The relationship graph, identity masking, relay messaging, ratings, and ads are
**vertical-agnostic**. What changes per vertical is the *fulfillment object* (what a completed
transaction is) and the *settlement shape* (how money moves). Design the core so `vertical` is a
first-class field with a pluggable fulfillment workflow, and the same platform hosts all of
these — each "a little different," all recognizably the same machine:

| Vertical | Seller / Broker / Buyer | Offer object | Fulfillment | Settlement | Watch out for |
|---|---|---|---|---|---|
| **Wholesale goods & closeouts** (launch) | Manufacturer, liquidator / goods broker / reseller | Listing (lot or replenishable) | Shipment, POD | Escrow → split payout | Freight damage disputes; manifest accuracy |
| **Transport / freight** (goods or vehicles) | Carrier / freight broker / shipper | Capacity or posted load | The haul, proof-of-delivery | Escrow until POD (factoring later) | FMCSA broker authority + $75k bond (US); insurance certs |
| **Business financing** | Lender / ISO-loan broker / borrower | Term sheet (rate, amount, term) | Funding event | Commission ledger on funded deals (no escrow) | State lending & broker licensing; disclosure rules |
| **Personal loans** | Lender / broker / consumer | Credit offer | Funding event | Commission ledger | Heaviest regulation (TILA/ECOA, adverse-action notices) — likely lead-gen only, late-stage |
| **Insurance** | Carrier / licensed agent-broker / insured | Quote | Bound policy | Commission ledger | Per-state, per-line licensing |
| **Miles / rewards points** | Points holder / points broker / traveler | Redemption offer | Ticket/booking issued | Escrow until issuance | Program T&Cs prohibit sale; fraud-heavy; gray-market — eyes-open, maybe never |

Two things make this work with one codebase:

- **Generalize `Listing` → `Offer`** with a vertical-specific payload schema, and
  `Order` → `Request` (an order, a load tender, a credit application, a quote request). The
  buyer-side experience is identical everywhere: *a shelf of selectable, priced options from
  masked counterparties you've been approved into* — the inversion in §0. In lending and freight
  today you inquire and wait; here you compare term sheets or quotes side by side, from which
  storefront, and pick.
- **Two settlement shapes cover everything**: escrow-and-split (physical fulfillment) and
  commission-ledger (financial products where the platform never touches principal). Both
  already exist in the data model (`LedgerEntry`).

Sequencing: goods first (unregulated, escrow model already built), **freight second** — it's the
same offline anonymity culture (carriers and shippers are masked from each other today, brokers
bond-protected in the middle) *and* it composes with goods: the transport-broker vertical can
carry the closeout orders moving on-platform, which makes each vertical the other's customer.
Financial verticals (financing, insurance) come later behind licensing, likely as
commission-tracked lead-gen before any origination. Points brokering only with legal eyes open.

### Launch vertical: wholesale closeouts

Closeouts/liquidation fit the broker model unusually well — sourcing relationships (store
raids, overstock contacts, bankruptcy channels) *are* the business, and everyone in the chain
already works hard to protect their source. What the goods core needs added for lots:

- **Lot semantics**: one-off finite quantity (not replenishable stock), first-committed-wins,
  time-boxed listings; auctions/best-offer windows fit naturally (§12.5).
- **Manifests**: attachable manifest files (the standard artifact of the liquidation trade) with
  per-line item detail; "manifested vs unmanifested" as a listing attribute that feeds pricing
  and dispute rules.
- **Condition grades**: new / shelf-pull / customer returns / salvage — a structured field, not
  free text, because it drives ratings disputes.
- **FOB / freight terms** on the listing (buyer-arranged vs delivered), which is also the
  natural bridge into the freight vertical.
- **All-sales-final flags** with grade-dependent dispute windows.

## 15. Data model (entities → real schema later)

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

## 16. Tech roadmap

| Phase | What | Stack |
|---|---|---|
| **0 — Prototype (done, in this folder)** | Clickable single-file simulation of every mechanic; use it to pressure-test the model with real sellers/brokers before writing a backend. | `index.html`, localStorage, role-switcher |
| **1 — MVP** | Real accounts, one category, manual KYC, Stripe Connect escrow+splits, relationship graph, source+derived listings, relay messaging with regex filtering, invoicing, double-blind ratings. | Next.js + Postgres (**row-level security is the natural fit for the approval graph**) + Stripe Connect + Resend/Twilio |
| **2 — Liquidity** | Access-request marketplace, teasers, sponsored listings, negotiation, sample orders, mobile PWA. | + Redis, background jobs, S3 image pipeline |
| **3 — Moats** | Logistics, financing, API/EDI, AI matchmaking, subscriptions. | + partner integrations |

**Why RLS matters:** every query in this product is "what is this account allowed to see through
the relationship graph" — encoding that once, at the database layer, prevents the entire class
of identity-leak bugs that would kill trust in a masked marketplace.

## 17. Open questions to settle with real users

- Take rate: flat % vs. per-role (charge the broker's margin, not the seller's base?).
- Should sellers see the broker's resale price? (Price floor says yes at least partially.)
- Reveal economics: free after N orders vs. paid conversion fee.
- Who eats payment processing on multi-hop splits.
- ~~Category to launch with~~ **Decided: wholesale closeouts/liquidation** (see §14) — the
  broker model's natural home. Remaining sub-question: which closeout niche first (general
  merchandise, apparel, electronics returns?).

---

*Prototype: open `marketplace/index.html` (or the GitHub Pages URL once merged) and use the
"Acting as" switcher to walk the same order through buyer → broker → seller.*
