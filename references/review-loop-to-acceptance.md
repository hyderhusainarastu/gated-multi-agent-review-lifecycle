# Review loop to acceptance

A reusable loop for taking a draft artifact through independent review until it is accepted. It
sits on top of the gate lifecycle in `SKILL.md`: each review stage below is a gate, and every
verdict uses `verdict-schema.md`.

## 1. Stage skeleton

Run the stages in order. A stage may repeat, but a stage never starts before the previous one has
a recorded verdict.

1. **Draft.** The owner produces the artifact and freezes a version (a hash, tag or copy).
2. **Independent review.** One or more reviewers examine the frozen version and write findings.
3. **Triage.** The coordinator merges findings, removes duplicates, and assigns each a severity
   and a disposition (section 3).
4. **Fix.** The owner changes the artifact and records what changed against each finding.
5. **Verification.** A verifier who did not write the fix checks each finding against the revised
   version.
6. **Acceptance check.** The coordinator applies the acceptance proxy (section 4).
7. **Closure.** An independent verifier signs off, or the loop returns to stage 2.

## 2. Independence rule

- Reviewers see only the frozen version. They never see fixes in progress, fix notes or the
  owner's replies until they have submitted their findings.
- A reviewer's findings are written before they see any other reviewer's findings.
- The person or agent who made a fix never verifies it.
- A finding raised in one round may only be cleared by a reviewer in a later round who
  re-examines the revised text; an owner's statement that it is fixed does not clear it.

## 3. Severity and disposition taxonomy

Severity (how much the problem matters):

| Level | Meaning |
|---|---|
| Blocker | The artifact is wrong, unsafe or unusable. It cannot be accepted. |
| Major | A real defect that a reader would act on. Must be fixed or explicitly waived. |
| Minor | A real but low-impact defect. Fix when cheap. |
| Note | An observation or preference. No action required. |

Disposition (what happened to it):

| Disposition | Meaning |
|---|---|
| Fixed | Changed in the artifact; awaiting or past verification. |
| Verified | A verifier confirmed the fix against the revised version. |
| Rejected | The owner disagrees and gives a reason; a neutral adjudicator decides. |
| Deferred | Valid but out of scope; recorded with an owner and a revisit trigger. |
| Waived | Accepted risk, approved by whoever has the authority to accept it. |

Every finding ends in exactly one terminal disposition: Verified, Rejected-and-upheld,
Deferred or Waived.

## 4. Acceptance proxy

You cannot prove a document or plan is perfect, so use an observable stand-in. Accept when all of
these hold on the same frozen version:

- No open Blocker. No open Major unless it is Deferred or Waived with a named approver.
- A full review round of independent reviewers raised no new Blocker or Major.
- Every finding from the previous round has a terminal disposition.
- The count of new findings per round has fallen to Minor and Note only.

If the number of new Major findings is not falling after two rounds, stop iterating and escalate
(see failure routing in `SKILL.md`): the problem is usually the brief or the scope, not the draft.

## 5. Closure by an independent verifier

Closure is a separate step with its own verdict. The closer:

- is not the owner, not a fix author and not the coordinator who triaged;
- reads the final version and the finding ledger, and spot-checks Verified items against the text;
- confirms the version they checked is the version that will ship (same hash or tag);
- records `PASS`, `PASS WITH CONDITIONS` or `FAIL` using `verdict-schema.md`.

A change made after closure reopens the loop at stage 2 for the changed parts.

## 6. Minimal ledger row

```
finding | reviewer | severity | disposition | fixed in | verified by | version checked
```

Keep one row per finding in the evidence ledger (`evidence-ledger-template.md`).
