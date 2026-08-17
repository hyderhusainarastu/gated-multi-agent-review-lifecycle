---
name: gated-multi-agent-review-lifecycle
description: Defines a reusable gate lifecycle for multi-agent or multi-step work — structured PASS / PASS WITH CONDITIONS / FAIL verdicts with blocking flags and resolution tests, failure routing with two-strikes escalation and adjudication, a no-self-approval rule, an evidence ledger for traceability, and a recursive research-draft-review cycle for iterative work. Use when orchestrating a workflow — a research program, a content or manuscript pipeline, a software release process, a planning-then-execution program, or any multi-agent system — where one agent's output must be independently verified before downstream work is allowed to depend on it, and where "looks done" must not be allowed to substitute for "verified done."
---

# Gated multi-agent review lifecycle

This skill defines a **template methodology**, not a fixed set of gates. It tells you how to design,
run, and enforce a chain of quality gates for any multi-step or multi-agent workflow — and it supplies
the exact schemas, rules, and routing procedures that make the gates actually bind, instead of becoming
a checklist that gets rubber-stamped under time pressure.

Read this whole document before defining your first gate. It is meant to be followed end to end by an
agent that has no other context on this methodology.

## 1. When to use this

Use this skill whenever you are orchestrating work that:

- has more than one stage, where a later stage's output is only trustworthy if an earlier stage's
  output was actually correct (research → synthesis → drafting → publication; design → build → test →
  release; sourcing → verification → integration);
- involves more than one agent, worker, or role, especially when some of them might be tempted (or
  instructed) to mark their own work "done";
- has a real cost to letting a defect propagate downstream — rework, wasted spend, a factually wrong
  claim shipped to a reader, a broken release, an unresolvable data-integrity problem; or
- needs a defensible audit trail: something a human reviewer, auditor, or the eventual end user can
  reopen later and see exactly who verified what, when, and against which evidence.

Do **not** use the full machinery for genuinely single-step, single-owner tasks with no downstream
dependents — the ceremony has to be worth its cost. If you're unsure, start with a small number of gates
(three or four) rather than gating every micro-step; you can always add a gate later where the workflow
actually shows a defect slipping through.

## 2. Core vocabulary

- **Gate.** A named checkpoint a work product must pass before downstream work is allowed to rely on it.
  Each gate has an objective, entry criteria, exit criteria, named artifacts, an owning task/role, an
  independent reviewer, and a stated policy on whether non-blocking conditions may carry forward past it.
- **Verdict.** A structured report a reviewer issues about one or more artifacts: `PASS`,
  `PASS WITH CONDITIONS`, or `FAIL`. Never a prose paragraph alone — always the structured schema in
  §4 below, so verdicts are machine-routable and comparable across gates.
- **Blocking condition.** A defect flagged `blocking: true` in a verdict. A gate may not be passed while
  any blocking condition on its artifacts is open.
- **Non-blocking condition.** A defect flagged `blocking: false`. It may, under stated rules (§7), be
  carried forward past the gate rather than fixed immediately — but it is never silently dropped.
- **Owner.** The task, role, or agent responsible for producing an artifact and for repairing any defect
  found in it. An owner may not also be the reviewer of its own artifact (§5).
- **Independent reviewer.** A worker who authored none of the artifact under review, assigned specifically
  to review it, empowered to determine severity and blocking status, and never empowered to declare its
  own prior finding resolved without reopening the artifact fresh (§5, §6).
- **Adjudicator.** A reviewer of last resort, invoked only when two qualified workers disagree about a
  substantive finding. Never the same worker as either side of the dispute (§6).
- **The orchestrator / moderator.** Whatever routes work and status between owners, reviewers, and the
  user — a human, a coordinating agent, a script. The orchestrator may read and route verdicts. It may
  **never** determine severity, reinterpret a failure as a pass, or declare a blocking condition resolved.
  That authority belongs only to a fresh independent reviewer verdict (§5 rule 2) or, for value/scope
  choices, to the user (§6 step 4).

## 3. Step 1 — Define your gates

Before any work starts, enumerate the gates your workflow needs. For each one, fill out this template —
copy it verbatim into your own plan document (see `references/gate-definition-template.md` for a blank
copy and `examples/example-workflow-gates.md` for a fully worked synthetic instance):

