# Strategy structure and handoff

Read in steps 4–5. This file is how to turn the context + research + answers into an approvable
strategy, and how to hand that strategy to the execution skill once the user says yes.

## Contents
- Strategy structure
- Carrying the security posture and UX weight
- Recording the decision
- Handoff: routing table
- Handoff: how to trigger with the strategy as input
- The boundary (what the downstream skill still does)

## Strategy structure

Present a strategy the user can act on — concise, honest, decision-ready. Cover:

- **Goal** — the idea restated as the problem to solve and the intent behind it.
- **Context & constraints** — what the project is (stack, conventions, recorded decisions) and the
  limits that bound the solution (from CLAUDE.md/memory/README and the codebase analysis).
- **Options considered** — the realistic alternatives, each with pros/cons, effort and risk, **backed
  by the research** (cite the evidence). Don't present a single option as if there were no choice.
- **Recommendation + rationale** — the option you'd pick and *why*, tied to the constraints and the
  evidence. This is the heart of the plan.
- **Scope** — what's in and, just as important, what's explicitly out for now.
- **Risks & unknowns** — the things that could go wrong or that remain uncertain, each with a
  mitigation; be explicit about residual uncertainty rather than hiding it.
- **Phased approach** — the sequence of phases/milestones, ordered so each is independently valuable
  and the riskiest assumptions are tested early.
- **Success criteria** — how the user will know it worked.
- **Open decisions** — anything still needing the user before execution can start.

Keep it proportional to the idea: a small idea gets a short strategy; a direction-setting one gets the
full treatment. The plan must be enough for the user to approve the "what" and the "why" before any
code is written.

## Carrying the security posture and UX weight

Carry the project's **security posture** (real app vs PoC/demo — the gate in
`../create-application/references/security.md`) into the strategy, and flag any part of the idea with
notable security weight (auth, payments, uploads, PII) or notable UX weight (new user-facing surface).
Why: the downstream execution skills gate their hardening and their UI/UX bar on exactly these signals,
so deciding them at the strategy level means the handoff carries the right defaults instead of
re-litigating them mid-build.

## Recording the decision

When the user asks (or when the decision is significant enough to outlive the conversation), record the
approved strategy so later work honors it — an ADR or note under `docs/`, a line in the project
`README`, or the project **memory** for cross-cutting decisions (posture, stack, direction). This is
the one write this skill makes, and only on request/agreement. Why: a decision that lives only in the
chat is lost the next session; recording it keeps the other skills aligned with it.

## Handoff: routing table

Map the approved strategy to the skill that executes it:

| The strategy is about… | Hand off to | What it owns |
|---|---|---|
| Starting a brand-new project (no codebase yet) | `create-application` | interview, stack, scaffold, build with tests |
| Adding/building one or more features in an existing app | `implement-feature` | clarify, plan, impact statement, build with tests |
| Restructuring/normalizing an existing app to the target practices | `refactor-application` | assess, phased restructure plan, execute, validate |
| Running or creating tests | `test-application` | detect/map coverage, create missing tests, run, report |

A large strategy can fan out across more than one (e.g., refactor first, then implement a feature) —
sequence the handoffs and tell the user the order. If the strategy calls for work outside these four
(an audit, a security/quality review), point the user at the appropriate skill they have rather than
forcing a fit.

## Handoff: how to trigger with the strategy as input

On approval, invoke the matching skill (via its slash command / trigger, e.g. `/implement-feature`) and
**pass the approved strategy as its input**, stating plainly what is already decided so the downstream
skill starts from the plan instead of from scratch:

- the **scope** (what's in/out) and the **build order** the strategy fixed,
- the **security posture** (so its PoC/demo gate is already answered),
- the **stack/approach** chosen and why,
- the **open decisions** that are now resolved (so it doesn't re-ask them).

Frame it as: "Plan approved — here is the strategy; proceed with `implement-feature` using this as the
plan." The downstream skill then runs its own flow from there.

## The boundary (what the downstream skill still does)

plan-strategy is the brainstorm/research/strategy layer; it does **not** absorb the execution skills'
responsibilities. The downstream skill still runs its **own** detailed planning (its slices, its task
list), its **impact analysis** (blast radius, before touching code) and its **tests** — informed by the
strategy, not replaced by it. Why: keeping the boundary clean means the strategy can stay
high-level and honest, while the execution skill remains the authority on the code-level plan, the risk
to existing behavior, and the green-test gate. Don't pre-empt those steps here.
