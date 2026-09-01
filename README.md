# loom-anchor

External tamper-evidence anchor for the **Loom** ledger. This repository is **not application code** — it holds
periodic checkpoints of the ledger's tip so that a rollback, truncation, or rewrite of the on-box ledger becomes
*detectable from outside the box*.

## Why this exists

Loom's ledger (`loom.db` on the box) is hash-chained, so tampering with the *interior* of its history is
self-evident. But a verifier running on the same box cannot, by itself, catch a **tail-truncation** (dropping
the most recent events) or a **full rewrite-and-re-chain** — an attacker, or a benign platform **Reset**, could
recompute the whole chain. Closing that hole needs an anchor in a trust domain the box's own credentials
**cannot rewrite**. That is this repository.

## The contract

- The box **appends** a checkpoint under `anchors/` on a schedule (the tip's `seq` and `row_hash` at publish
  time). It uses a **fine-grained token scoped to `contents:write` on this repo only** — never an admin token.
- **`main` is branch-protected**: force-push and deletion are blocked, even for the box's token. So the box can
  add a new checkpoint but can never rewrite or remove a past one. A rollback on the box then shows up here as a
  tip that no longer extends the last anchored checkpoint.
- The box **never holds a credential that can disable the protection** — that separation is the whole security
  value. The owner administers this repo; the box only appends.

## What a checkpoint is

Each checkpoint records the ledger tip it witnessed: the highest `seq`, that event's `row_hash`, and the
wall-clock time of publication. A verifier (a person, or a small GitHub Action added later) confirms each new
checkpoint's `seq` is greater than the last and that the chain is a consistent extension — the monotonicity
check that makes truncation and rewrite detectable.

*Provisional, alongside the Loom spec set. Nothing here is a build.*
