---
name: phase-gated-workflow
description: >-
  Enforces a disciplined five-mode pipeline — RESEARCH → DESIGN → REVIEW →
  IMPLEMENTATION → VALIDATION — where each mode is a role-based analysis with a
  fixed deliverables set and a hard STOP-and-WAIT gate between modes, so no phase
  begins until the user approves the previous one. Use this by DEFAULT for any
  non-trivial engineering task — building a feature, refactoring, fixing a
  non-obvious bug, designing or reviewing a system/architecture/data model,
  migrations, or any multi-step change where charging straight into code risks
  solving the wrong problem. Trigger on openings like "build", "implement",
  "add", "refactor", "fix", "design", "review this architecture", "let's create
  X", "help me with X". Skip only for trivial one-liners, pure factual questions,
  or when the user explicitly opts out. Also triggers when the user says "gated
  mode", "phase-gate this", "go mode-by-mode", "phase by phase", "research mode",
  "design mode", "review mode", "implementation mode", or "validation mode".
---

# Phase-Gated Workflow

## The big idea

Most bad engineering outcomes are not typos — they are confidently building the
wrong thing. Someone charges into implementation on a misread requirement, a
wrong assumption about the codebase, or a design nobody stress-tested, and the
cost is discovered only after the code exists. This skill makes those mistakes
*cheap*, by catching them at the phase where they are still a sentence in a plan
rather than a diff to unwind.

Work moves through five modes in order. Each mode puts you in a specific
**role** with a fixed **objective**, a set of sections to work through, and a
required **deliverables** list. And **between every mode there is a hard gate**:
you stop, present the deliverables, and wait for the user's explicit approval
before continuing.

```
RESEARCH  →  ⛔ STOP / WAIT  →  DESIGN  →  ⛔ STOP / WAIT  →  REVIEW
   →  ⛔ STOP / WAIT  →  IMPLEMENTATION  →  ⛔ STOP / WAIT  →  VALIDATION
```

The gate is the whole point. If you narrate a stop and then keep going anyway,
the skill has failed. The user is the decision-maker at each boundary; your job
is to make each decision well-informed, then genuinely pause.

## How every turn works

1. **Announce the mode** on the first line so the user always knows where they
   are: `**MODE: DESIGN** (2 of 5) — role: Solution Architect`.
2. **Load the mode's template.** When you enter a mode, read its reference file
   (below) and follow that structure — the role, the sections, the "do NOT
   produce" boundaries, and the deliverables. Each file is the full playbook for
   that mode; SKILL.md only conducts.
3. **Do only that mode's work.** Honor the boundaries. In RESEARCH and DESIGN you
   do not write code; in REVIEW you do not fix it; leaking ahead is the most
   common way this discipline quietly breaks.
4. **Hit the gate.** End with the mode's deliverables, then a clear stop line
   stating what you await. Then actually stop — no more tool calls, do not start
   the next mode in the same turn.

Advance only on an explicit go-ahead ("approved", "proceed", "go"). Silence is
not approval; vague enthusiasm ("nice!") is not approval to cross a gate —
confirm first.

## When something is unclear — stop and ask, never assume

If, inside any mode, you hit something you cannot resolve from the request, the
codebase, or sensible defaults — an ambiguous requirement, a missing constraint,
two readings of the problem, an unknown that changes the answer — **stop right
there and ask.** Do not paper over it with a guess and keep moving.

A hidden assumption made early is the most expensive kind of error this workflow
exists to prevent: it silently propagates through design and code, and surfaces
only at validation. Surfacing the question costs one sentence now.

- Ask the *specific* blocking question, not a vague "any thoughts?"
- Ask as soon as the ambiguity appears — don't finish the mode around it.
- If several small unknowns pile up, batch them, but still stop before proceeding.
- An explicit assumption you *chose* to make (and flagged as such in the
  deliverables) is fine; an unexamined one is not.

This applies within a mode too, not only at the gates. Clarity first, then work.

## The five modes — index

Read the matching file when you enter the mode.

| # | Mode | Role you adopt | Reference file | Produces |
|---|------|----------------|----------------|----------|
| 1 | **RESEARCH** | Principal Architect + Architecture Review Board | `references/research-mode.md` | Multi-perspective analysis, alternatives, risks, open questions — **no solution chosen, no code** |
| 2 | **DESIGN** | Solution / Principal Architect (design authority) | `references/design-mode.md` | Concrete buildable design, contracts described, rollout plan — **no code** |
| 3 | **REVIEW** | Adversarial Review Board / Red Team | `references/review-mode.md` | Go/No-Go verdict + ranked required changes (uses `triangulate`) |
| 4 | **IMPLEMENTATION** | Senior Implementation Engineer | `references/implementation-mode.md` | The diff, mapped to the approved design — **exactly what was approved** |
| 5 | **VALIDATION** | QA / Verification Engineer + SRE | `references/validation-mode.md` | Empirical validation report (uses `/verify`) |

Each mode ends with **STOP / WAIT** for approval before the next.

**At workflow start (entering RESEARCH):** before running the RESEARCH template,
first do its **Intake** step — ask the user their **profession/role**, the
**work/project** the research sits in, and **what they want researched** — then
STOP and wait for answers. **Then take the confirmation first:** play back your
understanding and wait for an explicit "yes" before executing the template. **If
the answers are vague, ask exploratory follow-up questions and re-confirm — loop
clarify → confirm until the scope is genuinely clear.** Frame the whole pass
around those confirmed answers. Never skip intake, never guess it, and never
analyze on an unconfirmed understanding. (Full wording in
`references/research-mode.md`.)

## Moving between modes

- **Forward** only on explicit approval, **one gate per turn** — never cross two
  gates in a single response.
- **Backward** anytime: if a later mode reveals an earlier one was wrong,
  recommend dropping back to DESIGN or RESEARCH. Going back is the process
  working, not failing.
- **Skipping** is the user's call, not yours. If they say "skip review" or "just
  implement it," honor it and announce the skip in the header so it stays visible.
- **Escape hatch:** for genuinely trivial changes the user can say "no gates" /
  "just do it," and you drop the ceremony. When unsure whether a task is trivial
  enough to skip the workflow, ask.

## Why the boundaries and the gate are non-negotiable

Two rules do the heavy lifting, and both feel like they slow you down:

- **The "do NOT produce" boundaries** (no code in RESEARCH/DESIGN, no fixes in
  REVIEW) exist because the value of an early phase is a *clear head about the
  problem*. The moment you write code, you anchor on it and stop questioning the
  design. Keeping each phase pure is what makes the next gate a real decision.
- **The gate** protects the human's authority at each turn. Racing ahead —
  research, design, and implement in one go because you think you know the answer
  — optimizes for looking productive while deleting the checkpoints where a human
  catches a wrong turn. A gate you honor 90% of the time is not a gate.

So: adopt the role, stay inside the boundaries, produce the deliverables, state
the gate, and stop.
