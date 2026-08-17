# Worked example — a synthetic four-gate content pipeline

**Everything in this file is synthetic illustrative data.** The workflow ("Aurora Field Notes," a
fictional quarterly market-trends report), the people/roles, the findings, the numbers, and the quotes
below are all invented for this example. No real project, company, dataset, or individual is described.

This example shows the full gated lifecycle from SKILL.md running end to end: four gate definitions, a
gate-to-task index, a `PASS WITH CONDITIONS` verdict, a `FAIL` verdict that triggers failure routing, a
two-strikes reassignment, an adjudication between two disagreeing reviewers, and two correction-log
entries.

---

## The four gates

```text
## Gate 1 — Source research

Objective:
  Establish that every trend claim the report will make traces to a specific, checkable source.

Entry criteria:
  1. The report's target topic list is fixed (five candidate market segments).

Exit criteria:
  1. Every candidate segment has at least two independently sourced data points, each with an
     exact locator (report name, page or section, publication date).
  2. Any segment with fewer than two sources is explicitly flagged UNDER-SOURCED, not silently
     dropped or padded with a single source presented as sufficient.
  3. A source-conflict log lists any two sources that disagree on the same figure, with both
     locators — conflicts are recorded, never quietly resolved by picking the more convenient number.

Named artifacts:
  research/source_log.md; research/source_conflicts.md

Owning task / role:
  research-analyst

Independent reviewer:
  research-reviewer (never the research-analyst who produced source_log.md)

Non-blocking carry-forward:
  PERMITTED for an UNDER-SOURCED flag on a segment that will not be drafted this cycle.
  NOT PERMITTED for any source conflict bearing on a segment already scheduled for drafting.

## Gate 2 — Draft

Objective:
  Establish that the drafted section makes only claims the source log supports, in permitted wording.

Entry criteria:
  Gate 1 passed for every segment this draft covers.

Exit criteria:
  1. Every quantitative claim in the draft cites a source_log.md entry by id.
  2. No claim states a causal or predictive result ("X will drive Y") where the underlying source
     only supports a correlational or historical one ("X co-occurred with Y in the observed period").
  3. Word budget for the section is met.

Named artifacts:
  drafts/<segment-slug>.md

Owning task / role:
  drafting-writer

Independent reviewer:
  draft-reviewer (never the drafting-writer)

Non-blocking carry-forward:
  PERMITTED for style and transition wording only.
  NOT PERMITTED for any unsupported or overstated claim.

## Gate 3 — Editorial integration

Objective:
  Establish that the assembled report is internally consistent and no approved claim was altered
  by integration.

Entry criteria:
  Gate 2 passed for every section being assembled.

Exit criteria:
  1. A claim-by-claim diff against each passed draft shows no approved claim's substance changed.
  2. Terminology is unified across sections without flattening any section's stated uncertainty.
  3. Any surviving cross-section contradiction is recorded and routed, not smoothed over.

Named artifacts:
  deliverables/aurora_field_notes_q_synthetic.md; integration_report.md

Owning task / role:
  integration-editor

Independent reviewer:
  integration-reviewer (never the integration-editor)

Non-blocking carry-forward:
  PERMITTED for transition and cross-reference wording only.
  NOT PERMITTED for any altered claim or flattened uncertainty.

## Gate 4 — Publication readiness

Objective:
  Establish the report is complete, accurate to its own sources, and ready to leave the workflow.

Entry criteria:
  Gate 3 passed with no blocking condition.

Exit criteria:
  1. Every citation resolves to a real entry in source_log.md.
  2. A final adversarial read finds no claim exceeding what its source supports.
  3. The accountable principal (here: the fictional "program owner" role) signs off.

Named artifacts:
  deliverables/aurora_field_notes_q_synthetic.md (final)

Owning task / role:
  integration-editor

Independent reviewer:
  final-auditor; sign-off: PROGRAM OWNER

Non-blocking carry-forward:
  NOT PERMITTED. Any open condition at this gate is blocking by construction — this is the exit
  gate for the whole cycle.
```

### Gate-to-task index

| Gate | Owning task | Independent reviewer(s) |
|---|---|---|
| Gate 1 — Source research | research-analyst | research-reviewer |
| Gate 2 — Draft | drafting-writer | draft-reviewer |
| Gate 3 — Editorial integration | integration-editor | integration-reviewer |
| Gate 4 — Publication readiness | integration-editor | final-auditor; PROGRAM OWNER |

