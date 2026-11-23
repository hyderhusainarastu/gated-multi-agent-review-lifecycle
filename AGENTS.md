# gated-multi-agent-review-lifecycle

An Agent Skill: a gate lifecycle for multi-agent or multi-step work, with verdict templates and a review loop.

## Using this in your project

- Use it when one agent's output must be independently verified before later work depends on it.
- Read `SKILL.md` first. It is self-contained; `references/` holds copy-paste templates.
- Define your gates with `references/gate-definition-template.md`. Every gate has an owner and an independent reviewer who is never the owner.
- Reviewers emit verdicts in the shape of `references/verdict-schema.md`: `PASS`, `PASS WITH CONDITIONS` or `FAIL`. Each condition carries a blocking flag, evidence, an owner and the test that proves it fixed.
- Route failures to the owner. After two failed repairs of the same defect, reassign to a different specialist. Send real disagreements to a neutral adjudicator.
- Never let a worker approve its own output. Only a fresh independent review clears a flagged problem.
- Record verdicts and repairs in the ledger (`references/evidence-ledger-template.md`) and the correction log (`references/correction-log-convention.md`).
- For iterative work, follow `references/review-loop-to-acceptance.md`: reviewers see only the frozen version, and an independent verifier closes.
- The skill writes nothing by itself. Put filled-in gates, verdicts and ledgers in your own project, not in this skill's folder.

## Working on this repo

- Install locally with `./install.sh --claude` or `./install.sh --agents --project DIR`; use `--dry-run` first.
- There is no build and no test suite; check that every file named in `SKILL.md` section 13 exists.
- Keep `SKILL.md` frontmatter `description` at 300 characters or fewer.
- Do not add venue, person or project names to examples; keep them synthetic.
