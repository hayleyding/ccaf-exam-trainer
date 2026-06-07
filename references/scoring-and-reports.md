# Mastery Tracking, Reports, and Score Forecast

## Mastery state

Track every one of the 30 task statements with one of three states:

- `pending` — not yet tested
- `failed` — answered incorrectly at least once and not yet recovered; must be re-tested
- `mastered` — answered correctly (a `failed` point becomes `mastered` once a later
  re-test on a different sub-concept is answered correctly)

Because this runs in a chat, **render the tracker after every answer** so state is never
lost. Keep it compact:

```
Progress: 14/30 mastered
D1 ▓▓▓▓▓░░  5/7   D2 ▓▓▓░░  3/5   D3 ▓▓▓▓░░ 4/6   D4 ▓░░░░░ 1/6   D5 ▓░░░░░ 1/6
To revisit (failed): 2.2, 4.4
Up next: 4.1
```

Maintain, in your working notes for the session, a per-point log of: attempts, first-attempt
correct (yes/no), and current state. First-attempt correctness drives the forecast, so record
it even after a point is later mastered.

## Session flow

1. On first invocation, confirm the mode (see SKILL.md) and the order. Default order: walk
   domains 1→5 in blueprint order, but weight how many questions you draw toward the
   heavier domains if the learner picks "simulated exam".
2. Ask **one question at a time**. Wait for the answer (A/B/C/D). Accept letter or restated text.
3. Grade, explain, update state, render the tracker.
4. Insert failed points back into the queue a few questions later (not immediately) with a
   fresh question on a different sub-concept.
5. Continue until all 30 are `mastered`.

## Completion report (when all 30 are mastered)

Tell the learner clearly: **"You've now answered every point the exam is expected to cover
correctly at least once."** Then show:

```
# CCAF Coverage Report
All 30 task statements mastered ✅

First-attempt accuracy by domain (drives your forecast):
- D1 Agentic Architecture & Orchestration (27%, 16q on real exam): X/7  (xx%)
- D2 Tool Design & MCP Integration        (18%, 11q on real exam): X/5  (xx%)
- D3 Claude Code Config & Workflows       (20%, 12q on real exam): X/6  (xx%)
- D4 Prompt Engineering & Structured Out  (20%, 12q on real exam): X/6  (xx%)
- D5 Context Management & Reliability     (15%,  9q on real exam): X/6  (xx%)

Points that needed more than one attempt: <list ids + one-line theme each>
Recommended review before exam day: <2-4 concrete topics>
```

Keep "needed more than one attempt" honest — those are the weak spots worth revising.

## Re-test mode and the score forecast

If the learner wants another round after full coverage (or asks for a mock exam at any
point), regenerate **fresh** questions across the points (new scenarios, new angles). For a
practice exam or exam mode, default to **60 questions** drawn to domain weights (D1:16,
D2:11, D3:12, D4:12, D5:9) across 4 randomly selected scenarios from the full pool of 13,
mirroring the real exam format.

At the end of a re-test, give feedback **and a forecast scaled score**.

### Forecast formula (state that it's an estimate, not an official prediction)

1. For each domain, compute first-attempt accuracy `a_d` (fraction correct) on this round.
2. Weighted accuracy `W = 0.27·a1 + 0.18·a2 + 0.20·a3 + 0.20·a4 + 0.15·a5`.
3. **Forecast scaled score = round(100 + 900 · W)**, clamped to [100, 1000].
4. Passing is **720**. By this mapping, `W ≈ 0.69` is the pass line.

Report it like:

```
Forecast: ~<score>/1000   (pass line 720) → <Likely pass / Borderline / Likely below>
Weighted accuracy this round: <W as %>
Biggest drag on the score: Domain <n> (<accuracy>) — weighted at <weight>
Smallest gain for effort: lift Domain <n> from <x%> to <y%> ≈ +<points> scaled
```

Be straight about borderline results: within ~30 points of 720 is "borderline, revise the
flagged domains before booking". Don't inflate. The forecast is a study signal, not a promise.
