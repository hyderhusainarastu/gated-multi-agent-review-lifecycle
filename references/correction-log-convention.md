# Correction log convention

Governing documents (plans, gate definitions, status trackers) drift as work proceeds — a status changes,
a filename planned up front turns out different from what got built, a finding gets discharged, a count
needs updating. **Never silently rewrite history to make the document look like it was always right.**
Use a dated, numbered correction log instead.

## Format

Append entries to the end of the governing document (or a dedicated corrections section), never
interleaved invisibly into the original prose:

```markdown
### C-<NN> — <date> — <one-line description of what this correction fixes>

**Finding.** <What was wrong, stated precisely, with the exact evidence that established it was wrong —
quote or closely cite the finding that surfaced this, don't paraphrase it into something vaguer.>

**Disposition.** <What was changed, and why this option was chosen over the alternatives. If a defect
could have been fixed two different ways, say which was picked and why the other was rejected — that
reasoning is often what a later reader most needs.>

**What changed here.** <Exactly which cell, line, or section was rewritten — in place where possible,
so existing citations to this document by line number don't silently go stale. If a rewrite could not
avoid moving lines, say so explicitly and tell future citers to use section anchors instead.>

**What this correction does not do.** <Explicitly list what was NOT touched, and why — usually because
it belongs to another owner's artifact, or because clearing the underlying finding is reserved to a
fresh independent reviewer verdict, not to this correction. A correction that repairs an artifact is not
the same statement as a correction that clears the finding that reported the defect — say which one this
is.>

**Correcting party.** <Who made this correction, distinct from who authored the original text and from
who authored any prior correction — so drift in *authorship* of the corrections themselves stays
visible too.>
```

## Rules

1. **Rewrite in place where you can.** Editing an existing cell or sentence, rather than inserting or
   deleting lines around it, keeps every other citation to this document (by line number) valid. Only
   move lines when there's no honest alternative, and when you do, say so in the entry and tell future
   citers to switch to section-anchor references.
2. **Distinguish "repaired" from "cleared."** Fixing the underlying defect is something any qualified
   worker can do. *Verifying* that the fix actually resolves the finding, and closing the finding, is
   reserved to a fresh independent reviewer verdict (see SKILL.md §5, rule 2). A correction entry that
   blurs this distinction — declaring a finding "resolved" on its own authority — has quietly broken the
   no-self-approval rule by doing it inside a correction log instead of a verdict.
3. **Never edit another owner's artifact to make your correction land**, even when the discrepancy spans
   both. Fix what you own; name the exact locator of the other half and route it to its actual owner in
   the same entry, so it doesn't get lost between the two corrections.
4. **Keep a running blocking-condition count visible at the top of the governing status document,
   attributed to whichever verdict most recently established it.** When a correction changes that count,
   say so in both the correction entry and the status line it affects — and say who established the new
   count, not just what the new count is.
5. **A correction log with only ever-decreasing finding counts, and never a widened scope or a newly
   discovered issue, is suspicious.** Real corrective work on a live program surfaces new things as often
   as it closes old ones. If your log only ever shrinks, check whether findings are being closed on
   authority that should have required a fresh independent verdict first.

## Worked micro-pattern (synthetic, illustrative)

```markdown
### C-04 — 2026-03-02 — Task-id cross-reference repaired; adjudication route was undefined

**Finding.** Conformance check `CHK-07` (`reports/conformance_2026-03-01.md` ll.40-51): this plan's
routing section names task `ADJ-1` as the adjudicator, but no row for `ADJ-1` exists in `tasks.csv`.
Blocking; owner: plan-integrator.

**Disposition — reference kept, row created.** The finding permitted either deleting the reference or
creating the row. Deleting it was rejected: the adjudication route is mandated by this workflow's own
failure-routing policy (SKILL.md §6 step 3), and a role brief for `ADJ-1` already existed with no matrix
row to back it. Creating the row was the smaller, more honest gap to close.

**What changed here.** `tasks.csv` gained one new row (`ADJ-1`), appended rather than inserted, so every
existing `tasks.csv l.NN` citation elsewhere in this project stays valid.

**What this correction does not do.** It does not claim any adjudication actually occurred — `ADJ-1` was
never invoked in this program to date; this correction records that a route exists, not that it was used.
Nor does it clear `CHK-07` itself: that requires a fresh conformance re-run, not this entry's own word.

**Correcting party.** plan-integrator, fresh instance, 2026-03-02.
```
