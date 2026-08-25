# 02-TRUST — intents.md Trust & Integrity

**Layer:** 2 (standardized hooks, optional practice) · **Status:** Normative

## 1. Status of this document

This document specifies the intents.md trust and integrity mechanisms. RFC 2119 keywords apply as defined therein. The *mechanisms* are normative; their *use* is optional. Layer 2 exists so that trust services ([SERVICES]) compose across implementations instead of siloing in per-product cages — the hooks are standardized here, from launch, so they never need retrofitting.

Trust decisions — ranking, filtering, weighing — are always the receiver's ([SERVICES] §4). The core never adjudicates trust.

## 2. Trust levels

| Level | What | Gives the reader |
|---|---|---|
| **L0 — Anonymous** | No signature | Content only. Valid for posts and casual listings. |
| **L1 — Self-signed** | Same key across documents | Persistent pseudonymous identity; reputation over time; tamper-evidence — any edit breaks the signature. |
| **L2 — Domain-bound** | Key fetched by the **verifier** from the registrable domain's `/.well-known/intents-md/key.asc` (§2.1) | Cryptographic binding to a domain the reader can check. A spoofed "issuer: Example Shop" document without Example Shop's key at its own domain fails verification. |

**Soft-fail rule (normative):** an invalid or unverifiable signature MUST NOT hide a document. Renderers display a visible "signature invalid / unverified" state; content still renders. Trust is information for the reader — never an execution decision. Any receiver MAY additionally filter by signature state ([SERVICES] §4); hiding at the renderer's display layer is forbidden, filtering at the receiver's policy layer is sovereign.

**Downgrade rule (normative).** Signature state is tracked per issuer identity — the verifier-computed key fingerprint (§3.1 `keyid`), and the registrable domain for L2. A later document from an identity previously seen signing that arrives unsigned, or whose signature fails verification, is a **downgrade event**: renderers MUST display a visible downgrade state (never a silent return to L0), and receivers SHOULD suppress or down-rank the document pending re-verification. Soft-fail still governs display — content renders — but a downgrade is never presented as the same trust level the issuer previously held. Stripping a signature block to shed a bad history is thus visible and counterproductive; rotating to a *new* identity resets nothing a receiver keys on domain or computed fingerprint.

### 2.1 L2 binding (normative — governs the L2 row above)

A document is **L2-verified** only when ALL of the following hold:

1. It carries a signature block that verifies (§3).
2. The verifier derives the **registrable domain** (eTLD+1 per the Mozilla Public Suffix List, as currently published) from the document's `issuer-url`, and fetches the key itself at `https://<registrable-domain>/.well-known/intents-md/key.asc`. The verifier MUST NOT fetch the raw `key:` URL from the signature block; that URL is advisory display metadata only.
3. The fetched key's fingerprint matches the document's `keyid`.
4. **Fetch basis.** The verifier's key fetch uses exactly `https://<registrable-domain>/.well-known/intents-md/key.asc` — scheme `https`, default port, fixed path. Any port, path, query, or fragment present in `issuer-url` MUST be ignored for this fetch; an `issuer-url` cannot steer the fetch to an attacker-chosen endpoint.
5. **No issuer-url, no L2.** A document without an `issuer-url` cannot attempt L2 verification; display falls back to the signature state (L0/L1). Soft-fail, never a rejection.

**Binding is to the domain, never to the name.** L2 binds a document to a registrable domain. It does NOT bind the human-readable `issuer` name, and MUST NOT be presented as doing so. Renderers displaying L2 state MUST display the registrable domain (e.g. "Signed · alex.example") — never a bare checkmark and never the self-declared `issuer` name alone.

**Free-host cap.** If the registrable domain is itself on the Public Suffix List or a suffix of a PSL entry (e.g. `github.io`, `pages.dev`), the hostname is a platform tenancy, not an owned identity: verifiers MUST treat keys served there as **L1-equivalent** for all ranking and badge purposes. The domain-cost sybil defense applies only to domains an issuer actually had to register.

