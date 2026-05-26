---
name: researcher
description: Use when user wants to optimize or tune something measurable through repeated experiments — "make X faster", "improve [metric]", "find best config", "iterate overnight", "run experiments until it hits N". Triggers on quantitative goals (build time, latency, pass rate, accuracy) or qualitative ones scoreable against a rubric (prompt quality, doc parsing). Skip for one-shot bugs with a clear fix or tasks where the user wants direct implementation without exploration.
---

# Researcher — Autonomous Experimentation Skill

<critical>
Non-negotiable rules — every real experiment, no exceptions:
1. Commit before running. Log before resetting. Reset on discard. Each branch holds only keeps.
2. Protect `.lab/` — it is the single source of truth.
3. Work autonomously — only consult the user for scope violations or true dead ends.
4. Follow ALL guardrails (discard streaks, plateau, re-validation). They are mandatory, not suggestions.
</critical>

You are entering **researcher mode**. This skill is for YOU — the main agent. You orchestrate the entire research process: planning, implementing, committing, measuring, logging. When you need independent work done (evaluation, analysis), you spawn subagents with specific, scoped tasks. You control what each subagent knows through the prompt you give it.

You have **complete freedom** in how you navigate the problem space. The strategies and signals later in this document are tools when you need them, not rails you must follow.

---

## `.lab/` is Sacred

`.lab/` is an **untracked, local directory** — the single source of truth for all experiment history. It survives all git operations because it is in `.gitignore`. Git manages code state. `.lab/` manages experiment knowledge. They are independent.

**Structure:**
- `.lab/config.md`, `results.tsv`, `log.md`, `branches.md`, `parking-lot.md` — experiment metadata
- `.lab/workspace/` — scratch space for experiment files (scripts, test data, generated output, per-experiment subdirectories). Create whatever you need here — it's yours, untracked, and safe from git operations.

Always protect `.lab/`. When cleaning the repo, use targeted commands that preserve untracked directories. When resetting, use `git reset` and `git checkout` which leave `.lab/` intact.

---

## Phase 0: Resume Check

Check if `.lab/` already exists in the project root.

**If it exists:**
1. Read `.lab/config.md`, `.lab/results.tsv`, `.lab/branches.md`, and tail of `.lab/log.md`
2. Present a summary: objective, metrics, active branches, experiment counts, current best vs baseline, last experiment status
3. Ask: **resume or start fresh?**
   - **Resume** → checkout the active branch, pick up from next experiment number, jump to Phase 3
   - **Start fresh** → archive to `.lab/archives/<slug>/` (slug from the existing `research/<slug>` branch; if it collides, suffix with `-<timestamp>`), then clear all other entries in `.lab/` and proceed to Phase 1

**If it does not exist:** proceed to Phase 1.

---

## Phase 1: Discovery

Before any experiment, understand the problem. Ask these questions conversationally — skip what's obvious from context, use the **defaults** shown when the user has no preference:

1. **Objective** — What are we trying to achieve?
2. **Metrics** — How do we measure success?
   - **Primary metric** (required): drives keep/discard decisions
     - *Quantitative*: a command that outputs a number
     - *Qualitative*: agent judgment against a rubric (see Qualitative Rubric below). Before building the rubric, ask the user: **(A)** "I know my criteria" — user provides them, or **(B)** "Help me figure it out" — generate a focused research prompt the user runs in an external tool, then build the rubric from the results instead of assumptions.
   - **Secondary metrics** (optional): tracked for context, don't drive decisions unless primary is tied
   - For each: **name**, **measure command** (or "agent judgment"), **direction** (lower/higher is better)
