# Setup Playbook — Entity, Payments & Merchant Approval

> Founder-facing operational notes. **Not legal, tax, or financial advice** — every item marked
> ⚠️ needs your attorney/accountant to confirm before you rely on it. This exists so the platform
> is built and launched in the order that keeps money *flowing* instead of *frozen*.

The single most important fact on this page: **a marketplace that collects a buyer's money and
pays it out to a seller is moving money on behalf of others.** Do that on an ordinary
Stripe/Square/PayPal merchant account and you *will* get frozen the moment volume looks real —
that's called "aggregation" and it's an instant account-killer. Everything below is about not
doing that.

---

## 1. The payments architecture (decide this first — it shapes everything)

### Use a *platform/marketplace* payments product, never a flat merchant account

| What you are | What you must NOT use | What you use instead |
|---|---|---|
| A marketplace paying many sellers | A normal Stripe/Square/PayPal merchant account (= illegal aggregation → frozen funds) | **Stripe Connect** (each seller is their own verified sub-merchant) |

**Why Stripe Connect is the default answer:**

- Each seller/broker becomes a **connected account** — Stripe KYC/KYB-verifies them individually.
  You are not aggregating; you are facilitating between verified sub-merchants. This is the exact
  thing that keeps you off the "money transmitter" hook. ⚠️ (Confirm with counsel that the
  destination-charge / separate-charges-and-transfers pattern keeps *you* out of money-transmitter
  licensing — Stripe is the transmitter of record, but state rules vary.)
- **Split payouts** are native: one buyer charge → seller cost basis to the seller, broker margin
  to the broker, platform fee to you. This is precisely the multi-hop model in the blueprint.
- **Escrow-style holds**: you can hold funds in the platform balance and release on delivery
  confirmation via delayed transfers. ⚠️ Holding *briefly* until delivery is normal; holding *long*
  or acting bank-like invites scrutiny. For true long-hold escrow consider a licensed partner
  (below).

**Alternatives, and when they matter:**

- **Adyen for Platforms** — same model, better at scale/international; heavier onboarding. Revisit
  at volume, not at launch.
- **Dwolla / ACH rails** — wholesale invoices are often paid by ACH/wire, not card. ACH is far
  cheaper (~$0.25–1 flat vs ~2.9%+$0.30 on cards) and chargeback-light. Strongly consider
  **offering ACH as the default for large B2B orders** and cards for smaller ones. Stripe does ACH
  too, so you may not need a separate provider day one.
- **Dedicated escrow (Escrow.com API, Trustap, Balance, Tilled)** — if buyers demand real
  third-party escrow for big lots, or if the delivery-hold window is long. **Balance** and
  **Tilled** are built specifically for B2B marketplaces (net terms, ACH, invoicing) and are worth
  a look as a Stripe alternative once you have volume.

### The escrow nuance (closeouts makes this real)

Closeout lots are big-ticket and "as-described" disputes happen — so your hold-until-delivery
window matters. Launch plan: **Stripe Connect with manual/delayed payout**, release on buyer
delivery confirmation (auto-release N days after delivery). If lot sizes or disputes push hold
times long, graduate to a licensed escrow partner. ⚠️ Confirm the hold mechanism with Stripe's
policy team in writing before launch so it isn't reclassified as unlicensed escrow.

---

## 2. Getting merchant-approved cleanly (and staying approved)

Underwriters approve or freeze you on a handful of signals. Hit all of them *before* you apply:

**Before you apply — the checklist underwriters actually check:**

1. **Registered entity + EIN + business bank account, all name-matched.** Mismatched names are a
   top decline reason.
2. **A live website with real policies** — Terms of Service, Privacy Policy, refund/dispute policy,
   and a reachable contact/address. The underwriter *visits the site*. "Coming soon" = auto-decline.
   (Your `MARKET-TERMS.md` is the raw material; the lawyer turns it into the posted ToS.)
3. **Apply AS a platform/marketplace from day one.** In Stripe: enable Connect and describe the
   business as a wholesale/closeout B2B marketplace. Do not sign up as a plain store and bolt
   Connect on later — and never misrepresent the model. Misrepresentation is what turns a review
   into a *permanent* ban with a 180-day fund hold.
4. **A clear, honest business description + realistic volume estimate.** Underwriters flag "vague"
   and they flag "sudden 10× over stated volume." Estimate high enough that early growth doesn't
   trip a review.
5. **KYB your sellers** — Connect does this for you; keep it on. Verified sub-merchants are the
   whole basis of not-aggregating.

**What keeps you approved after launch:**

- **Chargeback ratio under ~0.9%.** Good news: **B2B/wholesale chargebacks run far lower than
  consumer retail** — invoices, POs, verified businesses, and your acknowledgment-screen audit
  trail (from `MARKET-TERMS.md` §5) are exactly what wins disputes. Your model is *favorable* here.
- **Category cleanliness.** Closeouts/liquidation is a fine, approvable category. The risk is
  *what's inside the lots*: supplements, electronics-with-warranty, recalled goods, and
  brand-restricted items raise chargeback and brand-complaint risk. Your prohibited/restricted-goods
  rules (`MARKET-TERMS.md` §10) are also a *merchant-approval* asset — keep them enforced.
