---
name: plan-strategy
description: Turns a raw idea into a researched, decision-ready strategy — a more autonomous alternative to plan mode. Acts as the planning agent: reads the project's CLAUDE.md, memory and README and analyzes the codebase for context, brainstorms with the user by asking the questions needed to remove ambiguity, and dispatches a web-research subagent that gathers valid, current information across multiple sources and compiles it back. Synthesizes context, research and answers into a concrete strategy (options, trade-offs, recommendation, scope, risks, phases) for approval; on approval, hands off to the matching devrothe skill (implement-feature, create-application, refactor-application, test-application) so execution flows from the plan. Use when the user wants to research, brainstorm or plan an idea before building. Triggers on "plan-strategy", "help me plan", "research and plan", "brainstorm this idea", and in Portuguese "planeia isto", "ajuda-me a planear", "pesquisa e planeia", "brainstorm desta ideia".
---

# plan-strategy

Turn the user's idea into a concrete, decision-ready strategy. This skill is the **planning agent**: it
grounds itself in the project's reality, brainstorms with the user, delegates web research to a
dedicated subagent, and synthesizes everything into a plan the user can approve — then hands that plan
off to the right execution skill. It is meant to be a more autonomous, better-served alternative to the
built-in plan mode: it does the research and the questioning for you instead of one-shotting a plan.

**Rule: plan, don't build.** This skill produces a strategy — it does **not** write or change code,
install dependencies, or touch files (beyond optionally recording the decided strategy as a doc/note
when the user asks). Why: planning must stay cheap to iterate and free of side effects so the user can
explore freely; the moment the plan is approved, a dedicated execution skill takes over and does the
building under its own safeguards. Mixing planning and building defeats the point of a plan-mode
replacement.

**Rule: context before ideas.** Always start from what the project actually is — read its `CLAUDE.md`,
the project memory and the `README`/docs, and analyze the codebase (a full analysis when the idea
reaches across the app). Why: a strategy that ignores the existing stack, conventions, constraints and
prior decisions is fiction; the plan has to build on what is really there, and the recorded preferences
(memory, CLAUDE.md) are exactly the context that makes the plan fit this project rather than a generic
one.

**Rule: delegate the web research to a subagent.** Dispatch a dedicated **web-research subagent** (via
the Task/Agent tool, with web access) to gather valid, current information for the idea's open
questions and compile a briefing back; keep the planning role in the main thread so it can run the
interactive Q&A with the user (see `references/research-agent.md`). Why: research is a read-heavy,
parallelizable fan-out that would otherwise flood the planning context, while the back-and-forth with
the user can't be delegated to a subagent — so split them: the subagent reads the web, the planner
talks to the user and decides.

**Rule: research must be multi-source and current.** The research has to cross-check several
**independent** sources, prefer current/dated information, capture versions, flag conflicts and
uncertainty, and cite where each finding came from. Why: a plan is only as trustworthy as the evidence
under it — a single unverified or stale source quietly steers the whole strategy wrong.

**Rule: ask whatever it takes to remove doubt.** Brainstorm with the user — surface options and their
trade-offs and ask the clarifying questions needed before committing to a direction; never guess on a
decision that changes the strategy. Why: the goal is to serve the user better than a one-shot plan
mode, and that means converging on what they actually want (constraints, priorities, taste) instead of
assuming it.

**Rule: hand off, don't duplicate.** On approval, route to the matching devrothe skill and pass the
approved strategy as its input, stating what is already decided (see
`references/strategy-and-handoff.md`). Why: plan-strategy is the higher-level brainstorm/research/
strategy layer; the execution skills (`implement-feature`, `create-application`, `refactor-application`,
`test-application`) own their own detailed planning, impact analysis and tests — feed them the strategy
so they don't redo discovery from scratch, but let them do their job rather than re-deciding it here.

## Workflow

Copy this checklist into the response and tick items off:

```
- [ ] 0. Context — read CLAUDE.md, memory, READMEs/docs; analyze the codebase (full analysis if the idea is broad); note the security posture and constraints
- [ ] 1. Frame — restate the idea/goal; list the open unknowns, split into "ask the user" vs "research the web"
- [ ] 2. Research — dispatch the web-research subagent(s) across multiple independent sources; get a compiled, cited briefing (can run in parallel with an initial clarify round)
- [ ] 3. Clarify — brainstorm with the user: present options/trade-offs (informed by the research) and ask the questions needed to remove ambiguity
- [ ] 4. Strategy — synthesize context + research + answers into a concrete strategy (options, recommendation + rationale, scope, risks, phases, success criteria); present for approval; iterate 2–4 as needed
- [ ] 5. Hand off — on approval, trigger the matching devrothe skill (implement-feature / create-application / refactor-application / test-application), passing the strategy as input
```

### 0. Project context

