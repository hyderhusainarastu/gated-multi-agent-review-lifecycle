# Verdict schema — field-by-field

This is the structured verdict every independent reviewer must emit at every gate. Copy the block
verbatim as a template; do not accept a review that only exists as free-form prose, because prose
cannot be mechanically routed, audited, or checked for the disqualifying conditions below.

```yaml
gate:                  <which gate this verdict belongs to>
date:                  <ISO date>
reviewer_task:          <the reviewing task/role/agent identifier>
reviewer_role:          <its named role in the workflow, e.g. "independent methods reviewer">
reviewer_model_or_worker: <exact identifier of who/what performed the review — never "default" or "auto">
authored_under_review:  false   # MUST be false
artifacts_reviewed:     [<exact path or identifier>, ...]
verdict:                PASS | PASS WITH CONDITIONS | FAIL
conditions:
  - id:                <finding id, e.g. F-01>
    blocking:          true | false
    severity:          <the reviewer's own scale, defined once and stated in the verdict>
    evidence:          <exact locator(s) the reviewer itself opened and checked>
    owner:             <task/role responsible for repairing this, or USER for a value/scope choice>
    resolution_test:    <the single observation that would settle whether this is fixed>
total_blocking:         <integer — count of conditions above with blocking: true>
advance_permitted:      YES | NO
reviewer_may_clear_own_conditions: false   # ALWAYS false
```

## Why each field exists

- **`gate`** — ties the verdict to exactly one entry in your gate-to-task index. A verdict with no gate
  field cannot be checked against that gate's stated exit criteria later.
- **`date`** — verdicts go stale. Anything that cites this verdict later needs to know how old it is,
  especially against a moving underlying artifact.
- **`reviewer_task` / `reviewer_role` / `reviewer_model_or_worker`** — three separate facts: which task
  slot did the review, what role that slot plays in the workflow, and precisely who or what executed it.
  Never collapse these into one vague "reviewer: yes" field — you need the exact identity to check
  independence (§5 of SKILL.md) and to know who to route a dispute to.
- **`authored_under_review`** — the self-disqualifying field. If this is `true`, the verdict is void
  regardless of what `verdict` says, because the reviewer has violated the no-self-approval rule. Put this
  field first in your own checking logic, before you even look at the verdict value.
- **`artifacts_reviewed`** — an exact list, not a description. "The manuscript" is not a list; the actual
  file paths or record identifiers are.
- **`verdict`** — exactly one of the three values. Do not invent a fourth ("mostly passes," "conditional
  fail") — if you need a fourth value, you are trying to smuggle a PASS through a FAIL, or vice versa.
- **`conditions[].blocking`** — required on every single condition, with no default. A condition with no
  stated blocking flag must be treated as blocking until a reviewer states otherwise — never the reverse.
- **`conditions[].severity`** — the reviewer defines its own scale and states it in the verdict (e.g.
  SEVERE / MODERATE / LOW). Different gates in the same workflow may use different scales; what matters is
  that each verdict is internally consistent and the scale is stated, not implied.
- **`conditions[].evidence`** — must be evidence the reviewer *itself* opened, not a locator copied from
  someone else's summary. If the reviewer is relying on another party's prior finding, say so explicitly
  and cite that party, rather than presenting inherited evidence as freshly checked.
- **`conditions[].owner`** — every condition needs an owner, even a value/scope condition that has no
  technical owner — those get `owner: USER` (or whatever your workflow's accountable-principal role is
  called) rather than being left blank.
- **`conditions[].resolution_test`** — if you cannot write this, the condition is not yet specified clearly
  enough to be actionable. Go back and sharpen the finding before publishing the verdict.
- **`total_blocking`** — a plain count, so status rollups don't have to re-derive it by re-reading every
  condition every time.
- **`advance_permitted`** — `YES` only when `total_blocking == 0` **and** this specific gate's definition
  states that carry-forward is permitted for whatever non-blocking conditions remain (or there are none at
  all). A `PASS WITH CONDITIONS` verdict with any blocking condition must set this to `NO`.
- **`reviewer_may_clear_own_conditions`** — always `false`. It exists as a standing, visible reminder in
  every single verdict that clearing a condition requires a *fresh* verdict, from this reviewer or another,
  reopening the artifact — never a follow-up note appended to the same verdict saying "now fixed."

## What an incomplete verdict looks like (avoid these)

- A verdict with a `verdict: PASS` but no `conditions` field at all when the reviewer's own prose
  mentions issues found. If issues were found, they belong in `conditions`, even if none of them are
  blocking.
- A `PASS WITH CONDITIONS` verdict where every condition is missing a `blocking` flag. Treat this as
  equivalent to `FAIL` until the reviewer supplies the flags — do not assume they're all non-blocking.
- `evidence` fields that just say "reviewed the document" without a specific locator (a line, a section,
  a field name, a timestamp). That is not evidence a dispute can be adjudicated against later.
- A verdict silently missing `authored_under_review`. Treat any verdict missing this field as
  disqualified until the reviewer confirms it — never assume it defaults to `false`.
