# 01-CORE — intents.md Core Format

**Layer:** 1 (hardened floor) · **Status:** Normative

## 1. Status of this document

This document specifies the intents.md core format. The key words MAY, MUST, MUST NOT, OPTIONAL, RECOMMENDED, REQUIRED, SHALL, SHOULD, and SHOULD NOT are to be interpreted as described in RFC 2119.

Everything in this document is **normative** except where explicitly marked otherwise. Layers 2 and 3 are defined in companion documents ([TRUST], [SERVICES]). This split supersedes the earlier single-document layout.

## 2. Overview

An **Intent Document** is a single UTF-8 plain-text file served at an ordinary HTTPS URL, expressing what its issuer wants to offer, say, or ask. It rides on existing infrastructure — HTTPS transport, `text/markdown; charset=utf-8` media type — and requires no new ports, DNS records, registries, or accounts.

Recommended file naming: `<slug>.intents.md` for individual documents (an ordinary `.md` file; plain `.md` and any other filename are also valid). A bare `intents.md` at a site root is the conventional index — the site's or publisher's own intents, linking to its other documents. Naming is a discovery convenience only and carries no trust: a document named `intents.md` is not thereby trustworthy, and a valid document at any other filename is not thereby invalid.

**Personal documents (non-normative).** The same format works privately: an `intents.md` in a workspace or vault, read by its owner's own agents, expressing standing goals and current priorities. Trust is by possession — an agent that can read your filesystem is already trusted — so signatures are unnecessary there. Nothing in this specification changes for public documents.

A document has three parts, in order:

1. **Header block** — `name: value` lines, terminated by the first blank line.
2. **Body** — free-form plain text.
3. **Signature block** — OPTIONAL, defined in [TRUST].

**The core invariant:** there is nothing in a conforming document that any renderer could execute, even by accident. Publishers ship data. Readers render it with components they trust. The reader — not the publisher — is sovereign over rendering.

## 3. Header block

Format: one `name: value` per line. Names are lowercase; a single space follows the colon. Values are single-line; continuation lines (beginning with a space) are permitted for long values. The header block ends at the first blank line.

### 3.1 Core headers

| Header | Status | Rule |
|---|---|---|
| `intent` | **REQUIRED** | Version integer. This specification: `2`. |
| `type` | **REQUIRED** | A content type from §5, or any custom string. Unknown types MUST be rendered as `post`. |
| `title` | **REQUIRED** | One line of plain text. |
| `published` | RECOMMENDED | ISO 8601 date or datetime, UTC. |
| `updated` | RECOMMENDED | ISO 8601 date/datetime of the last material change (status, price, terms). |
| `expires` | RECOMMENDED | ISO 8601. After expiry, renderers SHOULD display "expired" but MUST NOT hide content. |
| `status` | OPTIONAL | `active` (default) \| `sold` \| `withdrawn`. Renderers SHOULD display non-active status prominently; content MUST remain visible. Issuers SHOULD re-sign on status change so receivers can corroborate it ([SERVICES] §4.2). |
| `lang` | OPTIONAL | ISO 639-1 code of the body's language. Multilingual issuers publish one document per language. |
| `issuer` | OPTIONAL | Display name of the publisher, free text. |
| `issuer-url` | OPTIONAL | HTTPS URL of the publisher's site; used for key binding ([TRUST] §3). |
| `id` | OPTIONAL | Stable identifier: UUID, slug, or URL. |
| `prev` | OPTIONAL | `sha256:…` fingerprint of the previous version's canonical form — hash chaining, defined in [TRUST] §5. MUST remain optional; standalone documents are always valid. |

### 3.1a Subject binding headers 

These headers bind reviews, offers, and any machine-readable reference to another document. They apply to `type: review`, negotiation citations, and any document or message that references another intent document. All OPTIONAL on the document; the aggregation rule in §5 governs what receivers do without them.

