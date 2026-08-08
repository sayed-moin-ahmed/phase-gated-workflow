# phase-gated-workflow

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) that
enforces a disciplined five-mode engineering pipeline with **hard approval gates**
between phases — no phase begins until the user explicitly approves the previous
one.

```
RESEARCH  →  ⛔ STOP / WAIT  →  DESIGN  →  ⛔ STOP / WAIT  →  REVIEW
   →  ⛔ STOP / WAIT  →  IMPLEMENTATION  →  ⛔ STOP / WAIT  →  VALIDATION
```

## Why

Most bad engineering outcomes are not typos — they are confidently building the
wrong thing on a misread requirement, a wrong assumption, or an unstress-tested
design. This skill makes those mistakes cheap by catching them at the phase where
they are still a sentence in a plan rather than a diff to unwind. The gate is the
whole point: the user is the decision-maker at every boundary.

## The five modes

Each mode puts the agent in a specific **role** with a fixed objective, hard
"do NOT produce" boundaries, and a required deliverables list.

| # | Mode | Role | Produces |
|---|------|------|----------|
| 1 | **RESEARCH** | Principal Architect + Review Board | Multi-perspective analysis, alternatives, risks, open questions — no solution chosen, no code |
| 2 | **DESIGN** | Solution Architect / Design Authority | Buildable design, contracts described, numbered testable acceptance criteria (AC-1…N), rollout + rollback plan — no code |
| 3 | **REVIEW** | Adversarial Review Board / Red Team | Go / No-Go verdict + ranked required changes (cross-model critique via `triangulate`) |
| 4 | **IMPLEMENTATION** | Senior Implementation Engineer | The diff, mapped to the approved design — exactly what was approved |
| 5 | **VALIDATION** | QA / Verification Engineer + SRE | Empirical validation report: every acceptance criterion checked, CI checks reproduced, trajectory review, explicit PASS / BLOCKED / FAIL |

Key disciplines baked into the modes:

- **Stop and ask, never assume** — an ambiguity blocks the mode until resolved.
- **Acceptance criteria as a contract** — DESIGN writes numbered, mechanically
  checkable criteria; VALIDATION maps every one to a check. A criterion changed
  after approval voids its verification mapping.
- **Boundaries** — no code in RESEARCH/DESIGN, no fixes in REVIEW, no scope creep
  in IMPLEMENTATION, no silent patches in VALIDATION.
- **Trajectory review** — VALIDATION audits the *path*, not just the diff: was
  the actual problem solved, did an unexamined assumption survive, were
  unnecessary files touched, was a simpler shape missed.
- **CI/CD honesty** — local green is not CI green; an authoritative check that
  could not be run is **BLOCKED**, not PASS.
- **Completion contract** — the workflow only ends with an evidenced
  `STATUS: PASS`, an explicit `BLOCKED` (with exact commands to finish later),
  or a `FAIL` with root cause and the mode to loop back to.

## Install

```bash
git clone https://github.com/sayed-moin-ahmed/phase-gated-workflow.git \
  ~/.claude/skills/phase-gated-workflow
```

Claude Code picks it up automatically from `~/.claude/skills/`.

## Usage

The skill triggers only on **explicit gate language** — it is the execution
gate, not the entry point for every task:

> "phase-gate this", "gated mode", "go mode-by-mode", "phase by phase",
> "stop between phases", "research mode" … "validation mode"

Moving between modes:

- **Forward** only on explicit approval ("approved", "proceed", "go") — one gate
  per turn. Silence or vague enthusiasm is not approval.
- **Backward** anytime — a later mode revealing an earlier mistake is the
  process working, not failing.
- **Skipping** is the user's call ("skip review", "no gates") and is announced
  in the mode header so it stays visible.

## Repository layout

```
SKILL.md                        # conductor: gates, turn protocol, mode index
references/
├── research-mode.md            # mode 1 playbook (incl. intake + confirmation loop)
├── design-mode.md              # mode 2 playbook
├── review-mode.md              # mode 3 playbook
├── implementation-mode.md      # mode 4 playbook
└── validation-mode.md          # mode 5 playbook
```

`SKILL.md` only conducts; each reference file is the full playbook for its mode
and is loaded when that mode begins (progressive disclosure).

## Companion skills

- `triangulate` — cross-model adversarial critique (codex + Antigravity) used
  by REVIEW mode.
- `/verify` — end-to-end behavior-check discipline used by VALIDATION mode.

Both degrade gracefully if unavailable; each mode states which path it took.
