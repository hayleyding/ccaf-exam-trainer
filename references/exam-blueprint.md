# CCAF Exam Blueprint — The Examinable Points

This is the authoritative checklist of everything the trainer must test. The unit of
mastery is the **task statement** (the items numbered like 1.1, 2.3, etc.). There are
**30 task statements** across 5 domains. A learner has "fully covered" the exam when
every one of these 30 points has been answered correctly at least once.

Each entry lists what the point tests and the key facts / anti-patterns to anchor
questions and distractors around. When re-testing a failed point, rotate to a
*different* sub-concept listed under that point — do not repeat the same question.

## Exam format facts (use for the simulated exam and the forecast)

- All questions are **multiple choice**: exactly one correct answer and three distractors.
- Questions are **scenario-based** — framed inside one of the 6 production scenarios.
- The real exam draws **4 of the 6 scenarios** at random.
- Scoring is a **scaled score from 100 to 1000**; the **passing score is 720**.
- Unanswered = incorrect; no penalty for guessing.

## Domain weightings (from the official guide — use these exact numbers for the forecast)

| Domain | Weight |
|---|---|
| 1. Agentic Architecture & Orchestration | 0.27 |
| 2. Tool Design & MCP Integration | 0.18 |
| 3. Claude Code Configuration & Workflows | 0.20 |
| 4. Prompt Engineering & Structured Output | 0.20 |
| 5. Context Management & Reliability | 0.15 |

(Note: some third-party sites list Domain 1 at ~25% and Domain 2 at ~20%. The official
guide values above are the ones to use.)

## The 6 scenarios (frame questions inside the relevant one)

1. **Customer Support Resolution Agent** — Agent SDK, MCP tools (get_customer, lookup_order, process_refund, escalate_to_human), 80%+ first-contact resolution, escalation.
2. **Code Generation with Claude Code** — slash commands, CLAUDE.md, plan mode vs direct execution.
3. **Multi-Agent Research System** — coordinator + subagents (search, analyse, synthesise, report), cited reports.
4. **Developer Productivity with Claude** — built-in tools (Read, Write, Bash, Grep, Glob), MCP servers, codebase exploration.
5. **Claude Code for CI/CD** — automated review, test generation, PR feedback, actionable output, low false positives.
6. **Structured Data Extraction** — extract from unstructured docs, validate with JSON schemas, handle edge cases.

---

## Domain 1 — Agentic Architecture & Orchestration (0.27)

**1.1 Agentic loops for autonomous task execution.** Continue the loop while `stop_reason` is `"tool_use"`, terminate on `"end_turn"`. Tool results are appended to conversation history so the model reasons over them. Model-driven tool choice vs pre-configured decision trees. Anti-patterns: parsing natural-language signals for termination, arbitrary iteration caps as the primary stop, checking for assistant text as a completion indicator.

**1.2 Multi-agent orchestration (coordinator-subagent).** Hub-and-spoke: coordinator manages all inter-subagent communication, error handling, routing. Subagents have isolated context (no automatic inheritance). Coordinator decides which subagents to invoke by query complexity; dynamic selection beats always running the full pipeline. Risk: overly narrow decomposition → incomplete topic coverage. Iterative refinement loops re-delegate when synthesis has gaps.

**1.3 Subagent invocation, context passing, spawning.** The `Task` tool spawns subagents; coordinator's `allowedTools` must include `"Task"`. Context must be passed explicitly in the prompt — no shared memory. `AgentDefinition` sets description, system prompt, tool restrictions. Spawn parallel subagents by emitting multiple `Task` calls in one response. Pass complete prior findings into a subagent's prompt; keep content separate from metadata (URLs, names, page numbers) for attribution.

**1.4 Multi-step workflows with enforcement and handoff.** Programmatic enforcement (hooks, prerequisite gates) vs prompt-based guidance. Deterministic compliance (e.g., identity verification before financial ops) needs programmatic gates — prompt instructions have a non-zero failure rate. Block downstream tools until prerequisites complete (e.g., block `process_refund` until `get_customer` returns a verified ID). Structured handoff summaries (customer ID, root cause, amount, recommended action) for human escalation.

**1.5 Agent SDK hooks for interception and normalization.** `PostToolUse` hooks run after the model decides to call a tool. They receive `(tool_name, tool_input, tool_output)` and can either return the original `tool_output` (pass-through) or return an override dict — including `{"blocked": True, "action": "escalate_to_human", ...}` — which is what the model sees instead. Returning `blocked: True` is the SDK mechanism for guaranteed policy enforcement (e.g., refunds over $500). Also used for normalising results before the model sees them (e.g., Unix → ISO 8601 timestamps, numeric status codes). Choose hooks over prompts when business rules require guaranteed compliance — prompt instructions have a non-zero failure rate.

