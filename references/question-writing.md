# Writing CCAF-style Questions

## Format (match the real exam exactly)

- One scenario-framed stem, then **four options A–D**: exactly one correct, three distractors.
- The stem describes a realistic production situation (logs, metrics, a symptom, a goal) and
  asks for the *best* decision — not just a definition. Aim for "what should you do / what is
  the most likely root cause / which approach is most effective".
- Keep each question tied to one of the 6 scenarios in the blueprint. Pick the scenario whose
  primary domains include the point being tested.
- Difficulty: the learner should need practical judgement, not recall. Distractors must be
  things "a candidate with incomplete knowledge might choose" — plausible but wrong.

## Distractor design — use the exam's known anti-patterns

The strongest distractors are the real anti-patterns the exam punishes. Build wrong options
from this list (correct stance in brackets):

**D1 — Agentic Architecture**
- Parsing natural language to decide loop termination. [Check `stop_reason`.]
- Arbitrary iteration caps as the primary stop. [Let the loop end on `end_turn`.]
- Prompt-based enforcement for critical business rules. [Use programmatic hooks/gates — prompts are probabilistic.]
- Sentiment-based escalation. [Escalate on objective criteria: policy gaps, capability limits, explicit requests.]
- Self-reported confidence scores for routing/escalation. [Use structured criteria and programmatic checks.]

**D2 — Tool Design & MCP**
- Generic error messages ("Operation failed"). [Return `isError`, `errorCategory`, `isRetryable`, context.]
- Silently suppressing errors / empty-as-success. [Distinguish access failures from empty results.]
- Too many tools per agent (18+). [Keep ~4-5 scoped tools per agent.]
- Hardcoding API keys in `.mcp.json`. [Use `${ENV_VAR}` expansion — never commit secrets.]

**D3 — Claude Code Config**
- Personal preferences in project-level CLAUDE.md. [Use `~/.claude/CLAUDE.md` for personal settings.]
- Using slash commands for complex tasks that need context isolation. [Use skills with `context: fork` and `allowed-tools`.]
- Same-session self-review in CI/CD. [Use separate sessions for generation and review.]

**D4 — Prompt Engineering**
- Vague instructions like "be thorough" or "only flag high-confidence". [Provide explicit, measurable criteria.]
- Assuming tool_use guarantees semantic correctness. [tool_use guarantees structure only — validate values separately.]
- Generic retry messages without specifics. [Append the specific error: field name, expected vs actual.]

**D5 — Context & Reliability**
- Progressive summarisation of critical customer details. [Use immutable case facts blocks at context start.]
- Aggregate-accuracy-only metrics. [Segment accuracy by document type/field — aggregate masks category failures.]
- No provenance tracking for multi-agent data. [Track source, confidence, timestamp, and agent ID for all data.]
- Bigger context window to fix attention dilution. [Split into focused per-file passes + integration pass.]
- Resuming with stale tool results. [Start fresh with a structured summary.]

**General over-engineering traps**
- Consolidating/over-engineering (routing classifiers, new ML models) when a low-effort fix
  (better tool descriptions, explicit criteria, few-shot examples) addresses the root cause.

A good four-option set usually pairs the correct root-cause fix with: one "right idea, wrong
layer" option (prompt instead of hook, few-shot instead of description fix), one over-engineered
option, and one that blames the wrong component.

## Each question must declare which point it tests

Internally tag every question with its task-statement id (e.g., `1.4`) so mastery tracking is
exact. Surface this to the learner only in the explanation, not the stem.

## After each answer, always explain

State the correct answer, why it is correct, and — briefly — why each distractor is wrong.
This mirrors the practice exam and is where most learning happens. Keep it tight.

## Worked example (style reference — write fresh ones, don't reuse verbatim)

**Point tested:** 1.4 (programmatic enforcement). **Scenario:** Customer Support Resolution Agent.

> Stem: Logs show that in ~10% of refund cases the agent calls `process_refund` before
> `get_customer` has verified the account, occasionally refunding the wrong customer. What is
> the most effective fix?
>
> A) Add a prerequisite gate that blocks `process_refund` until `get_customer` returns a
> verified customer ID. ✅
> B) Strengthen the system prompt to say verification is mandatory before any refund.
> C) Add few-shot examples showing the agent verifying first.
> D) Add a sentiment check and escalate angry customers to a human.
>
> Correct: A. Financial correctness needs a deterministic gate; prompt (B) and few-shot (C) are
> probabilistic and will still fail some fraction of the time. D solves an unrelated problem.

**Point tested:** 4.5 (batch processing). **Scenario:** Claude Code for CI/CD.

> Stem: You want to cut cost on two workloads: a blocking pre-merge security check and an
> overnight tech-debt report. A teammate suggests moving both to the Message Batches API.
>
> A) Move only the overnight report to batch; keep the pre-merge check synchronous. ✅
> B) Move both to batch with status polling.
> C) Keep both synchronous to avoid result-ordering issues.
> D) Move both to batch with a real-time fallback on timeout.
>
> Correct: A. Batch gives ~50% savings but up to a 24-hour window with no latency SLA — fine
> overnight, unacceptable for a blocking check. C wrongly assumes ordering is a problem
> (`custom_id` handles it); B and D add complexity to force batch where it doesn't fit.

## Variety on re-tests

When a point was failed and is being re-tested, choose a *different* sub-concept from that
point in the blueprint and ideally a different scenario, so the learner demonstrates real
understanding rather than memorising one item.
