# CHANGELOG — `rasa.module.field-log`

Reverse-chronological. Each entry is a version bump.

---

## 0.1.1 — 2026-07-09

### `parent_kind` → `[domain, tenant]` (canon SA-023)

- The `orchestrator` kind was folded into `tenant`; this module now mounts into a tenant or a domain (`requires.parent_kind: ["domain", "tenant"]`, was `["domain", "orchestrator"]`).

## 0.1.0 — INITIAL SHELL (2026-07-05)

Initial shell of the **field-engagement** module of the RasaOS business-ops
set. Ships the spine + the specification + the adapter seam + the live-ledger
template; skills deferred per extract-after-proof; the shared account-handle
spine threaded through.

### What ships

- **The spine** — `content/field-log-rules.md` (installs to
  `.claude/field-log-rules.md`, file-replace): the **Visit** record and its
  canonical field set, the **planned → completed → debriefed** lifecycle,
  the shared `@acct-<slug>` account spine (verbatim convention block), the
  boundary against `schedule` / `invoices`, and the ledger conventions.
- **The specification** — `content/BUILD_PLAN.md`: the Visit entity model,
  the lifecycle, the adapter seam, the deferred `/visit` `/debrief`
  `/field-log` skills (M-1..M-3), the soft cross-refs, and an honest scope
  section.
- **The adapter seam** — `seed/field-log-gate.md.template` (installs to
  `.claude/field-log-gate.md`, skip-if-exists): the four per-practice things
  — visit types, the debrief template, travel/mileage policy, billable-hours
  policy. Placeholder-free, honest defaults + commented examples.
- **The live ledger** — `seed/field-log/FIELD-LOG.md.template` (installs to
  `field-log/FIELD-LOG.md`, skip-if-exists): one entry per visit, with two
  illustrative example rows spanning the lifecycle.
- `content/README.md` — author-time doc describing what installs where.

### Notes

- **The proven one.** A live consulting/field practice already runs this
  exact loop by hand (schedule → drive → on-site → debrief), so the record
  is a real working shape. field-log is expected to be the first business-ops
  module to grow real skills.
- **Skills deferred.** `/visit`, `/debrief`, `/field-log` are the build phase
  (BUILD_PLAN M-1..M-3), held until a real parent declares the module in
  `requires.elements[]` (the `module.tasks` extract-after-proof precedent).
- **Account spine threaded.** Every Visit carries `account: @acct-<slug>`;
  the account's own record is the account hub (canon task **SA-032**), not
  seeded here.
- **Soft references only.** `schedule` (booking → visit), `invoices`
  (billable visit → line), `tasks` (follow-up → TASK-NNN), account hub
  (display-name resolution) — all present-if-mounted, never
  `requires.elements[]`.
- Forked from `rasa.module.core` baseline via `bin/new-element`; domain-core
  scaffold cruft (SHAPE.md, agents/, rules/, skills/, output-style-enforcement,
  CLAUDE.md.template, output-style.md.template) stripped.
- `bin/check-manifest` GREEN.

### Deferred to next version

- **v0.2.0** — author the `/visit`, `/debrief`, `/field-log` skills, once a
  real consumer declares the module. `bin/init` smoke-tested into it.
- Create `RasaOS/module-field-log` on GitHub; first commit + tag + push;
  register in `elements/REGISTRY.md` + `elements/CHANGELOG.md`.
