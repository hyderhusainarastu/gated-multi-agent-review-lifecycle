# Gate definition template

Copy this block once per gate into your workflow's governing plan document. Fill in every field —
an empty or vague field is the most common way a gate quietly stops meaning anything.

```text
## Gate <letter-or-number> — <name>

Objective:
  <One or two sentences. What must be true in the world for this gate to have passed? Write this
   as a state of affairs, not a task description — "the source is verified and citable," not
   "verify the source.">

Entry criteria:
  <What must already exist, or already be true, before this gate can even be attempted? List each
   as a checkable item. If a prior gate is a prerequisite, name it explicitly.>
  1. ...
  2. ...

Exit criteria:
  <The enumerated conditions that TOGETHER define "passed." Each one must be testable by the
   independent reviewer without asking the owner "did you do this?" — the reviewer should be able
   to check it directly against the artifact.>
  1. ...
  2. ...
  3. ...

Named artifacts:
  <The exact file(s), record(s), or deliverable(s) this gate governs, with their real locations
   or identifiers. "The report" is not sufficient once more than one report can exist.>

Owning task / role:
  <Who produces the artifact(s) above.>

Independent reviewer:
  <Who reviews it. MUST be a different worker from the owning task/role. State their exact
   identity or role, not "someone qualified" — an unnamed reviewer is not a reviewer.>

Non-blocking carry-forward:
  <PERMITTED, and for exactly which category of defect (be specific: "wording and cross-reference
   precision," not "minor issues") — OR — NOT PERMITTED, stated with the reason (usually because
   this gate touches safety, scope, legal/compliance, or an irreversible external commitment, and
   any open condition there is blocking by construction).>
```

## Checklist before you consider a gate "defined"

- [ ] Every exit criterion is something a reviewer can test directly, not something they have to take
      the owner's word for.
- [ ] The named artifacts have real, exact locations — not descriptive placeholders.
- [ ] The independent reviewer is a specific, different worker from the owner — never "TBD," never the
      owner itself, never the orchestrator.
- [ ] The carry-forward policy is stated explicitly for this gate, not inherited silently from "the usual
      rule" — write it out even if it matches every other gate's policy.
- [ ] This gate appears as one row in your workflow's gate-to-task index (gate, owning task(s),
      independent reviewer(s)) so a reader can audit the whole set at a glance for self-review or
      orchestrator-as-reviewer violations.