| Header | Status | Rule |
|---|---|---|
| `subject-keyid` | OPTIONAL | The `keyid` of the reviewed issuer's signing key. Required for aggregation when the subject is **signed** (signed-subject branch, §5); MUST NOT be required when the subject is unsigned. |
| `subject-url` | OPTIONAL | HTTPS URL of the subject document (the listing, rfp, or profile being reviewed). |
| `subject-hash` | OPTIONAL | `sha256:…` of the subject document's canonical form at review time — binds the review to the exact version seen, and is the only subject binding available when the subject is unsigned. |

Receivers MAY require any combination; publishers SHOULD provide `subject-url` and `subject-hash` always, plus `subject-keyid` when the subject is signed. A review of an unsigned subject binds by `subject-hash` only — keyid binding is impossible there by design.

**Negotiation citation.** Negotiation stays out-of-band (email or `contact:`). Any machine-readable offer, counter-offer, or negotiation message referencing a listing — whether or not it is published as an intent document — MUST cite the listing's document URL and canonical-form hash (`subject-url` + `subject-hash`). For plain-text threads (agent-drafted replies, ordinary email) the minimum citation block is two lines at the top of the message:

```
subject-url: https://example.org/listing.md
subject-hash: sha256:…
```

Offers published as intent documents use the §3.1a headers directly; the future public offer document type is reserved as `offer` (§5, alongside `deal`). An unsigned offer email carrying the citation block is a conformant *citation*, not a conforming *document* — the block travels; the envelope is optional.

**Citation mismatch handling.** If the fetched canonical hash differs from the cited `subject-hash`, the receiver MUST treat the offer as answering a stale version and MUST surface both hashes to the human before any reply is sent. If the URL fetch fails, the thread soft-fails with a visible "could not verify cited version" state. Agents MUST NOT auto-accept or auto-send on a mismatched or unverified citation. `subject-hash` in any citation — document headers or email blocks — MUST be the sha256 of the subject document's **canonical form** bytes ([TRUST] §4.1), computed exactly as signature verification computes it; hashing raw file content or non-canonical bytes produces a hash no conforming receiver can match. Agents and conforming negotiation tools MUST fetch and verify the cited URL before sending any reply; plain mail clients SHOULD display the cited hash as **unverified** until the reader's agent or the human verifies it out of band.

### 3.2 Commerce headers

Apply to `type: listing` and `type: rfp`. All OPTIONAL — a listing without `ask` is valid.

| Header | Status | Rule |
|---|---|---|
| `ask` | OPTIONAL | Number plus ISO currency code (e.g. `450 CHF`). Renderers MUST display verbatim; silent currency conversion is forbidden. A renderer MAY show a conversion if clearly labelled as its own calculation. |
| `negotiable` | OPTIONAL | `yes` \| `no`. Default: `no`. |
| `payment` | OPTIONAL | Comma-separated accepted rails, preference order, free text (e.g. `bank-transfer, escrow, x402-usdc`). Declares accepted rails only; payment itself happens out-of-band (§7). |
| `quantity` | OPTIONAL | Integer. Default: 1. |
| `condition` | OPTIONAL | Free text. Physical goods: e.g. `new`, `mint`, `lightly used`. Purely digital goods (a code repository, an app project, a domain): the honest value is `digital` — condition semantics do not apply, and stating so helps matching engines route digital vs physical. |
| `ships-from` | OPTIONAL | Free text (country or region). |
| `ships-to` | OPTIONAL | Free text (country or region). |
| `contact` | OPTIONAL | Email address or HTTPS URL where buyers or agents reach the issuer. |
| `floor` | OPTIONAL | Seller minimum. **Public information — see guidance before using.** A public floor is a quick-sale signal: it invites instant floor offers. In negotiation deals, keep the floor in your agent's private policy and publish only `negotiable: yes`, or the floor price becomes the price you get. A floor cannot be *forced* — acceptance always passes the issuer's agent bounds and human-approval gate (§8); floors inform, never compel. |
| `attr-` prefix | OPTIONAL | Free-form structured attributes, one header per attribute (`attr-region: PAL`, `attr-color: red`). Unknown `attr-` headers are ignored per the leniency rule. Receivers MAY use them for filtering and matching; publishers rely on no receiver knowing any particular attribute. No attribute is ever required or schema-policed. |