```text
## Gate <letter-or-number> — <name>

Objective:             <one or two sentences: what must be true for this gate to pass>
Entry criteria:        <what must already exist / already be true before this gate can even be attempted>
Exit criteria:         <the enumerated, checkable conditions that together define "passed" —
                         write these as things a reviewer can test, not aspirations>
Named artifacts:        <the exact file(s), record(s), or deliverable(s) this gate governs>
Owning task/role:       <who produces the artifact>
Independent reviewer:   <who reviews it — must differ from the owning task/role, see §5>
Non-blocking carry-forward: <PERMITTED, for what kinds of defect specifically | NOT PERMITTED, and why>
```

Guidance for filling this in:

- **Exit criteria must be testable, not aspirational.** "The design is good" is not an exit criterion.
  "Every claim in the design doc cites a source, or is explicitly marked unsupported" is.
- **Name the artifacts exactly**, including where they live. A gate that says "the report" without a path
  or identifier cannot be mechanically checked later, and drifts the moment two reports exist.
- **State the non-blocking carry-forward policy explicitly, gate by gate — never as a blanket rule.**
  Some gates (anything touching safety, scope, legal compliance, or an irreversible external commitment)
  should permit **no** carry-forward at all: any open condition there is blocking by construction. Other
  gates can reasonably tolerate carrying forward precision defects (a wording nit, a missing
  cross-reference) as long as nothing substantive is unresolved. Decide this per gate, in writing, before
  work starts — not in the moment a convenient exception would help a deadline.
- **Build a gate-to-task index** once all gates are defined: one table listing every gate, its owning
  task(s), and its independent reviewer(s). Scan it for two failure patterns before you start work: (a)
  any gate whose reviewer is unnamed or is the same worker as the owner, and (b) any gate where the
  orchestrator itself is listed as a reviewer. Neither is acceptable — fix the index, not the rule.