**Tenancy test (defines the operative test where the cap sentence above is ambiguous).** A hostname is platform tenancy when the prevailing public suffix for it comes from the **PRIVATE section** of the Public Suffix List (e.g. `github.io`, `pages.dev`, `vercel.app`, `netlify.app`, `neocities.org`). ICANN-section suffixes (`ch`, `co.uk`, …) are registry TLDs: registrable domains on them cost registry money and earn full L2. This reading covers the dominant `user.github.io` form — where the eTLD+1 (`user.github.io`) is itself free tenancy — and not only bare path-style publishers on a platform apex.

**PSL timing — L2 is a verifier-local judgment.** L2 status is computed at fetch time against the PSL snapshot the verifier currently holds; two verifiers MAY disagree, and a hostname's status MAY change when the PSL changes. Verifiers SHOULD pin a PSL snapshot and refresh on a documented cadence. This is consistent with receiver sovereignty ([SERVICES] §4): L2 is a badge a receiver grants, not a property a document carries.

**Honest cost of the cap (owned trade).** Free-host publishers who could have displayed "L2" under the old self-declared rule are demoted to L1-equivalent. That is deliberate: the domain-cost sybil rung must price identities at registry cost, and free tenancy prices them at zero. The path to L2 is registering a domain — a low-single-digit-CHF-per-year action — and everything else about zero-config static publishing in this spec is unchanged.

**PSL freshness gap.** The PSL lags reality: a new free-host platform is full-L2 until its suffix reaches the PSL PRIVATE section, and verifiers on different snapshot cadences MAY disagree in that window. Verifiers SHOULD therefore maintain a local denylist of known free-tenancy suffixes alongside the PSL and apply the same L1-equivalent cap to them; the PSL remains the normative source, the denylist only closes the lag. Known per-path multi-tenant hosts — unrelated publishers sharing one registrable domain by URL path — belong on the same denylist and carry the same cap. (Directory-grade verifiers SHOULD publish their denylist + refresh cadence — disagreement is honest when it is legible.)

**Lookalike honesty (non-normative note).** L2 still cannot prevent `alex-shop.ch` from impersonating `alex.ch` visually. The domain-display requirement above is the mitigation: the reader always sees which domain vouches, so lookalikes compete on legibility, not on crypto badges. Receivers MAY add an edit-distance UX nudge — visually highlighting when the issuer display name and the registrable domain diverge sharply (issuer `alex` over `alex-shop.ch`) — keeping L2 honest without overselling it as identity proof. In one line: **L2 proves a domain vouches; it does not prove human identity** — `alex-shop.ch`'s vouching is worth exactly what `alex-shop.ch` is worth, and lookalike domains remain a reader-literacy problem no cryptography in this spec solves. Launch copy MUST NOT oversell L2 as identity proof.

## 3. Signature block

A document MAY end with a detached signature block:

```
-----BEGIN INTENT SIGNATURE-----
algo: ed25519
key: https://issuer.example/.well-known/intents-md/key.asc
keyid: sha256:9f2a…c41b
signed: 2026-08-21T08:00:00Z
<base64 signature over the canonical form>
-----END INTENT SIGNATURE-----
```

### 3.1 Fields