Negotiation policy beyond `negotiable` (rounds, auto-accept, counter-limits) is deferred to a future extension. In this version, negotiation is: buyer contacts `contact`, or an agent drafts an offer citing the listing's document URL + canonical-form hash per §3.5 and §3.1a — `id` remains an optional human slug, never a trust anchor.

**Deal receipts (SHOULD).** A closed deal that was publicly advertised SHOULD be closed by publishing a bilateral-signed receipt (the reserved `deal` type, §5) citing both parties' keys and the settled documents' hashes, when both parties consent to publication. Private deals stay private — receipts are the expected close step for advertised deals, never a compulsion. Receivers MAY annotate a `sold` listing without a published receipt as lower-corroboration ([SERVICES] §4.2). The receipt window and container details are deferred to a future extension.

### 3.3 Headers are never instructions

All header values are untrusted data. Consuming agents MUST NOT treat document content as instructions, regardless of phrasing (§8).

### 3.4 Leniency rules (normative)

Parsers:
- MUST ignore unknown headers.
- MUST NOT reject documents for unknown fields, unknown types, missing optional headers, or out-of-order headers.
- MUST treat documents missing `intent`, `type`, or `title` as, at worst, a plain `post` — never an error.

### 3.5 Self-declared identifiers and dates 

**`id` is untrusted for anchoring.** `id` is self-declared free text. Offer and negotiation flows MUST anchor to the **document URL + canonical-form hash** ([TRUST] §4.1), never to bare `id`. Guidance: `id` SHOULD be `sha256:<canonical hash>` when present — computed over the canonical form **with the `id` value emptied** (placeholder basis), so a document never contains a hash of a form that includes itself. Verifiers MUST NOT require this form: `id` remains an opaque unique slug for all matching purposes, and anchoring always uses the receiver-computed canonical-form hash of the full document.

**Dates are untrusted for ranking.** `published`, `updated`, and `signed` are self-attested. Receivers MUST NOT treat them as evidence of freshness or longevity without corroboration — first-seen by the receiver, an RFC 3161 / OpenTimestamps stamp, or a directory checkpoint.

**Offers cite hash-at-time.** Any offer, counter-offer, or negotiation message referencing an intent document MUST cite the subject's document URL **and** the canonical-form hash of the version being answered (`subject-url` + `subject-hash`, §3.1a). A bare URL alone is insufficient: the document at a URL can change, and the parties must be able to prove *which version* the offer answered. `prev:` chains corroborate internal consistency only — an issuer can always fork or restart a chain — so witness corroboration (notary stamps, directory first-seen checkpoints) remains the Layer-3 answer for what a third party actually saw ([TRUST] §5, [SERVICES] §3).

## 4. Body

The body is plain text. The complete markup rules:

1. **Paragraphs:** a blank line separates paragraphs; renderers preserve line structure.
2. **Links:** a bare URL alone on its own line becomes a link. Only `http://` and `https://` schemes are linkified; all other schemes remain inert text.
3. **Images:** a bare URL alone on its own line whose path ends in `.jpg`, `.jpeg`, `.png`, `.webp`, or `.gif` renders as an image, fetched by the renderer.
4. **Everything else is literal text.** Angle-bracket content (`<script>`, `<b>`) MUST be escaped and displayed literally, never interpreted. There is no raw HTML passthrough, no stylesheets, no iframes, no scripts, no fonts, no remote includes of any kind.
5. **No executable content can exist.** Renderers that add interpretation beyond rules 1–3 are non-conformant.