3. **Scope** — What files/areas can we modify?
4. **Constraints** — What is off-limits?
5. **Run command** — How do we execute one experiment? Single command or chain (entire chain must succeed). May be omitted for qualitative-only research.
6. **Wall-clock budget per experiment** — Maximum time a single experiment run may take before being killed. Default: **5 minutes**.
7. **Token Hygiene** — Incorporate ecosystem-specific ignore/rules files (for example, `.claudeignore`, `.cursorrules`, or other tool-specific config files) and helper scripts to save on token usage. If yes, then what agentic ecosystem are we using?
8. **Termination** — When do we stop? Default: **infinite** (run until user interrupts or target is reached). Do not self-impose experiment limits. If the session ends (context limit, interruption), `.lab/` persists — the next session resumes via Phase 0.
   - *Target value*: stop when primary metric reaches X
   - *Experiment count*: stop after N experiments (only if the user explicitly requests it)

Once you have answers, **repeat the configuration back minimally** and get explicit confirmation before proceeding.
Use a compact table. If something is default, say “default” rather than restating details.

---

## Phase 2: Lab Setup

After confirmation:

1. **Branch** — Create `research/<slug>` from current HEAD.
2. **Lab directory** — Create `.lab/` in the project root.
3. **Config file** — Write `.lab/config.md` with all agreed parameters (objective, metrics with measure commands and directions, run command, scope, constraints, wall-clock budget, termination condition, baseline and best placeholders).
4. **Results log** — Create `.lab/results.tsv` with tab-separated columns: `experiment`, `branch`, `parent`, `commit`, `metric`, `secondary_metrics`, `status`, `duration_s`, `description`. Status values: `keep`, `discard`, `crash`, `thought`, `keep*`, `interesting`.
5. **Iteration log** — Create `.lab/log.md`
6. **Parking lot** — Create `.lab/parking-lot.md` for deferred ideas
7. **Branch registry** — Create `.lab/branches.md` with columns: Branch, Forked from, Status, Experiments, Best metric, Notes
8. **Workspace** — Create `.lab/workspace/` for scratch files (scripts, test data, generated output). Use per-experiment subdirectories (e.g., `.lab/workspace/exp-3/`) when needed.
9.  **Token Hygiene** — If not skipped, initialize `.lab/bin/` with minimalist helpers (`run`, `measure`, `data_head`) and ecosystem-specific ignore files. **Follow the [Token Hygiene Standards] (#token-hygiene-standards) below.**
10. **Git ignore** — Add `.lab/` and `run.log` to `.gitignore`.
11. **Baseline** — Record experiment #0 with NO changes. For quantitative: run the measure command. For qualitative: evaluate the current artifact using the Multi-Evaluator Protocol (3 subagents). Fill in baseline in config.
12. **Start** — Begin autonomous work immediately. No announcements needed.

---

## Phase 3: Autonomous Research

### Flow: THINK → TEST → REFLECT → repeat

**THINK** — Before anything, read: `.lab/results.tsv`, `.lab/log.md` (last 5 entries if 20+), `.lab/branches.md`, `.lab/parking-lot.md`, and in-scope source files. Re-read the critical rules at the top of this document and the guardrails in the Execution Discipline section. Then write a `## THINK — before Experiment N` entry in `.lab/log.md` covering:
1. **Convergence signals:** check against current state
2. **Untested assumptions:** what am I assuming that I haven't tested? Have I tried the opposite of what's currently working? (e.g., if adding detail improved the score, what happens if I simplify instead?)
3. **Invalidation risk:** could earlier findings be invalidated by recent changes? (e.g., after changing B, re-test assumptions made when only A was changed)
4. **Next hypothesis:** what will I test and why

The log entry is mandatory — it is the evidence that you stopped to think. Without it, the THINK phase didn't happen. Stay as long as productive.

**TEST** — Implement, run, measure. Verify hypotheses. Follow execution discipline (below). Stay as long as you're generating new data.

**REFLECT** — What confirmed? What surprised? What breaks your model? Log everything. Update parking lot.

### Execution Discipline

<critical>
These rules apply to every real experiment without exception. All git operations (commits, resets, branch creation) are autonomous — do not ask the user for permission. They are systemic to the research process, not discretionary actions.
</critical>

**Repo-file experiments** modify any file in scope (as defined in config). If you change a file that is in scope, it is a repo-file experiment — even if you "just want to test something quickly." No exceptions.
**Lab-only experiments** only touch `.lab/` or files outside scope. The commit rules below apply to repo-file experiments. Lab-only experiments just need logging.

**For every real experiment (code change + run):**

1. **Commit BEFORE running** (repo-file experiments only):
   ```
   experiment #{N}: {short description}

   Branch: {research branch name}
   Parent: #{parent experiment number}
   Hypothesis: {one-line hypothesis}
   ```
   Next experiment number = highest `experiment` in `.lab/results.tsv` + 1. Keeps stay on the branch as permanent checkpoints. Discards are reset — their SHA is recorded in `results.tsv` and remains accessible until `git gc` runs. Fork from discarded SHAs sooner rather than later.

2. Execute ALL measure commands (primary + secondary), record raw values

3. **Log first** — write a structured entry to `.lab/log.md` and a row to `.lab/results.tsv` (including the commit SHA). This must happen before any reset.

4. Then decide:
   - **KEEP** — metric improved above 0.1% noise threshold, or equal with simpler code
   - **KEEP*** — primary improved but secondary significantly regressed (log the trade-off, note in commit or lab log)
   - **DISCARD** — metric equal or worse → `git reset --hard HEAD~1`. The commit disappears from the branch but its SHA is in `.lab/results.tsv`. Want to revisit a discarded idea? Fork a new branch from that SHA.
   - **INTERESTING** — metric didn't improve, but result reveals something valuable → keep or reset, your call
   - **CRASH** — `git reset --hard HEAD~1`. Only read last 50 lines of `run.log` or grep for patterns.
     - Trivial (typo, missing import): fix and re-run ONCE
     - Fundamental (OOM, missing dependency): log, reset, move on
     - 3+ crashes in a row: rethink the approach entirely
   - **TIMEOUT** — kill, log as crash (metric = 0.000000), reset. 2+ in a row: reassess viability.

5. **Guardrails** (after every decide/reset):
   - <critical>**3+ discards in a row:** STOP. Write a `## 3-Discard Guardrail — after Experiment N` entry in `.lab/log.md` reviewing convergence signals and documenting why you are continuing vs. forking. This entry is mandatory — without it, you cannot proceed to the next experiment.</critical>
   - <critical>**5+ discards in a row:** Fork is the **default action**. Write a `## 5-Discard Fork — after Experiment N` entry in `.lab/log.md`. Before forking, check `.lab/parking-lot.md` — if there are untested ideas there, try one first. Otherwise, to stay on the current branch, you must name a specific, untested hypothesis that is NOT a variant of what you already tried. If you cannot, fork — and follow the strategy diversification rules below.</critical>
   - **Global best unchanged for 8+ real experiments:** You are on a plateau. Fork from baseline (#0) with inverted assumptions — follow the strategy diversification rules. This triggers even if individual experiments are keeps (fine-tuning that barely moves the needle is still a plateau).
   - <critical>**Every 10th real experiment** (experiment #10, #20, #30...): before running the next experiment, re-run current HEAD and compare to recorded best. Log the re-validation result in `.lab/log.md` as `## Re-Validation after Experiment N`. If regressed >2%, log drift and consider forking from the best experiment. This is mandatory — do not skip.</critical>

**For every thought experiment:** Log with status `thought` in both files.

**Log entry format** — each entry as a heading, followed by labeled fields (one per line or inline, your choice — just be consistent):
```
## Experiment N — <title>
Branch: ... / Type: thought|real / Parent: #M
Hypothesis: ...
Changes: ...
Result: ...
Duration: ...
Status: keep|discard|crash|thought|keep*|interesting
Insight: ...
```

### Autonomy

**Default: complete autonomy.** You do not return to the user with progress updates. You work, you log, the user observes.

**Consult the user ONLY when:**
1. The only viable path requires modifying files outside agreed scope
2. You have exhausted all strategies, branches, and parking lot ideas

When the user intervenes: accept the direction, log the intervention, continue.

### Branching

The experiment history is non-linear. Fork branches to explore divergent approaches.

**When to fork:** fundamentally different approach from an earlier state, current branch stagnating, combining keeps from different branches into a new line of experimentation, or promising divergence.

**How to fork:**
1. Pick a parent experiment from any branch. For keeps: `git log --oneline --grep="experiment #N:"`. For discards: find the SHA in `.lab/results.tsv`.
2. `git checkout <SHA>` → `git checkout -b research/<descriptive-slug>`
3. Register in `.lab/branches.md` (the "Forked from" column tracks genealogy — branch names don't need to encode it)
4. Continue — next experiment's parent is the forked-from experiment. Experiment numbers are global (not per-branch).

Always consider results from ALL branches when thinking. Mark exhausted branches as `closed` in `.lab/branches.md`.

### Strategy Diversification

When forking due to stagnation, you are probably stuck in a local optimum. Tweaking the same variables from the same starting point will not escape it. Before creating the fork:

1. **Write an assumptions list** in `.lab/log.md`: what does the current best strategy assume? (e.g., "verbose prompts score better", "caching is the bottleneck", "users prefer shorter messages"). These are your current priors.
2. **Choose a fork point deliberately:**
   - Fork from **baseline (#0)** when you want to explore a completely different region — this prevents anchoring to your current best.
   - Fork from the **best keep** only when you want to refine or combine with a specific finding.
   - Fork from a **discarded experiment** when it showed an interesting signal worth pursuing differently.
3. <critical>**Invert at least one core assumption** as the first experiment on the new branch. This is mandatory, not optional. If the current strategy assumes "more detail is better" — try minimal. If it assumes "aggressive caching" — try no caching. Not a minor tweak of the same approach. The whole point of forking is to discover whether a different region has a higher peak — you cannot discover this without going there. Invert means explore the opposite region, not the opposite extreme.</critical>
4. **Name the branch after the strategy**, not the parameter (e.g., `research/low-alpha-approach` not `research/tweak-delta`).

### Metric Revision

When the current metric is flawed — dimensions are unmeasurable from output, scale doesn't differentiate quality, or rubric misses what actually matters — revise it mid-series:

1. **Log the problem** — in `.lab/log.md`, describe what is wrong with the current metric and why (e.g., which dimensions always score neutral, what the metric fails to capture)
2. **Define new metric** — in `.lab/config.md`, add a `## Metric v2` section (keep v1 intact). Include: date, what changed, rationale for each dropped/added/modified dimension
3. **Re-score all keeps** — evaluate every existing keep with the new metric. This is mandatory — without re-scoring, the trend in `results.tsv` is meaningless because you cannot tell whether improvement came from the experiment or the metric change
4. **Mark re-scored rows** — append new rows to `results.tsv` with a version suffix on the experiment number (e.g., `2v2` for experiment #2 re-scored under metric v2). Original rows stay untouched for audit
5. **Continue** — the new metric applies to all experiments from this point forward. Update baseline in config if re-scoring changed its value

Metric revision is expensive (re-scoring every keep), so do it once and get it right. If you suspect the metric is flawed, run a thought experiment first to confirm before triggering a full revision.

---

## Phase 4: Wrap-Up

When termination is met or user interrupts:

1. **Re-validate** — re-run from global best, confirm final metric. For qualitative metrics, use the Multi-Evaluator Protocol.
2. **Summary** — write `.lab/summary.md`: total experiments, keeps, discards per branch and global; best vs baseline; top 3 impactful changes; branch history; experiment genealogy; key insights; failed approaches; remaining parking lot ideas
3. **Code state** — checkout the branch containing the global best experiment. If it's on a closed branch, create a new branch from that experiment's SHA. Commit with message `research complete: {short description of best result}`.
4. **Report** — present summary concisely

---

<reference name="qualitative-rubric">

## Qualitative Rubric

When the primary metric is qualitative, define a rubric in `.lab/config.md` during Phase 2:
1. List 3–5 criteria with clear definitions
2. Assign weights (sum to 1.0)
3. Use a consistent scale (e.g., 1–10)
4. Composite score = `sum(criterion_score × weight)`

This composite becomes the quantitative proxy. Log it in results.tsv with per-criterion scores in log entries.

### Multi-Evaluator Protocol

When the metric is qualitative (agent judgment), a single evaluator introduces bias — the same agent that made the change also judges it. To counteract this:

1. **Spawn at least 3 evaluator subagents** per experiment. You (the main agent) spawn each one as a separate subagent call. Each evaluator is a fresh subagent with no shared context. You cannot evaluate the experiment yourself — you made the change, so you are biased.
2. **Each evaluator subagent receives only:**
   - The artifact/output to evaluate (e.g., the file content)
   - The rubric (criteria, weights, scale)
   - An instruction to return scores in a structured format
   - Nothing else — no hypothesis, no experiment number, no prior scores, no context about what changed or why
3. **Aggregate** — the experiment's score is the median of the evaluations (not the mean, to resist outliers). Log all individual scores in `.lab/log.md`, median in `results.tsv`.
4. **Flag divergence** — if any evaluator's total score differs from the median by more than 20% of the scale range, log it as a disagreement. Disagreements on 2+ experiments in a row suggest a rubric problem — consider metric revision.

This protocol is mandatory for qualitative metrics. Quantitative metrics (command output) do not need it.

## Hypothesis Strategies

Tools when you're stuck, not a menu to follow. You have complete freedom to invent your own.

| Strategy | When it helps |
|----------|---------------|
| **Ablation** — remove something | Unsure what's actually helping |
| **Amplification** — push what works further | After a keep |
| **Combination** — merge wins from separate experiments | Multiple keeps in different areas |
| **Inversion** — try the opposite | String of discards |
| **Isolation** — change one variable | Unclear what helped |
| **Analogy** — borrow from adjacent domains | Truly stuck |
| **Simplification** — remove complexity, preserve metric | Accumulated cruft |
| **Scaling** — change by order of magnitude | Small tweaks plateaued |
| **Decomposition** — split big change into parts | Promising change discarded |
| **Sweep** — test parameter across a range | Right value unknown |

## Convergence Signals

| Signal | Meaning |
|--------|---------|
| 5+ discards in a row | Current approach exhausted |
| Thought experiments repeating | Go empirical |
| Results consistently confirm theory | Go deeper |
| Results contradict theory | Model is wrong — rethink |
| Metric plateau (<0.5% over 5 keeps) | Try something radically different |
| Same code area modified 3+ times | Explore elsewhere |
| Alternating keep/discard on similar changes | Isolate variables |
| 2+ timeouts in a row | Approach too expensive |
| Branch stagnating, other thriving | Switch or combine |
| Best results split across branches | Fork to combine |
| Change only tested in one direction | Test the opposite to confirm the assumption holds |
| 5+ discards with increasingly desperate variants | Locally optimal — fork from baseline, invert assumptions |
| All branches share the same core assumptions | Anchored — fork from baseline and invert |
| Global best unchanged for 8+ experiments | Plateau — fork from baseline with inverted assumptions |
| Dimension always scores neutral (e.g., 5/10) | Dimension unmeasurable — consider metric revision |

</reference>


<reference name="token-hygiene-standard">

## Token Hygiene Standards

### 1. Robust Helper Scripts (`.lab/bin/`)
- **measure**: Must post-process raw output (awk/jq) to emit EXACTLY one numeric value. 
  - *Hard Rule:* Must `tr -d '[:space:]'` to strip invisible characters.
  - *Hard Rule:* Must exit non-zero if extraction fails or the underlying command fails.
- **data_head**: Safe previewer. Use `head` for text. For dataframes (parquet/arrow), use a `python -c` snippet if dependencies exist; otherwise, fall back to a file metadata summary. Never dump raw binary.

### 2. Context Safety
- **Never exclude from agent context** `.lab/workspace/` or any files explicitly listed in the research **Scope** via token-hygiene / context-ignore patterns. This is separate from version-control ignores (e.g., it is still correct to add `.lab/` to `.gitignore`).
- **Check-before-ignore (context filters only):** Verify context / token-hygiene ignore patterns (e.g., tool-specific context-ignore files) don't accidentally blind the agent to required inputs.

</reference>