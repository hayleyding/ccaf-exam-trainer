# CCAF Exam Trainer

A personal Claude Code skill that drills you for the **Claude Certified Architect – Foundations (CCAF)** exam. It generates scenario-based multiple-choice questions, tracks your mastery of every examinable point, re-tests anything you get wrong, and forecasts a scaled exam score.


---

## Installation

You need [Claude Code](https://claude.ai/code) installed. Then clone this repo into your Claude skills directory:

```bash
git clone https://github.com/hayleyding/ccaf-exam-trainer.git ~/.claude/skills/ccaf-exam-trainer
```

That's it. The skill is available immediately in any Claude Code session — no restart needed.

---

## How to use it

Open a Claude Code session in any directory and invoke the skill:

```
/ccaf-exam-trainer
```

Claude will ask you to pick a mode:

| Mode | What it does |
|---|---|
| **Full coverage run** (default) | Walks all 30 examinable points until every one is mastered, then shows a coverage report |
| **Single domain** | Drills one of the 5 domains |
| **Simulated exam** | A mock exam drawn to the real domain weights, ending with a forecast score |

If you just say "quiz me", it defaults to full coverage.

---

## How the drill works

- One question at a time. Each is scenario-framed (4 of the 6 production scenarios from the real exam), with one correct answer and three plausible distractors.
- After you answer, Claude grades it, explains the correct answer, and tells you why each distractor is wrong.
- **If you got it wrong**, Claude pauses and asks if there's anything you want clarified before moving on. You give the greenlight (e.g. "next", "ok", "ready") when you're ready for the next question.
- A compact progress tracker is shown after every answer so you always know where you stand.
- Any point you get wrong is re-queued a few questions later with a *different* sub-concept, so you can't pass by memorising one answer.

---

## Scenarios

The official exam guide lists 6 scenarios; the real exam draws 4 at random. However, community reports from people who have sat the real exam indicate the actual pool may contain up to 13 scenarios — questions outside the official 6 have appeared on the live test.

This skill covers all 13: the 6 from the official guide plus 7 additional ones discussed by the community (Agentic Tool Design, Long Document Processing, Claude for Operations, Conversational AI Patterns, Agent Skills for Enterprise Knowledge Management, Agent Skills for Developer Tooling, Agent Skills with Code Execution). Mock exams draw from the full pool so you're not caught off guard.

---

## Exam facts

- **Format:** multiple choice, one correct + three distractors, scenario-based
- **Scaled score:** 100–1000, passing score is **720**
- **Domains and weights:**

| Domain | Weight |
|---|---|
| 1. Agentic Architecture & Orchestration | 27% |
| 2. Tool Design & MCP Integration | 18% |
| 3. Claude Code Configuration & Workflows | 20% |
| 4. Prompt Engineering & Structured Output | 20% |
| 5. Context Management & Reliability | 15% |

---

## Score forecast

After a full coverage run or simulated exam, the skill calculates a forecast scaled score using the domain weights and your first-attempt accuracy per domain. It's a study signal, not an official prediction — within ~30 points of 720 is "borderline, revise flagged domains before booking".

---

## Repo structure

```
SKILL.md                        — skill definition and loop instructions
references/
  exam-blueprint.md             — the 30 examinable points (source of truth for what gets tested)
  question-writing.md           — how questions and distractors are constructed
  scoring-and-reports.md        — mastery states, progress tracker format, forecast formula
```

---

## Choosing a model

The skill defaults to **Haiku** for fast responses during drill sessions. You can switch at any time with `/model` before invoking the skill:

| Model | When to use |
|---|---|
| `claude-haiku-4-5-20251001` | Default — fastest, great for drilling |
| `claude-sonnet-4-6` | Richer explanations if you want more depth |
| `claude-opus-4-8` | Most thorough — worth it for deep dives on weak areas |

Run `/model` without arguments to see all currently available versions.

---

## Tips

- The skill reads its files fresh each invocation — edits to any reference file take effect immediately on the next `/ccaf-exam-trainer` call.
- Focus on the "needed more than one attempt" list in the coverage report. Those are your weak spots.
