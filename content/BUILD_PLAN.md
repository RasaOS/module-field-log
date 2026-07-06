# `rasa.module.field-log` — Specification & Build Plan

**Status:** v0.1.0 = spine + spec + seam + ledger template. The skills are
the build phase (M-1..M-3 below) — gated, but this is the module with a
real live consumer, so the gate is closest to opening here.

This is the author-time specification. The installed spine is
`content/field-log-rules.md`; this file is the design record behind it.

---

## Why this module exists (and why it's the proven one)

The RasaOS business-ops set (`leads`, `schedule`, `scheduler`, `contracts`,
`invoices`, `field-log`) is six modules on one shared account spine. Five of
them are distilled from the *shape* of a workflow. **field-log is different:
a live consulting/field practice already runs this exact loop by hand** —

1. **schedule** the visit (a booking on the calendar),
2. **drive out** to the customer site,
3. do the **on-site work**,
4. **debrief** afterward — what we did, how it went, pros/cons, what happens
   next.

That is the actual working process, not an invented one. field-log records
step 4 (and the trip around it) as one durable record per visit. Because the
source has *run in production*, field-log is the natural **first** business-ops
module to grow real skills — it clears the `module.tasks` extract-after-proof
bar the moment a real parent declares it. v0.1.0 is still a disciplined shell;
overbuilding ahead of that declaration is the failure mode the whole set
guards against.

## Non-goals (hard boundaries)

- **Not the calendar.** *Booking* a visit is `rasa.module.schedule`. A
  booking hands off to field-log to become a visit; field-log doesn't own
  the calendar.
- **Not the invoice.** *Billing* a visit is `rasa.module.invoices`. A
  `completed` billable visit *feeds* an invoice line; field-log carries the
  `billable` flag and `onsite_hours`, not the money.
- **Not the account record.** The customer's name/contacts/address live on
  the account hub (SA-032). field-log references `@acct-<slug>` and never
  seeds a competing accounts registry.
- **Not a CRM activity firehose.** One record per real field visit, honest
  and human-authored — not an auto-logged stream of every touch.

---

## Entity model

One project-owned live ledger + one project-owned seam. The record is the
**Visit**.

### `field-log/FIELD-LOG.md` (the live ledger) — the load-bearing surface
Append-per-visit, ordered. One entry per Visit, carrying:

| Field | Type | Notes |
|---|---|---|
| `id` | `VIS-NNN` | Monotonic; never reused. |
| `account` | `@acct-<slug>` | **Required.** Shared account spine. |
| `date` | absolute date | Never relative. |
| `purpose` | one line | Why we went. |
| `location` | text | Where. |
| `attendees` | ours + theirs | Two lists — who went, who we met. |
| `travel` | `{depart, arrive, onsite_start, onsite_end, return}` | The trip envelope. |
| `onsite_hours` | number | Time on-site (feeds invoices if billable). |
| `what_we_did` | prose | Work performed. |
| `what_we_covered` | prose/list | Topics / systems / agenda. |
| `outcome` | one line | Where it landed. |
| `sentiment` | how-it-went | Honest read (scale defined in the gate). |
| `pros` | list | What went well. |
| `cons` | list | What went badly. |
| `followups` | list | Each: next-action + owner + date. The open loop. |
| `billable` | bool | Soft link to invoices. |
| `notes` | prose | Anything else. |

Rules: `account` required; ids monotonic; dates absolute; never revise a
visit's honest read after the fact; follow-ups closed in place with a
close-date, never deleted.

### `.claude/field-log-gate.md` (the seam) — the one per-practice thing
See "The adapter seam" below.

### The account hub (SA-032) — referenced, not owned
Not a ledger this module ships. The `@acct-<slug>` handle joins to it when
it exists; until then the handle is the join key and display names resolve
from a project-owned account ledger if one is mounted.

---

## The states / lifecycle

**planned → completed → debriefed**, with follow-ups closing over time
independently of visit state.

- **planned** — on the books, hasn't happened. `account`, `date`, `purpose`,
  `location`, intended `attendees`.
- **completed** — we went; on-site work done. `travel`, `onsite_hours`,
  `what_we_did`, `what_we_covered`, `outcome`, `billable`.
- **debriefed** — the debrief happened. `sentiment`, `pros`, `cons`,
  `followups`.

Honest notes: a drop-in/emergency visit can be born `completed` (no
`planned`); follow-ups outlive the visit — "open follow-ups" is a cross-visit
view, not a fourth state.

---

## The adapter seam — `.claude/field-log-gate.md`

The crux of the design, mirroring `module.tasks`' done-gate and
`module.releases`' release-gate. It holds the four things that genuinely
vary per field practice:

1. **Visit types** — the practice's own taxonomy
   (`install | audit | training | support | sales-call | …`). Shapes what
   `what_we_did` / `what_we_covered` mean.