**1.6 Task decomposition strategies.** Fixed sequential pipelines (prompt chaining) vs dynamic adaptive decomposition. Prompt chaining for predictable multi-aspect reviews (per-file then cross-file). Dynamic decomposition for open-ended investigation (map structure, find high-impact areas, build an adapting plan). Avoid attention dilution by splitting large reviews.

**1.7 Session state, resumption, forking.** `--resume <session-name>` continues a named session. `fork_session` branches from a shared baseline to explore divergent approaches. Inform a resumed session about file changes for targeted re-analysis. Starting fresh with a structured summary beats resuming with stale tool results.

## Domain 2 — Tool Design & MCP Integration (0.18)

**2.1 Effective tool interfaces.** Tool descriptions are the primary mechanism for tool selection; minimal descriptions cause unreliable selection among similar tools. Include input formats, example queries, edge cases, boundaries. Fix overlap by renaming + rewriting descriptions or splitting a generic tool into purpose-specific tools. Watch for keyword-sensitive system-prompt wording that overrides good descriptions.

**2.2 Structured error responses for MCP tools.** The MCP `isError` flag. Distinguish transient / validation / business / permission errors. Generic "Operation failed" blocks recovery. Return `errorCategory`, `isRetryable`, human-readable text. Mark business-rule violations non-retryable with a customer-friendly explanation. Distinguish access failures from valid empty results.

**2.3 Tool distribution and tool choice.** Too many tools (e.g., 18 vs 4-5) degrades selection reliability. Agents misuse out-of-specialisation tools. Scope each agent to its role; add limited cross-role tools for high-frequency needs (e.g., a `verify_fact` tool for the synthesis agent). `tool_choice`: `"auto"` (may return text), `"any"` (must call some tool), forced (`{"type":"tool","name":"..."}`).

**2.4 Integrate MCP servers.** Project scope `.mcp.json` (shared via VCS) vs user scope `~/.claude.json` (personal/experimental). Env-var expansion (e.g., `${GITHUB_TOKEN}`) keeps secrets out of VCS. All configured servers' tools are available at connection time. MCP resources expose content catalogs to reduce exploratory calls. Prefer existing community servers for standard integrations; enrich MCP tool descriptions so the agent doesn't fall back to built-ins like Grep.

**2.5 Built-in tools (Read, Write, Edit, Bash, Grep, Glob).** Grep = content search; Glob = path/name pattern matching; Read/Write = full file ops; Edit = targeted change via unique text match. When Edit can't find unique anchor text, fall back to Read + Write. Build understanding incrementally (Grep to entry points, then Read to follow imports) rather than reading everything upfront.

## Domain 3 — Claude Code Configuration & Workflows (0.20)

**3.1 CLAUDE.md hierarchy and modular organisation.** Hierarchy: user (`~/.claude/CLAUDE.md`), project (`.claude/CLAUDE.md` or root `CLAUDE.md`), directory-level. User-level settings are not shared via VCS. `@import` keeps files modular. `.claude/rules/` for topic-specific files vs one monolith. `/memory` shows which memory files are loaded.

**3.2 Custom slash commands and skills.** Project commands in `.claude/commands/` (shared via VCS) vs user commands in `~/.claude/commands/`. Skills live in `.claude/skills/` with `SKILL.md` frontmatter: `context: fork`, `allowed-tools`, `argument-hint`. `context: fork` isolates verbose/exploratory skill output from the main conversation. Personal skill variants go in `~/.claude/skills/`. Choose skills (on-demand) vs CLAUDE.md (always-loaded standards).

**3.3 Path-specific rules.** `.claude/rules/` files with YAML frontmatter `paths` globs load only when editing matching files, saving context/tokens. Glob rules (e.g., `**/*.test.tsx`) beat directory-level CLAUDE.md for conventions that span many directories.

**3.4 Plan mode vs direct execution.** Plan mode for large-scale, multi-approach, architectural, multi-file work; enables safe exploration before committing. Direct execution for simple, well-scoped changes. The `Explore` subagent isolates verbose discovery and returns summaries. Combine: plan to investigate, direct execution to implement. Complexity already stated in requirements → plan mode now, not "switch later if needed".

**3.5 Iterative refinement.** Concrete input/output examples beat prose when results are inconsistent. Test-driven iteration (write tests, share failures). Interview pattern (have Claude ask questions first). Bundle interacting issues in one message; fix independent issues sequentially.

**3.6 Claude Code in CI/CD.** `-p` / `--print` for non-interactive mode (prevents input hangs). `--output-format json` + `--json-schema` for machine-parseable findings. CLAUDE.md supplies CI context (testing standards, fixtures, review criteria). Session isolation: an independent reviewer instance beats the same session reviewing its own code. Include prior findings to avoid duplicate PR comments; supply existing tests to avoid duplicate test generation.