Scan check: no reviewer above matches its own gate's owning task; the orchestrator is not listed as a
reviewer anywhere in this table.

---

## A PASS WITH CONDITIONS verdict — Gate 1

```yaml
gate:                  Gate 1 — Source research
date:                  2026-02-10
reviewer_task:          research-reviewer
reviewer_role:          independent source-research reviewer
reviewer_model_or_worker: research-reviewer-instance-A
authored_under_review:  false
artifacts_reviewed:     [research/source_log.md, research/source_conflicts.md]
verdict:                PASS WITH CONDITIONS
conditions:
  - id:                F-01
    blocking:          false
    severity:          LOW
    evidence:          "research/source_log.md, segment 'micro-fulfillment' — only one source
                        (SL-14) currently logged; segment is not scheduled for drafting this cycle."
    owner:              research-analyst
    resolution_test:    "A second independent source is logged for 'micro-fulfillment' before that
                        segment is scheduled for drafting."
total_blocking:         0
advance_permitted:      YES
reviewer_may_clear_own_conditions: false
```

This passes because Gate 1's stated carry-forward policy explicitly permits an UNDER-SOURCED flag on a
segment not yet scheduled for drafting (see the gate definition above) — the policy was checked, not
assumed.

---

## A FAIL verdict — Gate 2, and the failure routing it triggers

```yaml
gate:                  Gate 2 — Draft
date:                  2026-02-17
reviewer_task:          draft-reviewer
reviewer_role:          independent draft reviewer
reviewer_model_or_worker: draft-reviewer-instance-A
authored_under_review:  false
artifacts_reviewed:     [drafts/last-mile-logistics.md]
verdict:                FAIL
conditions:
  - id:                F-02
    blocking:          true
    severity:          MODERATE
    evidence:          "drafts/last-mile-logistics.md, paragraph 3: 'this shift will reduce delivery
                        costs by a fifth' — source_log.md entry SL-22 reports only a *historical*
                        correlation observed in one region over one quarter, not a predictive claim."
    owner:              drafting-writer
    resolution_test:    "Paragraph 3 states the correlational finding as observed and scoped to its
                        region/period, with no forward-looking 'will' claim, or cites a source that
                        actually supports the predictive framing."
total_blocking:         1
advance_permitted:      NO
reviewer_may_clear_own_conditions: false
```

**Failure routing step 1 — return.** The orchestrator returns finding `F-02` to `drafting-writer`
unmodified — it does not soften "overstated causal claim" into "consider tightening this sentence," and
it does not attempt the rewrite itself.

---

## Two strikes, then a different specialist

```yaml
# Repair attempt 1 by drafting-writer: reworded to "this shift is associated with lower delivery
# costs in the observed region." Re-reviewed by draft-reviewer-instance-A:
gate:                  Gate 2 — Draft
date:                  2026-02-19
verdict:                FAIL
conditions:
  - id:                F-02
    blocking:          true
    severity:          MODERATE
    evidence:          "Same paragraph, new wording still implies the association generalizes beyond
                        the one region/quarter it was observed in — SL-22 makes no claim about
                        generalizability."
    owner:              drafting-writer
    resolution_test:    "The claim is explicitly scoped to the one region and one quarter SL-22
                        actually covers, with no implied generalization."
total_blocking: 1
advance_permitted: NO
```

```yaml
# Repair attempt 2 by drafting-writer: rewording still leaves the scope ambiguous. Same underlying
# defect (F-02) has now survived two repair cycles by the same owner.
```

Per SKILL.md §6 step 2, the orchestrator does **not** send a third request to `drafting-writer`. It
reassigns `F-02` to a different, qualified specialist — here, a senior editorial writer who did not
produce either failed revision — for the third attempt. The reassignment itself is logged:

```markdown
ROUTING NOTE — 2026-02-20
F-02 has survived two repair cycles by drafting-writer with the scope-generalization defect
recurring in a new form each time. Per the two-strikes rule, reassigned to senior-editorial-writer
for the third attempt. drafting-writer is not at fault for insufficient effort — this is treated as
a signal that the paragraph's underlying structure, not just its wording, needs to change.
```

---

## Adjudication — two reviewers disagree