2. **Debrief template / prompts** — what "pros / cons / how it went" *means*
   for this practice; the sentiment scale and the debrief questions the
   `/debrief` skill will ask.
3. **Travel / mileage policy** — how travel time + mileage are recorded and
   whether reimbursable/billable. Drives `travel` + `onsite_hours`.
4. **Billable-hours policy** — what makes a visit billable and how on-site
   hours become a billable line (feeds `invoices`).

Why a seam and not fixed logic: a good debrief for a plumbing call-out and a
good debrief for a strategy consult share the *record*, not the *questions*.
The horizontal core is the Visit record + the planned→completed→debriefed
discipline; the vertical part is the practice's own definitions. `/debrief`
reads the gate so its prompts fit the practice — inventing them is the one
inference the module refuses to make.

---

## Skills (the build phase — M-1..M-3) — DEFERRED, not implemented here

Three skills, MVP-scoped. Each is a thin driver over the ledger + gate; the
discipline lives in `field-log-rules.md`, not duplicated per skill. **None
are implemented in v0.1.0** — they are listed as the build plan.

### M-1 — `/visit`
- **`/visit`** — capture a visit. Creates or advances a Visit record:
  `planned` (booking → on the books) or straight to `completed` (a drop-in).
  Collects `account` (required — refuses without it), `date`, `purpose`,
  `location`, `attendees`, and the trip/`travel` + `onsite_hours` +
  `what_we_did`/`what_we_covered`/`outcome`/`billable` when completing.
  Assigns the next `VIS-NNN`.

### M-2 — `/debrief`
- **`/debrief VIS-NNN`** — the guided post-visit debrief. Reads
  `.claude/field-log-gate.md` for the practice's debrief template + sentiment
  scale, then walks: **what we did → how it went (sentiment) → pros → cons →
  follow-ups (each next-action + owner + date)**. Advances the visit to
  `debriefed`. Hard-stops with a clear message if the gate has no debrief
  template (won't invent one). Never softens the honest read.

### M-3 — `/field-log`
- **`/field-log [account @acct-<slug>] [open-followups]`** — read/render the
  ledger. Default: recent visits + all open follow-ups. Filtered by account,
  it's the account's field history in order. `open-followups` is the
  cross-visit chase list (next-action + owner + date). Read-only.

**Style/quality bar:** match `module.ingest` / `module.releases` SKILL.md
files — a crisp operation list, honest hard-stop behavior, no invented
plumbing. Fan-out plan when the gate opens: author `/debrief` as the
reference skill (it's the load-bearing one), then `/visit` + `/field-log` in
parallel against it, gate each independently (the four-engineering-modules
build pattern).

---

## Soft cross-references

All soft — present-if-mounted, graceful-if-absent, never `requires.elements[]`:

- **`rasa.module.schedule`** — a scheduled booking becomes a `planned`
  visit (schedule → field-log hand-off).
- **`rasa.module.invoices`** — a `completed` billable visit feeds an
  invoice line (field-log → invoices, via `onsite_hours` + `billable`).
- **`rasa.module.tasks`** — a follow-up that becomes assignable work can
  cite a `TASK-NNN`.
- **The account hub (SA-032)** — display-name resolution for `@acct-<slug>`.

---

## HONEST SCOPE / deferred

- **v0.1.0 (this) ships a SHELL.** The spine (`field-log-rules.md`), the spec
  (this file), the seam template, and the ledger template. **No skills.** The
  Visit record and the lifecycle are specified; the `/visit`, `/debrief`, and
  `/field-log` drivers are *not written*.
- **Skills are gated per extract-after-proof.** Even though a real practice
  runs this loop today, the discipline is: hold the skill build until a real
  parent (a `domain` or `orchestrator`) declares `rasa.module.field-log` in
  its `requires.elements[]`. field-log is expected to be the **first** of the
  six to clear that gate — but it clears it by declaration, not by
  anticipation.
- **The account hub is not here.** field-log leans on `@acct-<slug>` as a
  join key; the canonical account record is SA-032 and does not yet exist.
  When it lands, records refactor from shared-key join to hub FK with **no
  change to the records themselves**.
- **`sentiment` scale, visit types, debrief prompts, travel + billable
  policy are all in the seam**, not the spine. A practice must fill
  `.claude/field-log-gate.md` before `/debrief` can run against real
  definitions.

---

## Version plan

- **v0.1.0 (this)** — spine + spec + seam template + ledger template. No
  skills.
- **v0.2.0** — M-1..M-3 skills authored, once a real consumer declares the
  module (field-log is the likeliest first). `bin/init` smoke-tested into
  that consumer.
- **v1.0.0** — the seam format + ledger shape + record fields locked after
  the loop has run through the module (not just by hand) in a real practice.
