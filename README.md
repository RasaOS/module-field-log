# Rasa · Module · Field-Log

**Canonical name:** `rasa.module.field-log`
**Repo / folder:** `module-field-log`
**Kind:** `module` (canon Spec §6)
**Contract:** Element Contract v1.3.0
**Version:** 0.1.0 (initial shell)
**Status:** v0.1.0 shell — spine + spec + adapter seam + ledger template.
Skills deferred to the build phase.

## What this is

The **field-engagement** module of the RasaOS business-ops set. It owns
exactly one record — the **Visit**: one customer field engagement, captured
honestly as **drive out → do the on-site work → debrief (what we did, how it
went, pros/cons, follow-ups)**.

It's the proven one. A live consulting/field practice already runs this exact
loop by hand — schedule the visit, drive out, do the work, debrief afterward
— so the record here is a real working shape, not an invented one. That makes
field-log the natural first business-ops module to grow real skills. v0.1.0
is still a disciplined shell.

## The record it owns

One **Visit** per customer field engagement, moving through
**planned → completed → debriefed** (follow-ups close over time). Fields:
`id` (`VIS-NNN`), `account` (`@acct-<slug>`), `date`, `purpose`, `location`,
`attendees` (ours + theirs), `travel`, `onsite_hours`, `what_we_did`,
`what_we_covered`, `outcome`, `sentiment`, `pros`, `cons`, `followups`
(next-action + owner + date), `billable`, `notes`. Full definition in
`content/field-log-rules.md`.

## Element- vs project-owned files

| Source | Installs to | Policy | Owner |
|---|---|---|---|
| `content/field-log-rules.md` | `.claude/field-log-rules.md` | file-replace | **Element** — the spine; refreshes on upgrade. |
| `seed/field-log-gate.md.template` | `.claude/field-log-gate.md` | skip-if-exists | **Project** — the seam (visit types, debrief template, travel + billable policy). |
| `seed/field-log/FIELD-LOG.md.template` | `field-log/FIELD-LOG.md` | skip-if-exists | **Project** — the live visit ledger. |
| `seed/rasa.lock.json.template` | `.claude/rasa.lock.json` | init-only-with-sha | Connection-Contract lockfile, SHA-stamped at init. |

`content/README.md` + `content/BUILD_PLAN.md` are author-time docs (opt-in),
not installed into consumers.

## The account spine

Every Visit references its customer as `account: @acct-<slug>` — the same
opaque, stable handle the whole business-ops set (`leads`, `schedule`,
`contracts`, `invoices`, `field-log`) shares. That shared key is what lets a
visit join to the booking that scheduled it (`schedule`) and the invoice it
feeds (`invoices`). The account's own record lives on the account hub (canon
task **SA-032 — engagement-hub primitive**), which RasaOS doesn't have yet —
this module never seeds a competing accounts registry. When the hub lands,
records refactor from shared-key join to hub FK with no change to the records.

## Skills deferred to the build phase

The `/visit`, `/debrief`, and `/field-log` skills are **not** in v0.1.0. Per
the RasaOS extract-after-proof precedent (`module.tasks`), they're held until
a real parent declares `rasa.module.field-log` in its `requires.elements[]`.
field-log is expected to clear that gate first — but it clears it by
declaration, not anticipation. See `content/BUILD_PLAN.md` (M-1..M-3) for
their contracts.

## See also

- `~/rAI/rasa-os/elements/module-notes/` — the shell precedent (spine + spec
  + seam + ledger, skills deferred) this module follows.
- Canon Spec §6 — the `module` kind definition; `ELEMENT_CONTRACT.md` §7 —
  install policies.
- `~/rAI/rasa-os/elements/REGISTRY.md` — register this Element here on first ship.