## Domain 4 — Prompt Engineering & Structured Output (0.20)

**4.1 Explicit criteria to cut false positives.** Specific categorical criteria beat vague instructions ("flag when claimed behaviour contradicts actual behaviour" vs "check comments are accurate"). "Be conservative" / "only high-confidence" do not improve precision. High false-positive categories erode trust in accurate ones. Define explicit severity criteria with code examples; temporarily disable a noisy category while fixing it.

**4.2 Few-shot prompting.** The most effective technique for consistently formatted, actionable output when instructions alone are inconsistent. Demonstrates ambiguous-case handling, generalises judgement to novel patterns, reduces extraction hallucination. Use 2-4 targeted examples that show *reasoning* for choosing one action over plausible alternatives; demonstrate desired output format and varied document structures.

**4.3 Structured output via tool use + JSON schemas.** `tool_use` with JSON schemas is the most reliable path to schema-compliant output and eliminates JSON syntax errors. `tool_choice` `"auto"` vs `"any"` vs forced. Strict schemas remove syntax errors but not *semantic* errors (line items not summing, wrong field). Make fields optional/nullable when source may lack them, to stop fabrication. Use enums with `"unclear"` / `"other"` + detail for extensibility.

**4.4 Validation, retry, feedback loops.** Retry-with-error-feedback: append the specific validation error on retry. Retries fail when the info is simply absent from the source (vs format/structural errors which they fix). Track `detected_pattern` to analyse false-positive dismissals. Self-correction flows: extract `calculated_total` alongside `stated_total`; add `conflict_detected` for inconsistent source data. Distinguish semantic validation errors from syntax errors (the latter eliminated by tool use).

**4.5 Batch processing.** Message Batches API: ~50% cost savings, up to 24-hour window, no latency SLA. Good for non-blocking, latency-tolerant work (overnight/weekly); bad for blocking workflows (pre-merge checks). Batch API does **not** support multi-turn tool calling in one request. `custom_id` correlates request/response and identifies failures to resubmit. Refine prompts on a sample before processing large volumes.

**4.6 Multi-instance / multi-pass review.** A model retains generation reasoning, so it under-questions its own output in the same session. Independent review instances (no prior reasoning context) beat self-review instructions or extended thinking. Split large reviews into per-file passes + a cross-file integration pass to avoid attention dilution and contradictory findings. Larger context windows do **not** fix attention-quality issues.

## Domain 5 — Context Management & Reliability (0.15)

**5.1 Preserve critical info across long interactions.** Progressive summarisation risks losing numbers, percentages, dates, customer-stated expectations. "Lost in the middle": models process the start and end of long inputs reliably but may drop the middle. Tool results consume tokens disproportionately. Extract a persistent "case facts" block included in each prompt; trim verbose tool outputs to relevant fields; place key summaries at the start with explicit section headers.

**5.2 Escalation and ambiguity resolution.** Escalate on: explicit customer request for a human, policy gaps/exceptions, inability to make progress. Honour explicit human requests immediately (no investigation first). Acknowledge frustration but offer resolution when in-scope, escalating only if the customer reiterates. Sentiment and self-reported confidence are unreliable proxies for complexity. Multiple customer matches → ask for more identifiers, don't guess.

**5.3 Error propagation across multi-agent systems.** Return structured error context (failure type, attempted query, partial results, alternatives) so the coordinator can recover intelligently. Distinguish access failures from valid empty results. Generic statuses ("search unavailable") hide context. Anti-patterns: silently suppressing errors (empty-as-success) and terminating the whole workflow on one failure. Subagents recover locally for transient failures and only propagate what they can't resolve.

**5.4 Context in large codebase exploration.** Extended sessions degrade: models give inconsistent answers and cite "typical patterns" instead of specific classes found earlier. Use scratchpad files to persist findings; delegate verbose exploration to subagents; summarise a phase before spawning the next. Crash recovery via structured state exports (manifests) the coordinator reloads. `/compact` reduces context use during long exploration.

**5.5 Human review workflows and confidence calibration.** Aggregate accuracy (e.g., 97%) can mask poor performance on specific document types/fields. Use stratified random sampling to measure error rates and catch novel patterns. Calibrate field-level confidence with labelled validation sets. Validate accuracy by document type and field before automating high-confidence extractions; route low-confidence/ambiguous cases to humans.

**5.6 Provenance and uncertainty in multi-source synthesis.** Source attribution is lost when summarisation drops claim-source mappings. Require structured claim-source mappings (URLs, names, excerpts) preserved through synthesis. Annotate conflicting statistics with attribution rather than picking one. Require publication/collection dates so temporal differences aren't read as contradictions. Structure reports to separate well-established from contested findings; render content types appropriately (financial as tables, news as prose).
