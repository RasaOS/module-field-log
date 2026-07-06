# `rasa.module.field-log` — content

What this module ships and where it installs. This file is author-time
documentation (not installed into consumer projects).

## The one-liner

The **field-engagement** module: one honest record per customer visit —
**drive out → do the on-site work → debrief (what we did, how it went,
pros/cons, follow-ups)** — attached to a customer account by a stable
handle. Distilled from a live consulting/field practice that runs this
loop by hand today.

## What installs where

| Source | Installs to | Policy | What it is |
|---|---|---|---|
| `content/field-log-rules.md` | `.claude/field-log-rules.md` | file-replace | The spine — the Visit record, the planned → completed → debriefed lifecycle, the account spine, the sibling boundary. Element-owned; refreshed on upgrade. |
| `seed/field-log-gate.md.template` | `.claude/field-log-gate.md` | skip-if-exists | **The seam** — the practice's visit types, debrief template, travel policy, billable policy. Project-owned; the practice fills it. |
| `seed/field-log/FIELD-LOG.md.template` | `field-log/FIELD-LOG.md` | skip-if-exists | The live visit ledger. Project-owned; grows over the practice's life. |
| `seed/rasa.lock.json.template` | `.claude/rasa.lock.json` | init-only-with-sha | Connection-Contract lockfile, SHA-stamped at init. |

## What is NOT here yet

The `/visit`, `/debrief`, `/field-log` **skills** — they are the build
phase. See [`BUILD_PLAN.md`](BUILD_PLAN.md) for their contracts and the
extract-after-proof gate. field-log is the module *most* likely to build
them first (it has a real, live consumer), but v0.1.0 is deliberately a
shell.

## The account spine

Every Visit references its customer as `account: @acct-<slug>` — the same
opaque handle the whole business-ops set (`leads`, `schedule`, `contracts`,
`invoices`, `field-log`) shares. That shared key is what lets a visit join
to the booking that scheduled it and the invoice it feeds. The account's
own record lives on the account hub (canon task **SA-032**), not here — this
module never seeds an accounts registry.

## The shape

Toolkit module, `requires.parent_kind: [domain, orchestrator]`. Pure
Element-layer convention — no kernel engine. Cross-refs to `schedule`,
`invoices`, `tasks`, and the account hub are all soft (present-if-mounted).

## See also

- [`BUILD_PLAN.md`](BUILD_PLAN.md) — the full spec + build plan.
- `content/field-log-rules.md` — the installed spine.
- `elements/module-notes/` — the shell precedent (spine + spec + seam +
  ledger, skills deferred) this module follows.
- Canon `ELEMENT_CONTRACT.md` §7 — install policies; Spec §6 — the
  `module` kind.
