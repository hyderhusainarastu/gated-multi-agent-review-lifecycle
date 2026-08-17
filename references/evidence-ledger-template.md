# Evidence ledger template

Attach one of these to every substantive verdict, correction, or handoff in a gated workflow. Its
purpose is to let anyone later re-verify a claim without re-trusting the author's summary of their own
work — the same discipline the verdict schema applies to reviewers, applied to every worker.

```yaml
worker_identity: <who/what did this work, exactly — never "default" or left unspecified>

files_consulted:
  - <exact locator opened> — <what was read there, in enough detail that a re-reader could find
    the same passage again>
    claim_supported: <which specific claim in this verdict/report this evidence backs — quote or
    closely paraphrase the reviewer's/author's own finding, don't summarize it away>

files_not_accessed:
  - <what was deliberately not opened> — <why: out of this task's write/read scope, did not yet
    exist at authoring time, reserved to another task's exclusive authority, or judged irrelevant
    to this specific claim and say why>

project_claims:
  - claim: <a specific factual claim this work makes>
    status: implemented_verified | implemented_unverified | unsupported_open
    locator: <where the verifying evidence for this specific claim lives>

external_sources: <anything fetched from outside the project/workflow's own artifacts, with
  locators and access dates, or the literal word "none" if nothing was>

conflicts_or_unknowns:
  - <anything left genuinely unresolved by this pass, stated plainly> — <its governing rule, if
    one already exists, or "OPEN" with an owner if none does yet>

handoff_notes: |
  <What the next reader of this artifact most needs to know, in plain prose. Explicitly state what
   this pass did NOT do and why — silence is the thing that gets misread as completion later.>
```

## The three status values for `project_claims`, defined precisely

- **`implemented_verified`** — this worker itself opened the primary evidence and directly confirmed the
  claim. Not "someone told me this is true" — this worker checked it.
- **`implemented_unverified`** — the claim is very likely true and is carried from another party's own
  verified finding, but this worker did not itself reopen the primary evidence this pass. State whose
  finding it's carried from and where.
- **`unsupported_open`** — the claim is not yet backed by evidence this worker can point to, or is
  explicitly *not* being asserted by this pass even though it might be true. Use this status rather than
  silently omitting a claim that a careless reader might otherwise assume was verified.

## Two disciplines worth protecting deliberately

1. **Never present an inherited finding as freshly verified.** If you didn't personally reopen the primary
   evidence this pass, your `project_claims` entry must say `implemented_unverified` and name whose
   verification you're relying on — even when you are quite sure the inherited finding is correct.
2. **An evidence ledger with no `conflicts_or_unknowns` entries, on a genuinely complex piece of work, is a
   red flag, not a clean bill of health.** Real work on non-trivial problems leaves real unknowns. A ledger
   that reports none is usually one where the author stopped looking for them, not one where none existed.
