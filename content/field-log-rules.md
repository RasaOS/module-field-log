# Field-Log Rules

The portable field-engagement spine that `rasa.module.field-log` installs
at `.claude/field-log-rules.md`. It covers the one record this module owns —
the **Visit** — its canonical field set, the planned → completed → debriefed
lifecycle, the shared account-handle spine, the boundary against adjacent
business-ops modules, and the ledger conventions. **Read this file when
capturing a visit, running a debrief, tracking a follow-up, or asking "what
happened the last time we were on-site at this account?"**

This file is **Element-owned** — it refreshes on upgrade. It deliberately
does **not** decide the things that vary per practice:

- The practice's **visit types**, its **debrief template** (what "pros /
  cons / how it went" actually means for this practice), its
  **travel/mileage policy**, and its **billable-hours policy**.

Those live in the project-owned **`.claude/field-log-gate.md`** (the
field-log analogue of the task module's done-gate and the release module's
release-gate). This file references the seam; it hardcodes none of it. The
debrief and billable steps **read the gate** for the practice's own
definitions — inventing what a good debrief looks like for a practice we've
never seen is the one inference this module refuses to make.

## What this module owns — and what it doesn't

This module owns exactly **one record: the Visit** — a single field
engagement with a customer. One trip, one on-site, one interaction. It is
the honest, human-authored record of *we went there, this is what we did,
this is how it went, this is what happens next.*

It does **not** own:

- the customer's own record (name, address, contacts) — that's the account
  hub (see the account-spine section);
- the calendar booking that *scheduled* the visit — that's
  `rasa.module.schedule`;
- the invoice a billable visit *feeds* — that's `rasa.module.invoices`.

The Visit sits in the middle of that chain and references the others by
soft handle. It stands alone if they aren't mounted.

## Why this module is real (not speculative)

Unlike a shell built on a hypothesis, this one is distilled from a **live
consulting/field practice** that already runs this exact loop by hand:
**schedule the visit → drive out → do the on-site work → debrief afterward
(what we did, how it went, pros/cons, follow-ups)**. The record below is
that practice's actual working shape, not an invented one. That is why
field-log is the natural first business-ops module expected to grow real
skills — but v0.1.0 is still a disciplined shell (see BUILD_PLAN.md; skills
are gated per extract-after-proof).

## The Visit record — canonical field set

One Visit = one markdown entry in the live ledger (`field-log/FIELD-LOG.md`),
optionally with frontmatter. The canonical fields, every consuming practice
carries at minimum:

| Field | Type | What it is |
|---|---|---|
| `id` | `VIS-NNN` | Monotonic visit id; never reused. |
| `account` | `@acct-<slug>` | The customer handle (shared spine — see below). Required. |
| `date` | absolute date | When the visit happened (`2026-07-05`), never relative. |
| `purpose` | one line | Why we went — the goal of the trip. |
| `location` | text | Where — site/address/city. |
| `attendees` | two lists | **ours** (who went) + **theirs** (who we met). |
| `travel` | segment set | `depart`, `arrive`, `onsite_start`, `onsite_end`, `return` timestamps. |
| `onsite_hours` | number | Time actually on-site (derivable from `travel`, but recorded). |
| `what_we_did` | prose | The work performed on-site. |
| `what_we_covered` | prose/list | Topics, systems, agenda items addressed. |
| `outcome` | one line | Where things landed by the end of the visit. |
| `sentiment` | how-it-went | The honest read on how the visit went (the practice's scale lives in the gate). |
| `pros` | list | What went well. |
| `cons` | list | What went badly / friction / concerns. |
| `followups` | list | Each: **next-action + owner + date**. The open loop. |
| `billable` | bool | Whether this visit bills. Soft link to `rasa.module.invoices`. |
| `notes` | prose | Anything else worth keeping. |

Rules for the record:

- **`account` is required.** A visit with no account is a personal note,
  not a field-log entry — it belongs somewhere else.
- **`followups` are the live surface.** Everything else is history; the
  follow-ups are what the practice acts on after the debrief. Each carries
  an owner and a date so it can be chased and closed.
- **`sentiment`, `pros`, `cons` are honest, not performative.** The value
  of the debrief is that a future session can read "this account was
  frustrated about X" — softening it destroys the point.
- **Dates absolute. Ids monotonic.** `VIS-007`, never reused; never a
  relative date.

## The lifecycle — planned → completed → debriefed

A visit moves through three states. State is a property of the record, not
a separate ledger.

| State | Meaning | What's filled |
|---|---|---|
| **planned** | The trip is on the books; hasn't happened yet. | `account`, `date`, `purpose`, `location`, intended `attendees`. |
| **completed** | We went; the on-site work is done. | `travel`, `onsite_hours`, `what_we_did`, `what_we_covered`, `outcome`, `billable`. |
| **debriefed** | The post-visit debrief happened. | `sentiment`, `pros`, `cons`, `followups`. |

```
planned  ──(we went)──▶  completed  ──(debrief)──▶  debriefed
(on the books)           (work done)                 (how it went + follow-ups)
                                                            │
                                              follow-ups close over time
                                              (independently of visit state)
```

Two honest notes on the lifecycle:

- **A visit can be born `completed`.** If it wasn't scheduled ahead (a
  drop-in, an emergency call-out), skip `planned` and record it done.
