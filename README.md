# Gated Multi-Agent Review Lifecycle

A reusable methodology for orchestrating multi-step or multi-agent work so that a later stage can only
depend on an earlier stage's output once that output has been **independently verified** — not just
produced. It gives you the exact schemas and rules for structured pass/fail verdicts, how failures get
routed and escalated, why no worker may approve its own output, how to keep an auditable evidence trail,
and how to run an open-ended research-then-draft-then-review loop that doesn't collapse into a rubber
stamp under deadline pressure.

## What it does, in plain language

If you've ever run a pipeline where "agent A produces something, agent B is supposed to check it" and
watched the check quietly become a formality — the reviewer trusts the producer, or the producer marks
its own work done, or a real disagreement between two reviewers just gets absorbed into whoever wrote the
status update last — this skill is the fix. It defines:

- **Gates** — named checkpoints with an objective, entry/exit criteria, a named artifact, an owner, and
  an independent reviewer who is never the owner.
- **A structured verdict format** — `PASS` / `PASS WITH CONDITIONS` / `FAIL`, with every condition
  carrying a blocking flag, evidence, an owner, and the single test that would prove it's fixed.
- **Failure routing** — return the finding to its owner, reassign to a different specialist after two
  failed repair cycles on the same defect, route genuine disagreements to a neutral adjudicator, and
  escalate anything that turns out to be a value or scope choice to whoever actually has that authority.
- **A no-self-approval rule** — the same worker never both produces and independently reviews its own
  output, and only a *fresh* independent review can clear a previously flagged problem.
- **An evidence ledger and correction-log convention** — so every verdict, repair, and status update is
  independently checkable later, and governing documents get corrected transparently instead of silently
  rewritten.
- **A recursive research → draft → review cycle** — for open-ended work where a review failure should
  send you back to research on exactly the gap it found, not stop the pipeline; and where "this doesn't
  hold up" is treated as a legitimate finding, not a process failure.

It is a **template methodology**, not a fixed checklist — you define however many gates your own workflow
actually needs (three is plenty for a small pipeline; a large multi-phase program might define a dozen)
and fill in the provided templates with your own artifacts, roles, and criteria.

## Works with

| Surface | How | Status |
|---|---|---|
| Claude Code | native skill (`~/.claude/skills/` or `.claude/skills/`) | not tested |
| Codex | native skill via `.agents/skills` | not tested |
| Gemini CLI | native skill via `.agents/skills`, pointer in `GEMINI.md` | not tested |
| Cursor | native skill via `.agents/skills`, reads `AGENTS.md` | not tested |
| Copilot | pointer block in `.github/copilot-instructions.md` | not tested |
| ChatGPT / Claude.ai / Gemini (chat) | paste `PROMPT.md` | not tested |

## Installation

This is an Agent Skill: a `SKILL.md` file with YAML frontmatter plus reference templates. No
dependencies, no scripts beyond `install.sh`, no network access.

```sh
git clone https://github.com/hyderhusainarastu/gated-multi-agent-review-lifecycle
cd gated-multi-agent-review-lifecycle
./install.sh --claude                 # ~/.claude/skills/gated-multi-agent-review-lifecycle
./install.sh --agents --project .     # .agents/skills (Codex, Cursor, Gemini CLI)
./install.sh --copilot --project .    # pointer block in .github/copilot-instructions.md
./install.sh --dry-run --claude       # preview; add --uninstall to remove
```

For a chat UI with no skill support, paste `PROMPT.md` into the conversation. Instructions for
agents live in `AGENTS.md`; `CLAUDE.md`, `GEMINI.md` and `.github/copilot-instructions.md` point to it.

## Worked usage example (synthetic data)

`examples/example-workflow-gates.md` runs the whole lifecycle end to end on an entirely invented
four-gate content pipeline ("Aurora Field Notes," a fictional quarterly market-trends report — no real
project or organization). It shows:

1. Four gate definitions (source research → draft → editorial integration → publication readiness) and
   a gate-to-task index.
2. A `PASS WITH CONDITIONS` verdict where a non-blocking condition legitimately carries forward.
3. A `FAIL` verdict with a blocking condition, and the failure-routing step it triggers.
4. The same defect surviving two repair cycles by one worker, triggering the mandatory reassignment to a
   different specialist on the third attempt.
5. Two reviewers disagreeing about whether a claim is adequately sourced, routed to a neutral adjudicator
   who reopens the primary evidence itself and rules on which gate each reviewer's concern actually
   belongs to.
6. Two correction-log entries showing how a governing document gets fixed transparently — including the
   distinction between "repaired" and "cleared," which stays reserved to a fresh reviewer verdict.

To adapt it to your own workflow: copy `references/gate-definition-template.md` once per gate you need,
fill in your own artifacts and roles, and have every reviewer emit verdicts in the exact shape from
`references/verdict-schema.md`.

## Contents

```
SKILL.md                                    the complete, self-contained methodology
README.md                                   this file
AGENTS.md                                   agent instructions (CLAUDE.md, GEMINI.md and Copilot point here)
PROMPT.md                                   paste-in version for chat UIs
install.sh                                  installer for the skill targets above
LICENSE                                     MIT
references/
  review-loop-to-acceptance.md              stage skeleton, independence rule, severity taxonomy, closure
  gate-definition-template.md               blank gate template + a definition checklist
  verdict-schema.md                         the verdict schema, field by field, with anti-patterns
  evidence-ledger-template.md               blank evidence-ledger template
  correction-log-convention.md              correction-entry format + a worked micro-example
examples/
  example-workflow-gates.md                 full synthetic worked example (see above)
```

There is no bundled script besides the installer; this skill is pure methodology and templates, meant to be followed directly
by whatever agent or orchestrator is running your workflow.

## License

MIT. See `LICENSE`.

## Related

- [multi-agent-orchestration-playbook](https://github.com/hyderhusainarastu/multi-agent-orchestration-playbook): the umbrella guide for orchestrating multi-agent work.
- [evidence-ledger-and-phase-handoff-convention](https://github.com/hyderhusainarastu/evidence-ledger-and-phase-handoff-convention)
- [adversarial-peer-review-role](https://github.com/hyderhusainarastu/adversarial-peer-review-role)
- [agent-brief-schema-for-specialist-roles](https://github.com/hyderhusainarastu/agent-brief-schema-for-specialist-roles)

Keywords: quality gates, multi-agent review, verification, no self-approval, evidence ledger, agent skill.

Part of palimnote-skills - more at https://github.com/hyderhusainarastu