Later, on a different section, `draft-reviewer-instance-A` and a second reviewer brought in for a
cross-check (`draft-reviewer-instance-B`) disagree about whether a claim in `drafts/subscription-churn.md`
is adequately sourced.

```yaml
# draft-reviewer-instance-A's position
finding: "The churn-rate figure (source SL-31) is a self-reported vendor statistic with no
independent replication; citing it as a section finding without qualification overstates its
reliability."
blocking: true

# draft-reviewer-instance-B's position
finding: "SL-31 is the only figure available for this metric anywhere in the source log; the draft
already labels it '(vendor-reported)' inline, which is sufficient qualification per Gate 2's exit
criteria — this does not need to block."
blocking: false
```

This is routed to an adjudicator — `program-adjudicator`, who authored neither review and has no stake in
either outcome — per SKILL.md §6 step 3:

```yaml
adjudication:
  adjudicator:          program-adjudicator
  authored_either_bundle: false
  primaries_reopened:   [research/source_log.md entry SL-31, drafts/subscription-churn.md paragraph 2]
  ruling: |
    Gate 2's exit criterion 1 requires every claim to cite a source_log.md entry by id — it does not,
    on its own text, require a reliability qualifier beyond that citation. The inline "(vendor-reported)"
    label satisfies the letter of the criterion. However, Gate 3's exit criterion 2 (terminology unified
    without flattening uncertainty) means this qualifier must survive editorial integration unchanged —
    that is recorded as a standing carry-forward condition into Gate 3, owner: integration-editor,
    rather than blocking Gate 2 itself.
  disposition: "F-03 (draft-reviewer-instance-A's finding) is non-blocking at Gate 2, PROVIDED it is
    logged as a Gate-3 tracked condition rather than dropped. draft-reviewer-instance-B's position is
    correct as to Gate 2; draft-reviewer-instance-A's underlying concern is correct as to what Gate 3
    must protect."
```

This is a genuine adjudication, not a coin flip: the adjudicator reopened both primaries, located the
exact governing exit criterion each reviewer was implicitly relying on, and resolved the dispute by
naming which gate each reviewer's concern actually belonged to — rather than simply picking a side.

---

## Two correction-log entries

```markdown
### C-01 — 2026-02-22 — source_log.md id scheme corrected after Gate-2 F-02 routing

**Finding.** During the two-strikes reassignment above, senior-editorial-writer discovered that
source_log.md used two different id formats interchangeably ("SL-22" and "SRC-22" both refer to the
same entry in two different places), which was part of why the earlier repair attempts kept citing
the entry inconsistently.

**Disposition — one canonical id kept, alias noted.** "SL-##" was kept as canonical (it is used in
31 of 33 existing citations); the two "SRC-22"-style references were corrected to "SL-22," and a
one-line alias note was added to source_log.md's header so a future reader who encounters "SRC-22"
in an older draft knows what it refers to.

**What changed here.** source_log.md header gained one alias note; two citations in
drafts/last-mile-logistics.md were corrected in place.

**What this correction does not do.** It does not close F-02 — a fresh Gate-2 verdict from
draft-reviewer-instance-A (or another qualified reviewer) must still confirm the scope-generalization
wording is fixed before Gate 2 passes for this section.

**Correcting party.** senior-editorial-writer, 2026-02-22.

### C-02 — 2026-02-24 — Gate-3 carry-forward table updated with the adjudicated F-03 condition

**Finding.** The adjudication ruling above (program-adjudicator, 2026-02-21) established that F-03
must be tracked into Gate 3 rather than dropped once Gate 2 passed for subscription-churn.md, but no
row for it yet existed in the program's single running carry-forward table.

**Disposition.** Added F-03 to the carry-forward table with its original id, owner
(integration-editor), and resolution test ("the '(vendor-reported)' qualifier is still present,
unflattened, in the assembled report's churn section"), exactly as recorded in the adjudication.

**What changed here.** One row added to carry_forward_table.md. No gate definition, reviewer, or
prior verdict was altered.

**What this correction does not do.** It does not mark F-03 resolved — that is Gate 3's own
integration-reviewer's job, at Gate 3, against the resolution test stated here.

**Correcting party.** program-orchestrator (routing role only — did not judge severity or resolve
the underlying disagreement; that was the adjudicator's role above), 2026-02-24.
```
