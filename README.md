# intents.md

**Tell agents what you want.**

`AGENTS.md` told agents how to work. **intents.md tells them what you want.**

An intent document is a plain, optionally-signed markdown file at an ordinary URL — an offer, a request, an announcement. Any agent, directory, or browser can read it. No platform holds the documents; no publisher code ever runs. Submit your markdown. (The pun is free: intents, in tents. Pitch yours.)

## 60 seconds to your first intent

1. Write a markdown file.
2. Put it at a URL.
3. Done. That's the whole standard.

No SDK. No registry. No client. No account.

```markdown
intent: 2
type: listing
title: Mountain bike, lightly used
ask: 450 CHF
contact: mailto:seller@example.org

Cannondale Synapse, 2023, serviced, Zurich pickup.
```

That file is a complete, valid intent document. (The `intent:` header is the format-version field.)

## Why

Agents are scraping a web built for human eyes. intents.md flips it: publishers ship structured plain text once; readers — humans or agents — decide what it becomes. Personalization moves to the reader's side. No surveillance, no platform tax, no execution: the worst a hostile document can do is *be a lie*, never *run code*.

Naming is convenience, never trust: `<slug>.intents.md` is the recommended filename, and a bare `intents.md` at the root is the conventional index — but any markdown at any URL that parses is valid, whatever its name. Grammar is identity. Signatures are truth. Filenames are just signs.

The same file works privately: an `intents.md` in your vault, read by your own agents, telling them your standing goals and current priorities. Trust by possession — no signatures needed. One format, both lives.

## What this is not

- **Not an execution protocol.** Documents declare; they never run. No payments, no escrow, no negotiation machinery in the format — deals close out-of-band.
- **Not a platform.** No registry, no accounts, no client you must install.
- **Not JSON-in-a-well-known-path.** Humans read these files too. `cat` is a conforming renderer.
- **Not `intent.md`.** The singular is a dev-side spec convention — a per-feature brief, inside your repo, that your coding agent executes (IntentSpec and friends). intents.md is web-side: at a URL, read by any agent or human, expressing what you want.

## The three documents

| Doc | Status | What it defines |
|---|---|---|
| [01-CORE — Core Format](01-CORE.md) | Normative | Envelope, headers, content types, rendering tiers, money/secrets rules, agent safety, render integrity |
| [02-TRUST — Trust & Integrity](02-TRUST.md) | Normative | Trust levels L0–L2, signatures, canonical form, revocation, proof-of-control, verifier fetch policy |
| [03-SERVICES — Trust Services & Receivers](03-SERVICES.md) | Non-normative | The anti-abuse ladder, receiver posture, matching/advertising patterns |


## Key properties

- **Plain text forever.** A document is readable in `cat`. Tier-0 rendering is unconditionally safe.
- **Signatures optional, graduated.** L0 anonymous → L1 same-key pseudonymity → L2 domain-bound (verifier fetches the key from the issuer's registrable domain).
- **Receiver sovereignty.** The spec never adjudicates trust; receivers rank, filter, and weight — visibly.
- **No execution, no credentials.**
- **Hash-anchored commerce.** Offers cite document URL + canonical-form hash; reviews bind to subjects by key/hash; deals close out-of-band.

## Examples

[examples](https://intentsmd.org/examples) — one minimal document per type (`listing`, `rfp`, `profile`, `review`, `feed`), a signed listing showing the signature block shape, a personal `intents.md` (the vault variant), `seeking-collaborator.intents.md` — and `hostile-listing.intents.md`, a malicious fixture (script tags, tracker image, prompt injection) that every conforming renderer displays as inert text.

## Conformance

A second, independent implementation of canonicalization + sign/verify that agrees byte-for-byte is the ratification gate for the spec; the machine-readable conformance suite ships with the reference implementation at that gate.

## Governance & neutrality

intents.md is an open protocol, not a product, licensed MIT, and the name is free for any conforming implementation. Goldcote maintains this repo as a first implementer, not an owner: changes are by consensus, additions are permanent, and stewardship is intended to move to a neutral, community-governed foundation.

No Goldcote product is required. You can publish, sign, read, and verify an intent document with nothing but a text file, any HTTPS host, and a software signing key. Separate implementations compete above the open floor — none is ever a dependency.

## Status

**Draft** — MIT license; no trademark games. Consensus additions only, permanent once added.