Optional sugar (non-normative): renderers MAY style a first-line `#` prefix or `*emphasis*`, but publishers SHOULD NOT rely on it; the plain-text reading must always be complete and natural. Blessing markup subsets happens only by community consensus after demonstrated demand.

**Limits (SHOULD):** documents SHOULD stay under 1 MB total; headers under 64 KB. Media is referenced by URL, never embedded. Renderers MAY truncate larger documents with a visible notice; agents MAY decline to fetch unbounded documents.

**Media fetch policy.** Renderers fetching images per rule 3 MUST conform to the media-fetch policy in [TRUST] §9 item 3 — magic-byte validation, blocked destinations, and size/decode caps. SVG is not an image type in this version.

## 5. Content types

| Type | Meaning | Notes |
|---|---|---|
| `listing` | Something offered for sale | Uses the §3.2 commerce headers |
| `post` | Article, update, statement | Body is the whole point; also the fallback for unknown types |
| `rfp` | Request for something (buy/hire/rent) | Reverse listing; demand-side twin of `listing` |
| `profile` | Identity and pointers to other documents | Reputation anchor |
| `review` | Attestation bound to another document via the §5 two-branch rule: signed subjects — signed review + `subject-keyid` + `subject-hash`; unsigned subjects — **signed review** with `subject-hash` only (§3.1a; `id` is display-only) | Reputation as documents — the protocol's answer to "signatures don't make people honest" |
| `feed` | Ordered pointers to other intent documents | The RSS-again moment |

All types share the envelope, body rules, signature model, and rendering tiers. Templates per type are guidelines (SHOULD), never validation gates. The type registry is open: new types are added by community consensus after demonstrated usage.

**Review binding.** Two explicit branches govern whether a `review` document counts as reputation input. **Signed subject:** the review is itself signed, carries `subject-keyid:` matching the reviewed issuer's `keyid`, and satisfies the aggregation tightening below (subject-hash match when the receiver can fetch the subject). A `subject-keyid` that mismatches the fetched subject's actual signing `keyid` MUST NOT aggregate — even if `subject-hash` matches. **Unsigned subject:** the review MUST itself be signed — binding is `subject-hash` only (§3.1a), because the subject document has no signing key; `subject-keyid` MUST NOT be required, and any `subject-keyid` header present is ignored for aggregation — branch selection governs which rules apply. Reviewer identity cost still applies ([SERVICES] §4 reviewer-cost guidance). Unsigned or unbound reviews render fine — they simply do not aggregate.

**Branch selection (normative).** The receiver classifies the subject by fetching `subject-url` when available. Signed subject = the fetched subject carries a signature block that verifies ([TRUST] §2 soft-fail rule). Unsigned subject = no verifiable signature on the fetched subject. On fetch failure, the review does not aggregate until the subject class is known; receivers SHOULD surface a visible "could not verify subject" state (same posture as the §3.1a fetch-failure rule, stated here for reviews rather than by cross-reference).

**Aggregation tightening.** For aggregation, a review MUST satisfy the signed-subject or unsigned-subject branch above. Shared tail, both branches: a review missing `subject-hash` where the receiver could verify it renders fine but MUST NOT aggregate into reputation scores. Receivers SHOULD NOT aggregate reviews from reviewer keys younger than the receiver's first-seen threshold or lacking L2-bound reviewer identity ([SERVICES] §4 reviewer-cost guidance) — a fresh-key army praising each other's `keyid` is the naive attack; reviewer age and domain cost are the counter.

**Type templates (SHOULD).** Minimal body shapes per hollow type; guidelines, never validation gates:

