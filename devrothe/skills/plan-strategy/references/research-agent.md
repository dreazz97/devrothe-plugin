# The web-research subagent

Read before step 2. The planner stays in the main thread (it talks to the user); the **web research is
delegated to a subagent** so it can fan out across sources without flooding the planning context. This
file is how to brief that subagent and what to expect back.

## Contents
- When to dispatch (and when to skip)
- How to dispatch
- The brief
- Fan-out: one subagent per topic
- What the subagent must do
- What the subagent must not do
- The briefing it returns
- Graceful degradation (no web access)

## When to dispatch (and when to skip)

Dispatch when the idea has **external unknowns** — facts a source outside the codebase can answer:
technology/library choices and their maturity, recommended approaches and known pitfalls,
standards/specs, market or competitor landscape, pricing, compatibility/version constraints, security
advisories.

Skip (and say so) when the idea is **purely internal** — entirely answerable from the codebase and the
user's preferences. Don't send a subagent after what only the user knows (taste, budget, deadlines,
scope) — that belongs in the clarify step.

## How to dispatch

Use the Task/Agent tool to launch a subagent with web access (a general-purpose agent that can call
`WebSearch`/`WebFetch`). Run it in parallel with an initial clarify round when the user's answers don't
depend on the research. Portability note: the exact subagent type varies by environment — pick a
general-purpose research agent with web tools; if none exists, the planner does the web research inline
but keeps the same multi-source discipline below.

## The brief

Give the subagent enough to be useful without handing it the whole conversation:
- **The idea/goal**, in one or two sentences.
- **A short project-context summary** — stack, constraints and any recorded decisions (from CLAUDE.md/
  memory/README) that bound the answer, so it doesn't research irrelevant options.
- **The specific questions** for this topic — phrased as questions, not keywords.
- **The output contract** below (multi-source, current, cited, structured).

## Fan-out: one subagent per topic

When the "research the web" bucket has several **independent** topics, dispatch one subagent per topic
and run them in parallel — it's faster and keeps each briefing focused. Keep tightly coupled questions
in a single brief so the subagent can weigh them together. Don't over-fan: a handful of focused briefs
beats a swarm of one-question ones.

## What the subagent must do

- **Consult multiple independent sources** per question — not one blog post. Cross-check claims; prefer
  primary/authoritative sources (official docs, specs, maintainers, reputable benchmarks) over
  aggregators and SEO content.
- **Prefer current information** — note publication/update dates and library/tool versions; call out
  when something is recent, deprecated or fast-moving.
- **Flag conflicts and uncertainty** — when sources disagree, report the disagreement rather than
  picking silently; mark low-confidence findings.
- **Synthesize, then cite** — give the answer, then the evidence (source + date) behind it, so the
  planner can weigh and the user can verify.

## What the subagent must not do

- **Don't change files or run the project** — it is read-only research; no edits, no installs.
- **Don't make the user's decision** — it surfaces options and trade-offs with evidence; the planner
  and the user decide.
- **Don't pad with marketing fluff or restate the question** — evidence and synthesis only.
- **Don't present unsourced claims as fact** — flag anything from model knowledge as such.

## The briefing it returns

Expect, per question/topic:
- **Findings** — the answer, synthesized.
- **Sources** — links/titles with dates (and versions where relevant).
- **Trade-offs / options** — the realistic alternatives and how they compare.
- **Recommendation + confidence** — a leaning with a confidence level, when the evidence supports one.
- **Open gaps** — what couldn't be answered or needs the user's input.

The planner folds these into the clarify step (step 3) and the strategy (step 4) — surfacing the
options to the user and citing the evidence behind the recommendation.

## Graceful degradation (no web access)

If web access isn't available, the subagent (or the planner) must **say so explicitly**, fall back to
model knowledge **clearly flagged as unverified and possibly stale**, and turn the riskiest unknowns
into questions for the user. Never let a stale or unsourced guess pass as researched fact — the plan's
credibility depends on the evidence being real.
