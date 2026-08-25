# 03-SERVICES — intents.md Trust Services & Receivers

**Layer:** 3 (competing services at the edge) · **Status:** Non-normative

## 1. Status and design rule

This document describes patterns for trust services and receiver policies. Nothing here is mandatory; all of it is encouraged. The design rule: **hardened floor, extensible above, best layer wins at the edge.** The core ([CORE]) defines the format and rejects everything outside it; Layer 2 ([TRUST]) standardizes the hooks; Layer 3 is where services compete. Each receiver — directory, agent, renderer — chooses which layers to apply. The market of receivers, not the spec, decides which trust layer wins.

```
Layer 3 — Competing trust services (choose freely, per receiver)
          reputation graphs · bonded/staked reputation · proof-of-work
          stamps · paid ranking · payment rails (x402, escrow, bank)
          directory admission policies · review aggregators
          matching engines (rfp ↔ listing) · inverted advertising
          (offer delivery against declared intent) · longevity
          provenance (key age + signed history) · community badges
          (signed endorsements) · TEE-backed signing (Secure Enclave,
          HSM — hardened key custody) · timestamping & witness notaries
          (RFC 3161 / OpenTimestamps) · directory checkpointing

Layer 2 — Interop hooks (in core, from day one) — [TRUST]
          signature block · issuer-url + /.well-known key binding
          review type · contact-as-endpoint · payment rails list
          status lifecycle

Layer 1 — Hard core (never extended by trust logic) — [CORE]
          plain text · no execution, ever · leniency rules · size limits
```

Layer 2 is the load-bearing idea: hooks standardized so Layer 3 services compose across implementations instead of siloing.

## 2. The anti-abuse ladder (lowest cost to highest)

1. **Sybil cost via domain binding (L2, free, live now).** Domains cost money and registrars kill abuse cycles — an identity that self-replenishes its cost. The default floor every receiver can apply.
2. **Emergent reputation via signed reviews (free).** `review` documents referencing issuer keys aggregate into a web of trust. Nobody builds "the reputation system"; it is what the `review` type becomes when agents aggregate it. Reputation is earned, never bought.
3. **Proof-of-work stamps (free, costly at scale).** Hashcash-style stamps per submission, demanded by directories as admission policy. Free for an honest publisher; ruinous for a million-document spam run. Directory policy, never protocol.
4. **Bonded reputation (skin in the game).** An issuer posts a stake on any rail — crypto escrow, fiat deposit with a bond service — forfeited if reviews prove fraud. The crucial distinction: **bonded ≠ purchased.** Purchased ranking ("pay more → more reputation") is plutocracy: the rich spammer outbids the honest seller. Bonding inverts it: money buys *risk*, not *rank*.
5. **Payment-gated interaction (x402 and friends).** Because `contact` is just a URL, an issuer's contact endpoint may itself demand payment (HTTP 402): *messaging me costs 0.50 USDC*. Bot spam dies economically; serious buyers don't notice. Composition modes, all valid today with zero spec change: (a) `payment: x402-usdc` as a rail string; (b) 402-required contact endpoints; (c) directories charging admission or bonds via any rail. Best rail on top wins — crypto competes with escrow APIs, bank rails, and everything else, at the edge.

*L2 binding note: the L2 rung above is defined normatively in [TRUST] §2.1 — registrable-domain derivation (PSL eTLD+1), verifier-controlled key fetch, free-host tenancy cap demoting PSL PRIVATE-section hostnames to L1-equivalent.*

*L2 is a floor, not a ceiling. Registrable domains cost money but not much: a determined spam run at ~$1–15/domain is still feasible. L2 kills free-sybil, not paid-sybil. Directory admission policy (stamps rung 3, bonds rung 4, first-seen checkpoints) is the load-bearing defense at scale — directories SHOULD treat L2 as the entry floor and stack admission costs above it from launch, not retrofit them after the first flood.*

*Paid-sybil scaling: proof-of-control prices identity at one domain; volume floods still scale with domains purchased. Directories SHOULD combine proof-of-control with per-registrable-domain rate limits and first-seen checkpointing (§3) so paid-sybil cost scales with volume, not just identity. Below that, an honest limit stands: L2 prices identity, not intent — accepted, not solved.*

## 3. Matching, advertising, and community trust (patterns)

- **Matching engines.** `rfp` (demand) and `listing` (supply) are duals; any party may run a matching service over public documents without permission. Engines MAY rank by any signal, including geography (`ships-from` / `ships-to` express locality) and `attr-` attributes.
- **Inverted advertising.** Today's advertising pushes offers at people who never asked. In an intent-first web, offers arrive at the agent *because intent was declared* — an `rfp` says what is wanted, and sellers pay to deliver a relevant offer against it (e.g. via 402-gated contact). Payment-to-deliver replaces payment-to-interrupt; spam dies economically because the declared-intent channel is paid and the unsolicited channel is filtered.
- **Longevity provenance (chain-free).** Trust accrues to a signing key with a dated, verifiable document history: an issuer whose key has signed honest documents for years is demonstrably not a hit-and-run sybil. Wallet-provenance economics — old keys are worth protecting — without any chain or token.
- **Community badges (chain-free).** A community, brand, or club issues signed `review` / `profile` documents endorsing an issuer. A badge is a signature, verifiable by anyone; a badge issuer is accountable for their endorsements — their own key's reputation is at stake.
- **TEE-backed signing.** The strongest custody posture: the signing key lives in hardware and never exists in extractable form ([TRUST] §6).
- **Rfp flood posture.** Public `rfp`s invite solicitation floods by design (acknowledged, not "solved"). Publishers with broad demand SHOULD use payment-gated contact endpoints (402-style); receivers SHOULD apply stricter greylist defaults to unsigned `rfp` volume than to unsigned `listing` volume — demand-side spam is cheaper than supply-side spam, so the filter is asymmetric by design.
- **Directory checkpointing (SHOULD).** Directories SHOULD store the first-seen canonical hash per document URL and expose it to receivers — the [CORE] §3.5 dates-untrusted corroboration rule gains a concrete hook, and provenance/staleness display follows. No specific notary is mandated: first-seen is the minimal witness every directory already has. Directories exposing first-seen hashes SHOULD include them in API responses as a corroboration hint for offer validation — the cited-hash vs. first-seen-hash comparison gives commerce flows a concrete witness without mandating a notary. Non-normative field-name example for interop: `"first_seen_hash": "sha256:…"` beside the document URL in the response entry; one line, no API schema mandated.
- **Fuzzy dedup (MAY).** Receivers MAY fuzzy-dedup listing search results on normalized title + `ask` — NFC/NFD visual duplicates are distinct documents by canonical law ([TRUST] §4.1), and search is receiver territory, so dedup lives here, never in the canonical form.