- `feed` — exactly two meaningful line classes: absolute HTTPS URL lines (pointers) and `#`-prefixed lines (comments and titles — titles SHOULD be `# My Feed Title`). Any other line is ignored prose, never a pointer. (Pointer format only; ordering semantics are the consumer's.)
- `profile` — body is a short self-description in plain text; structured facts travel as `attr-` headers (`attr-location: Zürich`). A profile is the natural place to publish `issuer-url` for L2 binding.
- `review` — body is free-text assessment; SHOULD carry `subject-url` and `subject-hash` always, plus `subject-keyid` when the subject is signed (§3.1a); a numeric rating, if used, travels as `attr-rating: 1-5` (receiver-interpreted, never schema-policed).
- `deal` (future extension) — reserved: a bilateral-signed receipt container citing both parties' keys and the settled documents' hashes. The negotiation itself stays out-of-band by design; only the receipt standardizes. Recorded here so edge apps build toward the right shape from day one.
- `offer` (future extension) — reserved: the public offer/counter document type, carrying the §3.1a subject headers. Until standardized, offers live out-of-band with the §3.1a citation block; the reservation prevents namespace drift.
- `index-request` (edge, defined [TRUST] §7.1) — reserved: signed directory submission document; `subject-url` points at the document to index; proof-of-control via the [TRUST] §7.1 flows. Minimal template: `intent`/`type`/`title` REQUIRED (core envelope as any document); `subject-url` = the document URL to index; `issuer-url` REQUIRED, its registrable domain MUST equal the registrable domain of `subject-url`; a signature block REQUIRED, verified per [TRUST] §7.1 flow 2 (key fetched at the `subject-url` registrable domain, fingerprint matching `keyid`). Directories MAY require more; never less than this.

## 6. Rendering conformance

Three tiers; every tier is strictly safer than a normal web page because no publisher code exists in the channel:

- **Tier 0 — Any reader (always works):** open the file. It reads top to bottom in any editor, terminal, or `cat`. Zero fidelity loss of information; zero risk.
- **Tier 1 — Browser renderer:** a static page fetches the document and renders it as clean HTML/CSS per §4. No publisher code runs, because none exists.
- **Tier 2 — Agent-native:** an agent parses headers, renders in its own UI, and may act on structured fields — surface an offer, draft a reply, schedule expiry — behind its normal human-approval gates (§8).

**Adoption rule (normative):** agent-better, not agent-only. Every conforming document MUST remain fully readable by a plain browser (Tiers 0–1) and become better when rendered by an agent (Tier 2).

**Growth ladder for richer media (non-normative guidance):** as the protocol grows beyond text and images, pre-render server-side and ship pixels, or declarative formats decoded inside sandboxed native players. Capability expands only by adding declarative item types — never by opening script or network gates.

Renderer conformance: implement §4 rules; never execute; never silently convert currency; soft-fail signatures ([TRUST] §2); ignore unknown everything.

**Reference renderer guidance (non-normative):**
- The reference renderer should ship before third-party renderers exist, and should be safe by default: no raw HTML passthrough, hostile-document test vectors passing, embeddable as a library. The path of least resistance for implementers should be the safe path — a no-execution norm holds in practice through tooling, not paper.
- Renderer implementations are edge nodes. A renderer that executes document content is non-conformant (§4) and answerable to receiver reputation ([SERVICES] §4) — flagged, down-ranked, routed around.
- Tier 0 remains the unconditional floor: opening a document as raw text in any browser is always safe — browsers display plain text; they do not execute markdown. Execution risk exists only in renderers that add interpretation, which §4 binds and the test vectors enforce.

### 6.1 Render integrity: the interpretation boundary 

**Principle:** the rendered document is the ledger; the agent is the analyst. The analyst may report on the ledger, never edit it.

1. **Ground truth.** The document view is the ground-truth representation of what the publisher said. Conforming renderers MUST render document content deterministically per §4: the same document yields the same content for every reader. Styling MAY vary; content MUST NOT.
2. **Interpretation is separate.** Any transformation of document content — summary, translation, comparison, recommendation, "what this means for you" — is **interpretation, not document**. Interpretation MUST be presented in visually and structurally distinct chrome, outside the document view, and MUST carry an explicit provenance label (e.g. "agent summary — not the publisher's words").
3. **No interleaving, no silent rewrite.** Conforming renderers MUST NOT display AI-generated text interleaved into the document body, and MUST NOT paraphrase or rewrite body text inside the document view. Quote exactly inside the view, or interpret outside it — never both at once.
4. **Attribution integrity.** No conforming renderer may display generated text in a way a reasonable reader could attribute to the issuer, nor publisher text in a way that could be mistaken for renderer commentary. Misattribution in either direction is non-conformant.

*Rationale (non-normative): every website is a guess about what the reader wants — thousands of designers (and now AIs) solving the same presentation problem, still one-size-fits-all. The intent document stops guessing: ship the data once, the reader's side decides what it becomes. Today's personalization is server-side and paid for with surveillance; reader-side rendering delivers personalization without surveillance — the publisher learns nothing about the render. Render is the ledger; the agent is the analyst.*

**Conformance.** Render integrity is a conformance class: a Tier-2 renderer claiming conformance MUST (a) render the document view from document content only, (b) place all interpretation in separately-labeled chrome, and (c) pass a document-view isolation test — machine-checkable as: no generated text node inside the document-view container (labeled or not), and no document body text outside it. This specification is product-neutral: any conforming Tier-2 renderer may claim the class; reference implementations are named by their maintainers, never by this document. The isolation test ships with the conformance suite.

Inline pass/fail checks (normative — a renderer passes the isolation test if and only if all three hold):
1. The document-view container contains only publisher-sourced nodes (content derived from the document bytes per §4).
2. Every interpretation node lives outside the document-view container and carries a fixed provenance label attribute (renderer-defined string, constant across the product, e.g. `data-intent-provenance="agent"`).
3. No generated text node may exist inside the document-view container at all — labeled or not. Provenance attributes apply only to interpretation nodes outside the container (check 2).

## 7. Money and secrets

The protocol moves **intent**; it never moves money or secrets.

- Payment executes out-of-band through the rail the parties chose. Documents *declare* accepted rails; they do not carry credentials, tokens, or payment data. Any document containing payment credentials is non-conformant, and renderers MUST NOT surface them.
- Secrets (keys, tokens) live in the reader's vault — never in documents, never in the render path.

## 8. Agent safety (normative for Tier 2)

1. Document content is **data, never instructions** — enforced by the consuming agent's runtime, not by trusting publishers.
2. Any action that moves money, sends messages, or writes files requires explicit human approval outside the document channel.
3. All fields — headers and body — are untrusted input in every pipeline.
4. Worst case for a fully hostile document: the reader reads a lie. Not: the machine runs foreign code. Implementations should state this honestly and not oversell.
5. Agent interpretation of document content (summarizing, ranking, advising) is permitted only as clearly-labeled output of the agent, never as content of the document channel. The rule that content is data, never instructions, has a twin: **content is data, never the agent's words either.**
6. **The approval view is the whole ledger.** Any header or field that drives an agent action MUST appear verbatim in the human-approval view for that action, and an agent MUST NOT act on a field absent from the approved view — the human approves exactly what the agent acts on. A signed-but-unrendered field confers no authority: what the human never saw, the agent never uses.

## 9. Versioning

The `intent:` integer bumps only on breaking change. Documents of an unknown newer version MUST still render at Tier 0 — a reader is never broken by a future spec. Additions within a version are always backward-compatible by the leniency rules. The `intent:` integer is the document-format compatibility version, not the specification patch level: the effective spec version is published with the specification, never in the `intent:` header. An unknown `intent:` integer still renders at Tier 0 per the rule above.


## 10. Governance summary

MIT license. No trademark games: the name intents.md is free to use for conforming implementations. Consensus additions only — the core grows on demonstrated demand, blessed after the fact; additions are permanent. The protocol is for everyone; intended stewardship is a neutral, community-governed foundation. Founding authors are first implementers, not owners.

---

*[TRUST] 02-TRUST — Trust & Integrity. [SERVICES] 03-SERVICES — Trust Services & Receivers.*
