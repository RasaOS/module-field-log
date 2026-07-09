# CLAUDE.md — `rasa.module.field-log`

> **Who you are (SA-025).** `rasa.module.field-log` — the RasaOS module for field logging. Substrate: **RasaOS**; role: **module**. On install `bin/init` renders this into `.claude/rasa-identity.md`; `/whoami` composes the full identity with the project's deployment layer.


Per-repo working contract for Claude sessions opened inside this folder.
Extends `~/.claude/CLAUDE.md` and the workspace `~/rAI/rasa-os/CLAUDE.md`
(the `rasa.tenant.rasaos` tenant's contract); does not override them.

## What you are when you're in this folder

You are working on **`rasa.module.field-log`** — a `module`-kind, toolkit-shaped
business-ops Element. It is the **field-engagement** module of the RasaOS
business-ops set (`leads` / `schedule` / `contracts` / `invoices` /
`field-log`).

It owns exactly one record — the **Visit**: one customer field engagement,
captured as **drive out → on-site work → debrief (what we did, how it went,
pros/cons, follow-ups)**, across the **planned → completed → debriefed**
lifecycle. This is the *proven* one of the set: a live consulting/field
practice already runs this exact loop by hand, so the record is a real
working shape, not an invented one.

The portable spine is `content/field-log-rules.md`; the per-practice slice
(visit types, debrief template, travel + billable policy) is the
project-owned `.claude/field-log-gate.md` seam. The full specification and
build plan are in `content/BUILD_PLAN.md`.

### The account spine

Every Visit references its customer by a stable, opaque handle:
`account: @acct-<slug>`. This is the shared join key across the whole
business-ops set — a visit joins to the booking that scheduled it and the
invoice it feeds by this key. This module **never invents or mutates the
account's own record**; that lives on the account hub (canon task **SA-032**),
which doesn't exist yet. Do not seed a competing accounts registry here.

### Skills are deferred (extract-after-proof)

v0.1.0 is a **shell**: spine + spec + seam + ledger template, **no skills**.
The `/visit`, `/debrief`, `/field-log` skills are the build phase, held until
a real parent declares this module in its `requires.elements[]` (the
`module.tasks` precedent). field-log is expected to be the first of the six
to clear that gate — but by declaration, not anticipation. Don't author the
skills ahead of that.

## Status

**v0.1.0 — initial shell.** Spine (`content/field-log-rules.md`) + spec
(`content/BUILD_PLAN.md`) + adapter seam (`seed/field-log-gate.md.template`)
+ live-ledger template (`seed/field-log/FIELD-LOG.md.template`). Skills
deferred. See CHANGELOG.md.

## Source of truth

- **`~/rAI/rasa-os/canon/`** — authoritative for every architectural
  decision (Spec §6 defines the `module` kind). Canon wins.
- **`elements/domain-core/`** — the template this Element forked
  from. Shape questions go there, not here.
- **`rasa.json`** — this Element's formal declaration.
- **`~/rAI/rasa-os/elements/REGISTRY.md`** — the live workspace snapshot.

## Don'ts

- **You are NOT the template.** If this contract ever describes
  `domain-core` (or any other Element), the template-CLAUDE.md
  drift class is back — flag it.
- **Don't author content ahead of the authoring phase** without the user
  driving.
- **Don't `bin/init` this Element into itself.** `content/` is the
  source (workspace rule).
- **Don't push to GitHub from the Cowork sandbox.** Local commit + tag;
  the user pushes (workspace rule).

## How a version bump works

Each bump: edit `VERSION` + `rasa.json#version`, write a CHANGELOG entry,
run `bin/check-manifest`, commit + tag `v<version>`. Update
`~/rAI/rasa-os/elements/REGISTRY.md` +
`~/rAI/rasa-os/elements/CHANGELOG.md` (track #2).