## 4. Receiver posture (SHOULD)

Directories, agents, and renderer defaults SHOULD: rank L2-verified above L1 above L0; greylist unsigned flood volumes; weight the review graph; apply cost limits (stamps, admission fees) to unknown issuers. Trust is always *displayed* — badges, ranking, filters — never executed, and never hidden content removal. A receiver that ignores trust layers entirely is conformant; it is simply less useful, and the market answers that.

*Policy vs. soft-fail (clarification):* default feed ranking and filtering that omits low-trust documents is **receiver policy**, not renderer soft-fail hiding ([TRUST] §2) — the two are conformant together as long as the omitted document remains reachable by direct URL and any conforming renderer still displays it when fetched. Omission from a default feed is not hiding at render.

*Commerce default:* commerce-focused receivers SHOULD default unsigned listings out of default search ranking while still rendering them by direct URL (the policy-vs-soft-fail clarification above applies). The signatures-optional, edge-filtering-is-the-mechanism principle is untouched; this is that mechanism, stated as a sensible default.

**4.1 Sponsored-ranking disclosure (SHOULD).** A directory, matching engine, or other receiver that accepts payment — any rail — for ranking, placement, or offer delivery SHOULD disclose that fact to the reader (label, badge, or separate lane). Voluntary and unenforced: the norm exists so honest behavior is legible and standard, receivers can filter sponsored results on request ("ignore paid"), and clean services gain a credibility badge; dirty ones answer to Layer-3 reputation. Analogous to ad-labelling norms in search — written on day one, before anyone has incentives to fight it.

**4.2 Staleness decay.** The older a listing without an `updated` or `status` change, the less likely it remains available. Receivers MAY annotate possibly-stale listings and down-rank with age — display, never hide. `expires` makes the inference precise where present; its absence never blocks it. Issuers who maintain their documents (`updated`, `prev:` chains, re-signing on status change) rank visibly fresher — document hygiene becomes a trust signal.

*Dates caveat: `published`/`updated` are self-attested and MUST NOT be treated as evidence without corroboration (first-seen, notary stamp, checkpoint) — see [CORE] §3.5. Receivers use corroborated dates only.*

*Corroborated status: a status change corroborated by `prev:`-chain re-sign or directory first-seen of the changed document SHOULD rank above a bare `status` header edit — receivers weigh what they can verify. `sold` without a published receipt MAY be annotated as lower-corroboration ([CORE] §3.2 deal receipts). Default marketplace receivers SHOULD rank `sold` listings with a published bilateral deal receipt above `sold` by bare status header, and both above stale-dated `active` listings — expected hygiene made legible for matching engines and edge feed defaults; receipt publication itself is never mandated.*

*Reviewer-cost guidance: the review graph is only as strong as reviewer identity cost. Receivers weighting reviews SHOULD discount reviews from keys younger than the receiver's own first-seen history of them, and weight reviews from aged, L2-bound reviewer keys highest — a million fresh L1 keys signing glowing `subject-keyid` reviews for each other is the naive attack ([CORE] §5 binding stops unbound reviews; age + domain cost stops the army). Bilateral-signed deal receipts (future `deal` type, [CORE] §5) are the endgame: reviews anchored to completed, mutually-attested deals. Reference default for interoperable receivers (non-normative): do not aggregate reviews from reviewer `keyid`s first seen less than **30 days** before the review's `signed` date, unless the reviewer key is L2-bound — the exact window is receiver policy; publishing one reference default lets independent directories behave similarly.*

*Directory admission — proof of control: a directory that indexes by submitted URL SHOULD require proof of control before indexing via **[TRUST] §7.1 flow 1 (challenge file at the submitted URL's registrable domain) OR flow 2 (signed index-request document)**. Indexed documents will often also verify as L2-signed, but L2 verification of the *target* document is not proof of *submitter* control — a third party can sign nothing and still submit a victim's public listing; the §7.1 flows are the only admission proofs. The challenge-file path, nonce line format, and index-request flow are defined normatively in [TRUST] §7.1. Indexing arbitrary pasted URLs lets third parties index victims' documents or pollute search with unrelated content — proof-of-control closes the hijack without adding a permission layer (anyone who controls the URL passes; nobody who doesn't). The well-known-write-access honest limit also applies: path-only/CDN publishers cannot pass admission from those URLs ([TRUST] §7.1).*

---

*[CORE] 01-CORE — Core Format. [TRUST] 02-TRUST — Trust & Integrity.*