- **Never let the platform balance look like banking.** Release funds on a defined event
  (delivery), not "whenever." Document the flow.

---

## 3. Entity: existing "STY Enterprises" vs. a fresh entity

Both can work. The trade-off:

| | Start under STY Enterprises (existing) | Form a fresh entity |
|---|---|---|
| **Speed** | ✅ Fastest — EIN + bank already exist; can apply for Connect almost immediately | ❌ Days–weeks to form + open bank |
| **Underwriting** | ✅ Existing clean history helps *if* STY's profile isn't in an unrelated/high-risk MCC ⚠️ | Neutral — no history either way |
| **Liability** | ❌ Marketplace (holding others' money, disputes) shares liability with STY's other business | ✅ Isolated — the reason marketplaces get their own entity |
| **Clarity for the lawyer/accountant** | ⚠️ Mixed activities under one entity complicate books & the money-transmitter analysis | ✅ Clean single-purpose entity |

**Recommended path:** **Pilot under STY Enterprises** to move fast and validate — its existing EIN
and banking relationship genuinely speed Connect approval — but describe the marketplace activity
accurately in the Connect application (don't let it ride under STY's existing MCC if that's
unrelated). ⚠️ Ask your accountant whether STY's current classification is compatible; if STY is in
an unrelated/high-risk category, form the fresh entity instead.

**DECIDED: STY Enterprises (S-corp) is the starting entity.** S-corp works fine for Stripe
Connect and marketplace operation. Two S-corp-specific notes for the accountant/attorney ⚠️:
(1) S-corps have shareholder restrictions (≤100, individuals/US persons) — irrelevant for
operating, relevant only if outside investors ever come in, which is one more reason the
marketplace migrates to its own entity (likely LLC or C-corp) before raising or real scale;
(2) keep the marketplace's books separable inside STY from day one (its own bank account and
class/ledger) so the eventual spin-out is clean. **Migrate to a dedicated entity
before real GMV** — a business holding other people's escrowed money should not share a balance
sheet with anything else. This mirrors the blueprint's DBA-now / entity-before-scale guidance.

**Chain to be live on payments:** entity/EIN (have it via STY) → name-matched business bank
(Mercury/Relay if you want a fresh dedicated account) → **Stripe Connect** platform application →
post ToS/Privacy on the live domain → test-mode transactions → go live.

---

## 4. Domain shortlist (all verified available at standard registration, not resale)

Checked at standard registrar pricing (~$11.25/yr `.com` unless noted). Availability ≠ trademark
clearance — **run a USPTO TESS search on the winner before printing anything.** ⚠️

**Theme — discretion / source-protection (on-brand for a masked marketplace):**
`quietlot.com` · `lotmask.com` · `veiledlot.com` · `tradveil.com` · `sourcemask.com` ·
`keptlot.com` · `sourcekept.com` · `tradehush.com` · `lotcurtain.com` · `tradecurtain.com` ·
`palletveil.com`

**Theme — trade / lots / storage:**
`vaultlot.com` · `lotward.com` · `closelot.com`

**Get-prefixed / alt-TLD (fine for a launch brand):**
`getquietlot.com` · `trademesh.co` ($29.99) · `greymarket.co` ($29.99)

**Top picks for closeouts:** **QuietLot** (discretion + lots, phone-friendly, on-message),
**LotMask** (states the mechanic), **TradeVeil** (travels beyond closeouts into other broker
verticals). Grab your top two today at ~$11 each before they move — they're cheap enough to hold
both while you decide, and cheaper than losing the one you want.

**Already taken (stop chasing):** trademesh.com, trymesh, meshtrade, meshmarket, meshwholesale,
lotledger, lotwise, lotly, lotnexus, trustlot, harborlot, lotharbor, quiettrade, hushtrade,
closemesh, palletpost, casebreak, brokerbridge, palleto, palletto, tradolo, brokra, veylo, velqo,
hazl, tradna, lotara, closora, veqto, mercato, wholecrate, sourceveil, lotveil, veilmarket,
brokermesh, tradeshade, lotpact, midlot, sourcelot, palletrelay, shelflink, greylot, lotbridge,
lotrelay, lotmesh, lotvault, maskedmarket, backshelf, brokerlot, tradeveil.

---

## 5. Sequenced to-do (money-safe order)

1. **Pick the name**, buy the domain (+ hold a second), point DNS at Vercel.
2. **Confirm entity** with accountant: pilot under STY or form fresh (§3). ⚠️
3. **Attorney** turns `MARKET-TERMS.md` into posted ToS + Privacy + refund/dispute policy.
4. **Stand up a landing page** on the domain with those policies live (needed for merchant review).
5. **Apply for Stripe Connect** as a B2B wholesale/closeout marketplace; enable ACH. ⚠️
6. **Build MVP** (Next.js + Postgres + Connect per blueprint §16); test-mode the full
   escrow-split-payout flow before going live.
7. **Onboard the first cohort** (your SSD + vendor lists) as verified connected accounts.