- **Consider a meta-gate over your own gate definitions** for any workflow large enough to have more than
  six or seven gates, or more than one plan document: an independent, mechanical conformance pass (every
  reference resolves, every reviewer is distinct from every owner, every artifact path exists) plus a
  substantive audit pass (do the exit criteria actually cover what the objective promises; are there
  claims the plan makes that its own stated evidence doesn't support). This is the same discipline you're
  about to apply to the work — apply it to the plan first, because a plan with an internal contradiction
  will happily pass every one of its own gates without ever noticing the contradiction.

## 4. Step 2 — The verdict schema

Every reviewer, at every gate, emits exactly this structure. Do not accept a prose-only review — the
fields below are what make verdicts routable, auditable, and resistant to "I looked at it and it's fine."

```yaml
gate:                  <which gate this verdict belongs to>
date:                  <ISO date>
reviewer_task:          <the reviewing task/role/agent identifier>
reviewer_role:          <its named role in the workflow, e.g. "independent methods reviewer">
reviewer_model_or_worker: <exact identifier of who/what performed the review — never "default" or "auto">
authored_under_review:  false   # MUST be false — if true, this reviewer is disqualified; see §5
artifacts_reviewed:     [<exact path or identifier>, ...]
verdict:                PASS | PASS WITH CONDITIONS | FAIL
conditions:
  - id:                <finding id, e.g. F-01>
    blocking:          true | false     # REQUIRED on every condition, no exceptions
    severity:          <the reviewer's own scale, defined once and stated in the verdict>
    evidence:          <exact locator(s) the reviewer itself opened and checked — not inherited
                        from someone else's summary of the evidence>
    owner:             <task/role responsible for repairing this, or USER where it is a value
                        or scope choice rather than a defect>
    resolution_test:    <the single observation that would settle whether this is fixed>
total_blocking:         <integer — count of conditions above with blocking: true>
advance_permitted:      YES | NO
  # YES only if total_blocking == 0 AND this gate's carry-forward policy (§3) permits it here.
  # PASS WITH CONDITIONS may advance ONLY when no condition is blocking.
reviewer_may_clear_own_conditions: false   # ALWAYS false — see §5 rule 2
```

Two fields deserve emphasis because they are the ones people quietly drop under pressure:

- `authored_under_review: false` exists so a verdict is self-disqualifying the instant it is untrue. If
  the reviewer authored any of the artifacts it is reviewing, this field must read `true`, and any
  verdict where it reads `true` is void — it cannot pass the gate, no matter what the `verdict` field says.
- `resolution_test` on every condition exists so that "is this fixed?" is never a matter of opinion later.
  If you cannot state the single observation that would settle a finding, you have not actually specified
  the finding yet — go back and specify it.

## 5. Step 3 — Three rules that bind every gate

These three rules are what keep the gate lifecycle from decaying into theater. State them once, in your
governing document, and hold every gate to them without exception.

1. **A gate passes only when its named independent reviewer has issued a *fresh* qualifying verdict, and
   every blocking condition on its artifacts has been verified resolved by that same fresh verdict.** An
   artifact existing, or looking finished, is not a pass. A reviewer's memory of having looked at an
   earlier version is not a pass.
2. **Only a fresh independent-reviewer verdict may clear a blocking condition or verify that it has been
   resolved.** Not the artifact's own owner ("I fixed it, so it's fixed"), not the orchestrator, not a
   moderator summarizing the situation. An owner's belief that it repaired a defect clears nothing by
   itself — it creates a repair that is now *eligible* to be checked, not a cleared finding. Keep those two
   states visibly distinct in every status report: "repaired, awaiting fresh verification" is not the same
   sentence as "resolved."
3. **No worker may independently approve its own output, and the same worker may not both produce a draft
   and independently review that same draft.** This applies even when the worker is confident, even when
   deadline pressure makes reassigning a second reviewer feel wasteful, and even when the worker
   "obviously" got it right. The rule exists precisely for the cases where confidence and correctness have
   quietly come apart.

## 6. Step 4 — Failure routing

When an artifact fails a gate, or any condition on it is blocking, route it through these four steps in
order. Do not skip ahead to adjudication or escalation because a step feels unnecessary in the moment —
skipping is exactly how a real disagreement gets silently absorbed into "the reviewer's opinion won."

1. **Return.** The orchestrator returns the verdict's machine-readable findings to the responsible owner,
   unmodified. The orchestrator does not interpret them, rank them by its own judgment, soften their
   wording, or attempt the repair itself.
2. **Two strikes, then a different specialist.** If the *same defect* survives **two** repair attempts by
   the same owner, the orchestrator must assign a **different, qualified worker** to it on the third
   attempt. This is not discretionary, and it is not satisfied by re-prompting the original owner a third
   time with firmer wording. Two failed cycles on one defect is itself a signal — usually that the owner
   is missing context the defect needs, not that it needs to try harder at the same approach.
3. **Adjudication on disagreement.** If two qualified workers (for instance, an owner and its reviewer, or
   two reviewers) genuinely disagree about a substantive finding, route **both evidence bundles** — not a
   summary of either side — to an independent adjudicator who authored neither bundle and has no stake in
   either outcome. The adjudicator reopens the primary evidence itself before ranking either side's claim,
   and either names the governing rule that resolves the dispute or records it explicitly as
   `OPEN — DECISION REQUIRED` with: both parties' locators and dates, why the existing rules don't resolve
   it, a resolution test, a blocking flag, and an owner. An adjudicator may never adjudicate a dispute in
   which it authored either side's bundle.
4. **User (or accountable-principal) escalation.** If the conflict remains unresolved after adjudication,
   or if it turns out to require a genuine value or scope choice rather than a factual or quality
   determination, it is recorded and put to whoever has the actual authority to make that call — usually
   the end user or project owner, not any agent in the workflow. The orchestrator must not settle a
   value/scope question on its own authority, however confident it is about which answer the user would
   prefer.

**Illustrative pattern.** Picture a finding that survives one repair cycle because the owner records an
explicit "out of scope" disposition rather than a fix. That is exactly the trigger for step 2: the finding
gets reassigned to a different specialist track rather than sent back to the same owner a third time. Keep
this shape in mind — "the owner disagrees this is worth fixing" is not evidence the finding is wrong, and
it is not authorization to skip straight to closing it; it is the signal to reassign. This pattern recurs
across real gated workflows, which is why step 2 exists as a named step rather than an ad hoc judgment
call.

## 7. Step 5 — Non-blocking carry-forward, done honestly

`PASS WITH CONDITIONS` may allow work to advance past a gate only when **both** of these hold:

- every open condition on the verdict is `blocking: false`, and
- this specific gate's definition (§3) explicitly states that carry-forward is permitted, and for what
  category of defect.

When a condition does carry forward:

- it keeps its original id, its original owner, and its original resolution test;
- it must be tracked in one visible place — a single running table, not scattered across individual
  artifact files — so it cannot be quietly lost between gates;
- it gets **re-tested** at the next gate that touches the same artifact, not assumed resolved by
  inactivity; and
- the running total of blocking conditions across the whole program is reported honestly at every status
  checkpoint, attributed to whichever fresh verdict established that count — never asserted by the
  orchestrator on its own authority, and never left at a stale number after a fresh verdict changes it.

A carried-forward table with an owner and a destination for every row is the single artifact most worth
protecting in a long-running gated program: the discipline this whole methodology is defending against is
a defect getting fixed once, mentioned once, and then never re-verified or re-surfaced again.

## 8. Step 6 — The recursive research → draft → review cycle

Some workflows are not a clean waterfall through gates — they involve genuine open-ended investigation
where you don't know the shape of the final artifact until you've explored it. For that kind of work, run
this cycle instead of (or nested inside) the linear gate sequence:

```
research a slice of the problem
        │
        ▼
synthesize what was found
        │
        ▼
draft the artifact(s) that slice supports
        │
        ▼
adversarially review the draft
        │
        ├── review finds gaps or unsupported claims ──► return to research on exactly
        │                                                that gap, then redraft
        │
        └── review passes ──► the slice is done; move to the next slice, or to
                               whatever downstream gate depends on this artifact
```

Rules that make this cycle actually recursive rather than a one-way pipeline with a review step bolted on
at the end:

- **A review failure routes back to research or drafting on the artifact, never forward past the gate.**
  The reviewer is not asking "is this good enough to ship," it is asking "does every claim here trace to
  something verified, and is anything missing that the artifact's own scope requires."
- **Do the cycle per meaningful slice (per topic, per section, per component), not only once at the very
  end.** Running the full cycle once per slice, continuously, surfaces gaps while they're still cheap to
  research further. Saving the entire review for a single pass over the finished whole reproduces the
  waterfall problem this cycle exists to avoid.
- **A negative or "doesn't hold up" finding is a legitimate, expected outcome of research — not a failure
  of the process.** If a workflow only asks "how well does the evidence support candidate A" without ever
  entertaining "does candidate A actually survive scrutiny at all," it has built in confirmation bias by
  construction. Explicitly design your research step to be able to report "this doesn't hold, drop it" and
  treat that report as a success of the cycle, not a stall.
- **Every gate produced by a lap through this cycle still uses the verdict schema and failure routing from
  §4 and §6** — the recursive cycle changes *when* you review (continuously, per slice) and *what happens
  on failure* (loop back rather than stop), not the structure of the review itself.
- **Log every lap** — what was researched, what was synthesized, what was drafted, what the review found,
  and where it routed — to the same evidence ledger described in §9. A cycle that isn't logged cannot be
  distinguished later from a cycle that never actually ran.

## 9. Step 7 — The evidence ledger

Every substantive verdict, correction, or handoff should be accompanied by a structured evidence ledger
that makes its claims independently checkable later, without having to trust the author's summary of its
own work. Use this shape (see `references/evidence-ledger-template.md` for a blank copy):

```yaml
worker_identity: <who/what did this work, exactly — never "default" or unspecified>

files_consulted:
  - <exact locator opened> — <what was read there>
    claim_supported: <which specific claim in this verdict/report this evidence backs>

files_not_accessed:
  - <what was deliberately not opened, and why> — e.g. out of this task's write scope, not
    yet created at authoring time, or explicitly reserved to another task's authority

project_claims:
  - claim: <a specific factual claim this work makes>
    status: implemented_verified | implemented_unverified | unsupported_open
    locator: <where the verifying evidence lives>

external_sources: <anything fetched from outside the project, or "none">

conflicts_or_unknowns:
  - <anything left genuinely unresolved, stated plainly rather than smoothed over,
    with its governing rule if one exists and OPEN if none does>

handoff_notes: |
  <what the next reader of this artifact most needs to know — including what this pass
   deliberately did NOT do, and why, so nobody assumes silence means completion>
```

The two disciplines that make this worth the overhead:

- **Distinguish "verified by this task" from "carried from someone else's finding."** If you didn't open
  the primary evidence yourself, say so, and say whose finding you're relying on instead. Silently
  presenting an inherited claim as freshly verified is exactly the failure mode rule 2 in §5 exists to
  prevent, just moved into the evidence ledger instead of the verdict.
- **State what you did not do, as plainly as what you did.** "Files not accessed" and "conflicts or
  unknowns" are not admissions of failure — they are what makes the ledger trustworthy. A ledger with no
  unknowns in a genuinely uncertain project is a ledger someone stopped being honest in.

## 10. Step 8 — The correction log convention

Plans and gate definitions get things wrong, or go stale, as work proceeds. Never silently edit a
governing document to erase that history. Instead:

- Append a dated, numbered correction entry (`C-01`, `C-02`, ...) describing: what was wrong, the exact
  evidence that established it was wrong, what was changed, what was deliberately **not** changed (and why
  — usually because it belongs to another owner's artifact), and who made the correction.
- **Prefer rewriting an existing cell or line in place over inserting or deleting lines**, specifically so
  that existing citations to that document by line number don't silently go stale. Where a rewrite is
  unavoidable and line numbers do shift, switch future citations to that document to section-anchor
  references instead of line numbers, and say so in the correction entry.
- **A correction that repairs a defect is not the same statement as a correction that clears the finding
  which reported the defect.** Say explicitly which one you're doing. Clearing a finding is reserved to a
  fresh independent reviewer verdict (§5 rule 2) — a correction entry that fixes the underlying artifact
  should say "repaired; a fresh verdict must still confirm this" rather than declaring the finding closed
  on its own authority.
- **Never edit another task's or owner's artifact to make your own correction land, even when the
  discrepancy spans both.** Fix what you own, and explicitly route the other half to its actual owner in
  the same correction entry, naming the exact locator that needs the other repair.
- Keep a running count of unresolved corrections/findings visible at the top level of your governing
  status document, attributed to whichever verdict most recently established it — not silently updated by
  whoever happens to be writing the status line that day.

## 11. Reviewer capacity and independence

Match reviewer capability to the kind of judgment the gate actually requires, and keep three tiers
conceptually distinct even if your workflow only has two workers to assign them to:

- **Mechanical work** — schema conformance, cross-reference resolution, formatting, metadata population.
  Cheap, fast, highly parallelizable; a mechanical checker can run continuously without waiting for a
  human-grade reviewer's time.
- **Substantive single-artifact work** — reviewing one document, one component, one section for whether it
  actually supports what it claims, on its own terms. Needs sustained reading comprehension but not
  cross-document judgment calls.
- **Judgment and cross-artifact synthesis** — reconciling contradictions across many artifacts, adjudicating
  genuine disagreements, deciding whether a whole approach still holds up, final program-level sign-off.
  Reserve your most capable reviewers for this tier, and reserve this tier specifically for decisions that
  change a *rule* rather than apply an existing rule to one more instance.

Whatever tiers you use, hold two things constant: the reviewer at every tier must be a worker distinct
from the artifact's owner (§5 rule 3), and a review that changes the *governing rule itself* — not just
one artifact's compliance with it — belongs at your highest judgment tier, never delegated down because a
lower tier happens to be available.

## 12. Common failure modes to design against

- **The gate exists on paper but nothing actually checks it before downstream work proceeds.** A gate is
  only real if something enforces "artifact not gate-passed → cannot be depended on" mechanically or
  procedurally, not just documented as a should.
- **"PASS WITH CONDITIONS" quietly becomes "PASS."** Watch for status summaries that drop the "with
  conditions" the moment the conditions become inconvenient to keep repeating.
- **The orchestrator starts making severity calls.** The moment routing logic starts saying "this finding
  is probably fine, let's not block on it," the no-self-approval boundary has effectively been crossed by
  proxy — the orchestrator has become an unnamed, unaccountable reviewer.
- **Carried-forward conditions get lost between gates.** If there is no single running table, they will be.
- **A repaired defect gets treated as a cleared one** because the owner is confident, or because
  re-reviewing feels redundant. It is never redundant — that's the whole point of rule 2 in §5.
- **The two-strikes rule gets satisfied by asking the same worker more firmly.** Re-prompting is not
  reassignment. If you catch yourself about to send a third near-identical request to the same owner, stop
  and reassign instead.
- **Research-then-review only runs once, at the end, over the whole finished artifact**, instead of
  continuously per slice — which quietly turns the recursive cycle in §8 back into a waterfall with a
  review step bolted on.
- **A confirmatory research question ("how well does X hold up") is asked instead of a falsifiable one**
  ("does X hold up at all") — which structurally forecloses the negative finding described in §8.

## 13. Where to look next

- `references/gate-definition-template.md` — a blank copy of the gate template from §3, ready to fill in.
- `references/verdict-schema.md` — the verdict schema from §4 with field-by-field notes on why each field
  exists and what an incomplete verdict looks like.
- `references/evidence-ledger-template.md` — a blank copy of the evidence ledger from §9.
- `references/correction-log-convention.md` — the correction-entry format from §10 with a worked pattern.
- `examples/example-workflow-gates.md` — a fully worked, entirely synthetic four-gate content-pipeline
  example: gate definitions, a PASS WITH CONDITIONS verdict, a FAIL verdict that triggers failure routing,
  a two-strikes escalation, an adjudication, and two correction-log entries — showing the whole lifecycle
  running end to end on invented data.