| Field | Status | Rule |
|---|---|---|
| `algo` | **REQUIRED** | Algorithm identifier (e.g. `ed25519`, `pgp`). The specification never mandates *which* algorithm — only that it is declared. A block with no `algo` is treated as absent; the document displays as L0 unsigned. |
| `key` | RECOMMENDED | URL of the public key — the L2 domain-bound path. |
| `keyid` | RECOMMENDED | Fingerprint of the signing key. The *portable* identity: survives key-URL death and key rotation, and lets L1 reputation track an issuer with no domain at all. **The asserted string is a hint, never an identity:** verifiers MUST compute the fingerprint of the key that actually verifies (or that was fetched at the well-known path) and key all reputation, aggregation, and review-binding on the **computed** fingerprint — an asserted `keyid` alone never establishes identity (copying a reputable issuer's `keyid` onto an unsigned or differently-signed document verifies nothing). |
| `signed` | **REQUIRED** | Timestamp of signing (RFC 3339). A signature block without `signed` is treated as absent; the document displays as L0 unsigned. |
| (others) | OPTIONAL | Unknown fields inside the block are ignored, per the global leniency rule. |

### 3.2 Algorithm agility (normative)

**Freshness (normative).** `signed` bounds staleness, not forgery: it is block metadata, self-attested (§4.1 rule 1 — outside the signed range). Receivers SHOULD apply a max-age freshness window to `signed` (the window is receiver policy; directory-grade receivers publish theirs), and issuers of time-sensitive documents SHOULD set an `expires` header so staleness is declarative. A document whose signature verifies but whose `signed` is older than the receiver's window displays as verified-but-stale — visible state, never silent acceptance of an eternal claim.

Verifiers MUST NOT reject a document because its `algo` is unknown to them; they display it as unverified ("unknown algorithm") with content still rendering. Hard-coding a single algorithm into a renderer is non-conformant. The specification mandates declared algorithms, never chosen ones.

### 3.3 Immutability by signature

The cryptography is self-enforcing: any edit to the document (or the block) breaks verification and surfaces as soft-fail. A signed document is therefore immutable-by-signature. To amend, the issuer re-signs and republishes; versioning over time is new documents (or the same URL re-signed), never a mutated signed artifact. Amendment is always possible; *silent* amendment is impossible.

## 4. Canonical form (for signing and hashing)

- All header lines normalized to `name: value`: lowercase name, single space, template order — `intent, type, title, published, updated, expires, status, lang, issuer, issuer-url, id, prev` — then extension headers alphabetically. (Exact byte-level rules, including byte-exact values, are §4.1 — §4.1 governs wherever the two diverge.)
- One blank line.
- Then the body byte-exact as written. (Value byte-exactness per §4.1 rule 9: the canonical form preserves the value bytes after the `name:` prefix — `title:X` and `title: X` are distinct documents; §4.1 governs.)
- **Line endings normalized to LF (CRLF → LF) across the entire document before signing and before verification** — a round-trip through a Windows editor or mail client must never break a signature.

### 4.1 Exact canonical form (normative — governs over the §4 summary above wherever the two diverge)

The canonical form is a byte sequence built by the following rules, applied in order. Every conforming signer and verifier MUST produce identical bytes.

1. **Signed range.** The canonical form covers the header block and the body. The signature block (§3) is outside the signed range — a detached signature cannot cover itself. Block fields (`signed:`, `key:`, …) are self-attested metadata and MUST be displayed as such.
2. **BOM.** A leading U+FEFF, if present, MUST be stripped.
3. **Line endings.** CRLF → LF throughout.
4. **Trailing whitespace.** Trailing spaces/tabs on each line MUST be stripped.
5. **Duplicates.** If a header name appears more than once, the **last** occurrence wins and earlier duplicates are removed. Renderers MUST display the winning occurrence — the signed value and the displayed value are always the same occurrence (no sign/display divergence).
6. **Continuation lines.** Unfolded before signing: each continuation line (leading space or tab) is joined to its parent with a single space. The canonical form contains no continuation lines.
7. **Header order.** Core headers first, in exactly this order, each at most once after rule 5:
   `intent, type, title, published, updated, expires, status, lang, issuer, issuer-url, id, prev`
   then **all** remaining headers — extension headers (`ask`, `contact`, …), `attr-` headers, and unknown headers alike — in ascending **byte-wise ASCII order of the header name** (raw byte comparison, no locale, no case folding; all names are already lowercase).
8. **Unknown headers are signed.** No header line in the header block is ever dropped from the canonical form. "Ignored" ([CORE] §3.4) means *not interpreted*, never *excluded from signing*. The header block ends at the first blank line (the separator) or at the first line that is neither a continuation nor a valid `name:` header — whichever comes first; from that line the body begins, and no separator is emitted when the boundary was a non-header line. Everything after the boundary is body bytes, signed byte-exact (rule 10). This is the conventional-parse reading: a reader of the raw text sees body where the canonical form puts body.
9. **Value normalization: none.** Values are byte-exact after unfolding. No case folding and no Unicode normalization — NFC and NFD variants are simply different documents (both valid; never conflated, never silently unified). The value is the bytes after the **first** colon of the unfolded line (`contact: mailto:…` is one header, one value — a value may itself contain colons), and no whitespace normalization applies: `title: X` and `title:X` are different documents.
10. **Separator and body.** One blank line, then the body byte-exact subject to rules 2–4. Trailing blank lines at end of body are removed; the canonical form ends with exactly one LF.
11. **Header-name case.** Header names are matched case-insensitively and MUST be written lowercase in the canonical form. `Title:` and `title:` are the same header; a case-variant duplicate collapses under rule 5 (last occurrence wins). Sorting (rule 7) applies to the lowercased name. This normalizes names only — values remain byte-exact (rule 9, unchanged). Lowercasing is ASCII-only (0x41–0x5A → 0x61–0x7A): non-ASCII bytes in a name are never case-folded, so no Unicode locale can split two implementations' results. Header names SHOULD match `^[a-z0-9-]+$`.
12. **Execution pipeline (governs ordering).** The rules above are the inventory; execution order is: BOM strip (2) → line-ending normalization (3) → unfolding (6) → trailing-whitespace strip (4) → duplicate collapse, last-wins (5) → name lowercasing (11) → header ordering (7–8) → separator + body assembly (10). Where this pipeline and the rule numbering differ, the pipeline governs: unfolding precedes trailing-whitespace stripping, so a whitespace-only continuation line joins to its parent and contributes no residue (any resulting trailing space is stripped in the next stage).
    duplicate detection under rule 5 is by **lowercased** name — rule 11 makes `Title:` and `title:` the same header — so case-variants collapse last-wins before the lowercase rewrite.
13. **Empty body.** A document with an empty (or absent) body canonicalizes as: header block, the separator blank line, and a single terminating LF. The separator is retained so headers-only documents remain structurally valid: the canonical bytes end with the final header line's LF followed by exactly one blank-line LF (`…last-header\n\n`) and nothing more — no body LF, no extra terminator.
14. **Closed core list.** The core header sequence in rule 7 is CLOSED as of this version. Headers admitted later by consensus always sort among the extension/unknown set by byte order — never inserted into the core sequence. Canonical forms of existing documents therefore remain byte-stable across future spec versions; a version bump can never silently break an existing chain.

**Conformance test cases (normative — a machine-readable suite ships with the reference implementation at the ratification gate):** CRLF document verifies · BOM document verifies · duplicate `title:` displays and signs last-wins · continuation-line document folds identically across tools · NFC vs NFD bodies are distinct valid documents · a 3-document `prev:` chain links under every implementation · unknown-header document signs with the unknown header included · trailing-whitespace edit does not break signature · document with `prev:` sorts it after `id`, before extension headers · case-variant names (`Title:` / `title:` mix) canonicalize identically across tools · stacked continuation lines (two or more) fold left-to-right into one line identically across tools · whitespace-only continuation line folds and strips cleanly · empty-body document verifies and displays · future consensus header (e.g. `zz-future:`) sorts after core bytewise, byte-stable across versions.

*Authoring guidance (non-normative): publish NFC.* Canonical verification stays byte-exact (rule 9 — decided trade), but authors SHOULD normalize text to NFC at authoring time; publishing NFD invites visually-duplicate search entries at receivers that do not fuzzy-dedup ([SERVICES] §3).

## 5. Hash chaining (`prev:`)

A document MAY declare `prev: sha256:…` — the fingerprint of the previous version's canonical form. This is the Git/Merkle model without any coin.

What it buys:
- **Provable revision history** — a price change is demonstrably the same listing evolving, not a rewrite. Kills stealth-edit scams.
- **Provable longevity** — an unbroken chain of signed documents is a dated, ordered history; old keys become worth protecting.
- **Feed integrity** — a `feed` document chaining to its prior editions gets integrity for free.
- **Works unsigned** — hashing is not signing; L0 issuers still get sequence tamper-evidence. Chains are even more universal than signatures.

Honest limit: chains are self-attested. An issuer can always start a fresh history; a chain proves internal consistency, not completeness. Completeness is answered at Layer 3 — timestamping and witness notaries (RFC 3161 / OpenTimestamps style) and directory checkpoints ([SERVICES] §3).

`prev:` MUST remain optional; standalone documents are always valid.

## 6. Key custody (SHOULD)

Signing keys SHOULD live in a hardware-backed secrets vault under the issuer's control, never in plaintext on the publishing machine. Signing is one well-scoped operation — `signature = sign(canonical form)` — which a vault can perform without ever exposing the key itself. A hardware-backed vault is a trusted execution environment (TEE) in the practical sense — Secure Enclave, HSM, secure element — and is the recommended custody anchor for L1/L2 issuers. The strongest posture: the key exists in hardware and is never extractable at all.

## 7. Revocation (known limitation, deferred)

This version defines no *mandatory* revocation feed — no key revocation mechanism is required of any issuer. Optional `manifest.json` at the sibling well-known path provides the recommended revocation and rotation upgrade path below; if a key leaks before that path is adopted, L2 issuers mitigate by rotating the key file at their well-known URL: new documents verify against the new key, and old ones surface a visible key mismatch that readers judge for themselves. A full revocation design remains deferred; the manifest below is the bridge.

**Key manifest (RECOMMENDED upgrade path).** The well-known URL MAY serve a manifest listing current AND retired keys with validity windows + revocation dates. Verifiers check `signed` against the window; documents signed by a revoked key display "signed by revoked key". Full design still deferred, but the file format reserves this shape now so rotation isn't an identity reset later.

**Path harmonization.** The required well-known path remains `/.well-known/intents-md/key.asc` (§2.1 item 2 — single key, current shape). The manifest is OPTIONAL and lives at a sibling path, `/.well-known/intents-md/manifest.json`; verifiers MUST accept bare `key.asc` and MAY consult the manifest when present. This reserves the rotation upgrade without moving the required path — no later adoption can invalidate existing key.asc deployments.

**Manifest schema (RECOMMENDED shape).** When served, `manifest.json` SHOULD conform to:

```json
{
  "manifest": 1,
  "keys": [
    {
      "keyid": "sha256:…",
      "key": "-----BEGIN PGP PUBLIC KEY BLOCK-----…",
      "valid-from": "2026-08-01T00:00:00Z",
      "valid-until": null,
      "revoked": null
    }
  ]
}
```

Semantics: `keyid` = fingerprint as used in signature blocks; `valid-from`/`valid-until` bound the signing window — verifiers checking `signed` against the window display out-of-window signatures as "signed outside key validity"; `revoked` (ISO 8601, `null` when not revoked) marks the cutoff — documents with `signed` after `revoked` display "signed by revoked key". Unknown fields are ignored per the leniency rule. Serving a manifest is OPTIONAL; when absent, bare `key.asc` semantics apply unchanged.

### 7.1 Directory proof-of-control challenge 

Directory admission ([SERVICES] §4) needs an interoperable proof. Two flows, either satisfies:

1. **Challenge file.** The directory issues an opaque nonce (printable ASCII, ≤ 128 characters, single-use, validity window stated by the directory). The publisher serves it at `/.well-known/intents-md/challenge.txt` containing exactly one line: `intents-challenge: <nonce>`. The directory fetches once over HTTPS at the registrable domain **of the submitted document URL** — the document being indexed, never the directory's own domain and never a sidecar submission's headers — under the §2.1/§9 fetch basis and policy (fixed path, default port, no redirects beyond §9 caps). One verified fetch = control proven.
2. **Signed index request.** Alternatively, a signed intent document of reserved type `index-request` ([CORE] §5) carrying `subject-url` (the document to index) and `issuer-url` (**REQUIRED** for `index-request` per [CORE] §5); the two registrable domains MUST agree. The signature is verified per §2.1 items 1 and 3–5 against the **`subject-url` registrable domain** — the key is fetched at that domain's well-known path and its fingerprint must match the document's `keyid`: proof of control of the exact domain the indexed document lives on.

*Submission transport (non-normative interop note):* the typical submission is the signed `index-request` document bytes carried in the directory's admission API request body alongside the `subject-url`; directories verify per this flow before indexing. No specific API schema is mandated.

Both proofs bind control of the registrable domain. Nonce values are directory-issued and never interpreted by the publisher; the challenge file is deleted (or left to expire) after verification.

**Honest limit — well-known write access.** Both §7.1 flows require write access to the registrable domain's `/.well-known/intents-md/` tree. Publishers on path-only hosts (`github.io/<repo>`), raw CDN URLs, or corporate sites where another team owns the apex **cannot pass directory admission from those URLs**. Their documents remain fully valid for Tier-0/Tier-1 reading — proof-of-control gates directory admission, never document validity. The paths to admission: host the document at a URL on a domain whose well-known tree the publisher controls (a registrable domain is a low-single-digit-CHF-per-year action), or relocate to a convenience host that itself passes proof-of-control. Cross-domain vouching (a key on domain A admitting documents at domain B) is deliberately not defined here — weaker binding, needs its own threat model; recorded as a possible future extension ([SERVICES] §4).

**Consultation & display.** A verifier that displays L2 state SHOULD consult `manifest.json` when present (publishing it remains OPTIONAL). A verifier claiming L2 conformance that does not consult an available manifest MUST NOT display revoked or out-of-window keys as current. Exact display strings: `signed by revoked key` (documents with `signed` after the key's `revoked` date) and `signed outside key validity window` (`signed` outside `valid-from`/`valid-until`).

**Apply-if-obtained is MUST (clarification).** The SHOULD above governs *fetching* the manifest; *applying* it is stronger — a verifier that has obtained a manifest (fetched it, or holds it from any source) MUST NOT display a key as current L2 while that manifest marks it revoked or out-of-window. Fetching may stay SHOULD so day-one zero-config verifiers are conformant; ignoring a known revocation never is.

**Rotation playbook (minimum response to key compromise).** (1) Immediately replace `key.asc` at the well-known URL with the new key; (2) serve a manifest marking the old `keyid` `revoked` with the compromise date; (3) re-publish current documents signed by the new key (`prev:`-chained where a chain exists); (4) old documents signed by the revoked key then display `signed by revoked key` at conforming verifiers — visible history, not silent invalidation.

## 8. Phishing resistance

A fake page cannot mimic a login form it never renders. The reader's own renderer draws everything, and credentials live in the reader's vault — never in the document channel. Spoofed identity is handled by signatures (this document), not by mimicry. What no signature can give: honesty. A signed document can still lie about condition; countermeasures live at the deal layer — escrow rails and `review` documents, which are themselves intent documents.

---

## 9. Verifier & renderer fetch policy 

Any conforming implementation that fetches keys, documents, or media referenced by intent documents:

1. **Key fetches.** Only from the derived registrable-domain well-known path (§2.1). MUST refuse: non-HTTPS, IP-literal hosts, private/loopback/link-local destinations (RFC 1918, 127/8, 169.254/16, ::1, fc00::/7, fe80::/10), userinfo in URL, more than 1 redirect. MUST cap: 64 KB response, 10 s timeout. SHOULD cache by `keyid` ≥ 24 h. MUST require fetched-key fingerprint = `keyid` when present. SHOULD rate-limit key fetches per registrable domain (amplification defense: attacker docs cannot order the verifier's network activity).
2. **Document fetches** (renderers, directories, agents): same destination rules; MUST cap size (spec limit 1 MB) and time; MUST apply per-domain crawl budgets; redirects capped at 3.
3. **Media fetches** ([CORE] §4 rule 3): renderer MUST validate **magic bytes** (JPEG/PNG/WebP/GIF raster signatures) and render non-raster bytes — including anything served as SVG — as inert text. SVG is not an image type in this version. MUST block private/loopback/link-local destinations; SHOULD cap size (≤ 10 MB) and decode time (decompression-bomb defense); SHOULD cache.
4. **Privacy disclosure (SHOULD).** Publishers learn the receiver's IP/UA when media renders. Receivers SHOULD document this, and MAY render media only on explicit reader action (privacy mode default is conformant).
5. **Blocked destinations — completion and notation defenses.** In addition to the ranges in items 1–3, fetches MUST refuse: IPv6-mapped IPv4 (`::ffff:0:0/96`, e.g. `::ffff:10.0.0.1`), NAT64 (`64:ff9b::/96`), `0.0.0.0/8`, `100.64.0.0/10` (CGNAT), `198.18.0.0/15`, and IP literals in any alternate notation (integer, octal, hex, or mixed forms — e.g. `http://2130706433/`). A DNS name that resolves to ANY blocked address is blocked.
6. **Redirect and rebinding hardening.** Every redirect hop MUST independently satisfy all destination rules before being followed, and the connection target MUST be re-validated at connect time. Implementations SHOULD pre-resolve and pin (connect only to a pre-validated address) as DNS-rebinding defense. *(Stated once: connect-time revalidation is already MUST above and is the minimum rebinding defense **every** conforming fetcher owes its users; pre-resolve-and-pin is the recommended implementation strategy for that MUST and is additionally MUST for the reference renderer per the hardening note below. Third-party renderers are not second-class: skipping connect-time revalidation is non-conformant for them too.)*
7. **Fetch on action, not preview (SHOULD).** Key, document, and media fetches SHOULD be triggered by an explicit verify/reveal action, not by passive preview rendering. Privacy-mode-default implementations are conformant (extends item 4).
8. **Scope: all renderer-initiated fetches.** This section governs every fetch a conforming renderer/directory initiates because of a document — including `contact:` endpoints, URLs linkified in the body, `issuer-url`, and feed/profile pointers — not only keys, documents, and media. Two follow-ons: (a) navigation away from the document view (a human or agent clicking a body link) SHOULD carry an off-document warning — the destination is outside the rendered, no-execution channel; (b) agents MUST NOT auto-fetch `contact:` endpoints — contacting an issuer is a reader-authorized action, never an ambient one (an endpoint that learns your IP/UA on contact is a tracking surface; [CORE] §8 item 1's data-not-instructions rule has a fetch-side twin — see also §6.1 render integrity).

*Item 8 handling split:* (i) `mailto:` contact values open the reader's mail client — never fetched, never probed; (ii) HTTPS `contact:` values follow the full fetch policy if fetched, but the default is **no prefetch** — they open only on human- or agent-authorized action with the off-document warning (this prevents agents from probing contact URLs as health checks); (iii) body links follow the off-document warning rule (a) above.
9. **Media privacy default.** The reference renderer ships with privacy mode ON by default: media loads only on explicit reader action (item 7), and unique per-request image URLs (a fingerprinting vector — item 4) are flagged. Receivers MAY offer a proxied/cached image path as a reader option; publishers learn nothing about the render by default.

**Reference renderer hardening.** For the reference renderer specifically (the `/render?url=…` endpoint and its embeddable library), pre-resolve-and-pin (item 6) is **MUST**, not SHOULD: the reference implementation defines the safe path others copy, and an open public render endpoint without pinning is the single highest-risk surface in the ecosystem. Hostile fetch cases (IP-literal notations, mapped ranges, redirect chains to internal targets) MUST pass before public exposure.

**Conformance framing.** A renderer or directory that fetches without this policy is **non-conformant** — same weight as the no-execution rule. The reference renderer (`/render?url=…`) MUST implement this section from first public exposure.

*[CORE] 01-CORE — Core Format. [SERVICES] 03-SERVICES — Trust Services & Receivers.*