Before anything, read the metadata that explains the project and its preferences:
- `CLAUDE.md` at the root and in subfolders, and other agent instruction files (`AGENTS.md`,
  `.cursor/rules/`, `.github/copilot-instructions.md`).
- The project **memory** — preferences and decisions already recorded, which are first-class planning
  context.
- Text files: `README*`, `docs/`, `CONTRIBUTING.md`, `ARCHITECTURE.md`, ADRs and notes.

Then analyze the codebase to the depth the idea demands: a narrow idea needs only the area it touches;
a broad one (a new subsystem, a direction shift) warrants a full read of the structure, stack and
existing patterns. Also note the project's **security posture** (real app vs PoC/demo — see the gate in
`../create-application/references/security.md`), since it shapes the strategy and is carried into the
handoff. Why: the strategy must respect what is already there and what the user/team has already
decided. If there is no project yet (a brand-new idea), say so — the strategy will likely route to
`create-application`.

### 1. Frame the idea

Restate, in your own words, the idea and the goal behind it, so the user can confirm you understood the
intent — not just the words. Then list the **open unknowns** and split them into two buckets:
- **Ask the user** — preferences, constraints, priorities, taste, budget, deadlines, scope boundaries
  (things only the user can answer).
- **Research the web** — facts about technologies, libraries, approaches, market/competitors,
  standards, pricing, known pitfalls (things an external source can answer).

Why: separating "ask" from "research" is what makes the next two steps efficient — you don't waste the
user's time on what the web can answer, nor send the subagent after what only the user knows.

### 2. Research

Dispatch the **web-research subagent** for the "research the web" bucket — one brief per independent
topic when there are several, run in parallel for breadth. Brief it with the idea, a short project-
context summary and the specific questions; require it to consult **multiple independent sources**,
prefer current/dated information, flag conflicts and return a structured, cited briefing with a
confidence level (full instructions in `references/research-agent.md`). This step can run in parallel
with an initial clarify round (step 3) when the user's answers don't depend on the research.

If the idea has no external unknowns (purely internal to the codebase), skip the research and say so.
If web access isn't available, the subagent must say so and fall back to flagged model knowledge plus
questions for the user — never present unsourced guesses as fact.

### 3. Clarify

With the research briefing in hand, **brainstorm with the user**: present the realistic options and
their trade-offs (now backed by evidence), and ask the clarifying questions needed to remove ambiguity,
using `AskUserQuestion`. Cover scope boundaries, priorities, constraints, and any decision where the
research surfaced a genuine fork. Why: this is the conversation that a one-shot plan mode skips — it is
where the plan becomes *theirs*. Don't guess on a decision that changes the design; iterate with the
research subagent if a new unknown surfaces that the web should answer.

### 4. Strategy

Synthesize the project context, the research and the user's answers into a **concrete strategy** the
user can approve (structure in `references/strategy-and-handoff.md`): the restated goal, the
constraints, the options considered with their trade-offs, a clear **recommendation with rationale**,
the scope (in/out), risks and unknowns with mitigations, a phased/sequenced approach, and success
criteria. Carry the **security posture** and any notable UX weight into the strategy so the downstream
skill honors them. Be honest about residual uncertainty rather than papering over it.

Present it for approval. Iterate steps 2–4 as needed — more research, more questions, a revised
recommendation — until the user is satisfied. If the user asks, record the decided strategy (e.g., in
`docs/`, an ADR, or the project memory) so later work can honor it.

### 5. Hand off

Once the user approves, route to the matching devrothe skill and pass the approved strategy as its
input, explicitly stating what is already decided (posture, stack, scope, order) so the downstream skill
does not re-ask it (routing table and mechanics in `references/strategy-and-handoff.md`):

- **Brand-new project** → `create-application`
- **Add/build feature(s) in an existing app** → `implement-feature`
- **Restructure/normalize an existing app** → `refactor-application`
- **Test work (run/create tests)** → `test-application`

The downstream skill still runs its own detailed planning, impact analysis and tests — plan-strategy
feeds it the strategy, it doesn't replace those steps. If the user prefers to stop at the plan, leave
the strategy recorded and don't trigger anything.

## References

- **`references/research-agent.md`** — how to brief and run the web-research subagent: scope, the
  multi-source/current/cited requirement, fan-out per topic, the briefing format the planner expects,
  what the subagent must not do, and graceful degradation without web access. Read before step 2.
- **`references/strategy-and-handoff.md`** — how to structure the strategy and how to hand it off:
  the plan layout, carrying the security posture, recording the decision, the routing table to the
  other devrothe skills and how to trigger them with the strategy as input. Read in steps 4–5.
- **Reused from other skills**: `../create-application/references/security.md` (the PoC/demo gate that
  sets the security posture carried into the plan); the downstream skills' own SKILL.md files for what
  each expects as input.