- **Follow-ups outlive the visit.** A visit reaches `debriefed` once, but
  its `followups` close one at a time over the following days/weeks. "Open
  follow-ups" is a cross-visit view, not a fourth state.

## Customer / Account reference (shared spine)

> **Customer / Account reference (shared spine).** Every record this module
> owns references its customer by a stable **account handle**:
> `account: @acct-<slug>` in the record's frontmatter (for example
> `@acct-acme`). The handle is an opaque, stable join key. This module **never
> invents or mutates the account's own record** — the account's name,
> contacts, address, and billing details live on the **account hub**, which
> RasaOS does not yet have as a first-class primitive (tracked as canon task
> **SA-032 — engagement-hub primitive**). Until the hub lands, treat the
> handle as the join key and, if a project-owned account ledger is mounted,
> resolve display names from it; **do not seed a competing accounts registry
> here.** Every sibling business-ops module (`leads`, `schedule`, `contracts`,
> `invoices`, `field-log`) uses this same `@acct-<slug>` handle, so records
> join across modules today by shared key and refactor to the canonical hub
> FK — with no change to the records themselves — when SA-032 resolves.

## The boundary against adjacent modules — soft references

field-log is the middle of the field-engagement chain. It references
siblings by handle and **degrades gracefully** when they aren't mounted —
never a `requires.elements[]` dependency:

- **`rasa.module.schedule`** — a scheduled booking *becomes* a visit. When
  schedule is mounted, a `planned` visit can carry the booking id it came
  from; when it isn't, you just create the `planned` visit directly. The
  arrow is one-way: schedule hands off to field-log, not the reverse.
- **`rasa.module.invoices`** — a `completed` visit with `billable: true`
  *feeds* an invoice (its `onsite_hours` and date are the billable line).
  When invoices is mounted, the visit can note the invoice id it fed; when
  it isn't, `billable` is just a flag on the record for later.
- **The account hub (SA-032)** — the account handle is the join, per the
  shared spine above. Present-if-mounted for display-name resolution;
  never seeded here.

Rule of thumb: field-log records **what happened on the ground**. If you
find yourself recording *what was booked* (schedule) or *what was billed*
(invoices), that belongs to the sibling — link by handle, don't duplicate.

## The adapter seam — `.claude/field-log-gate.md`

The one thing this module cannot know for a practice it's never seen. The
gate holds the four things that genuinely vary per field practice:

1. **Visit types** — the practice's own taxonomy (e.g.
   `install | audit | training | support | sales-call`). `what_we_did`
   and `what_we_covered` mean different things per type.
2. **Debrief template / prompts** — what "pros / cons / how it went"
   *means* here. The practice's sentiment scale and its debrief questions.
   The debrief step reads this so the questions fit the practice.
3. **Travel / mileage policy** — how travel time and mileage are recorded
   and whether they're reimbursable/billable. Drives `travel` + `onsite_hours`.
4. **Billable-hours policy** — what makes a visit billable, and how
   on-site hours convert to a billable line (feeds `invoices`).

Ships with honest defaults + commented examples; the practice fills it in.
This file (the spine) references the gate; it hardcodes none of the four.

## Ledger conventions

- One live ledger: `field-log/FIELD-LOG.md`, git-versioned, project-owned
  (seeded skip-if-exists). Newest entries at the top or bottom — the
  project's call; be consistent.
- Markdown, human-first. Frontmatter/tables are fine, but the ledger must
  read cleanly top-to-bottom to a person — the retrievability is a human
  reading the account's history in order, not a query.
- **Never rewrite a visit's history.** A visit's `what_we_did` /
  `sentiment` / `pros` / `cons` are what was true that day. Correct a
  factual error, but never revise the honest read to be flattering after
  the fact — that defeats the module.
- **Follow-ups are checked off in place**, with a close-date, so the chain
  (raised on VIS-007 → closed 2026-07-20) stays traversable. Never delete
  an open follow-up; close it or supersede it.
- Ids monotonic (`VIS-NNN`), dates absolute.

## Soft cross-references (degrade gracefully)

This module stands alone, but plays well with siblings when mounted:

- **`rasa.module.schedule`** — the booking that became this visit
  (present-if-mounted).
- **`rasa.module.invoices`** — the invoice a billable visit fed
  (present-if-mounted).
- **`rasa.module.tasks`** — a follow-up that turns into assignable work can
  reference a `TASK-NNN`; the follow-up list here stays the lightweight
  open-loop view. Present-if-mounted.
- **The account hub (SA-032)** — display-name resolution for `@acct-<slug>`.

Do **not** harden any of these into a `requires.elements[]` dependency.
