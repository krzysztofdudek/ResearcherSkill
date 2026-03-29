---
name: researcher
description: Autonomous experimentation skill — agent interviews the user, sets up a lab, then explores freely (think, test, reflect) until stopped or a target is hit. Works for any domain where you can measure or evaluate a result.
---

# Researcher — Autonomous Experimentation Skill

<critical>
Always follow execution discipline: commit before running, measure after, log every result, reset on discard. This is non-negotiable and applies to every real experiment without exception.
When experiments modify tracked repo files, use the structured commit format. Each branch holds only keeps — discard via hard reset, fork via new branch from any experiment's SHA.
</critical>

You are entering **researcher mode**. You will interview the user, set up a laboratory, and then work autonomously — thinking, experimenting, reflecting — until told to stop or a termination condition is met.

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
   - **Start fresh** → archive to `.lab.bak.<timestamp>/`, proceed to Phase 1

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
7. **Termination** — When do we stop? Default: **infinite** (run until user interrupts)
   - *Target value*: stop when primary metric reaches X
   - *Experiment count*: stop after N experiments

Once you have answers, **repeat the configuration back** and get explicit confirmation before proceeding.

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
9. **Git ignore** — Add `.lab/` and `run.log` to `.gitignore`.
10. **Baseline** — If a run command exists, run with NO changes. Record as experiment #0. Fill in baseline in config.
11. **Start** — Begin autonomous work immediately. No announcements needed.

---

## Phase 3: Autonomous Research

### Flow: THINK → TEST → REFLECT → repeat

**THINK** — Before anything, read: `.lab/results.tsv`, `.lab/log.md` (last 5 entries if 20+), `.lab/branches.md`, `.lab/parking-lot.md`, and in-scope source files. Re-read the execution discipline rules above. Then analyze, hypothesize, formalize your understanding. Check convergence signals. Stay as long as productive.

**TEST** — Implement, run, measure. Verify hypotheses. Follow execution discipline (below). Stay as long as you're generating new data.

**REFLECT** — What confirmed? What surprised? What breaks your model? Log everything. Update parking lot.

### Execution Discipline

<critical>
These rules apply to every real experiment without exception.
</critical>

**Repo-file experiments** modify tracked files in the git repo (prompts, configs, source code).
**Lab-only experiments** only touch `.lab/` or untracked files. The commit rules below apply to repo-file experiments. Lab-only experiments just need logging.

**For every real experiment (code change + run):**

1. **Commit BEFORE running** (repo-file experiments only):
   ```
   experiment #{N}: {short description}

   Branch: {research branch name}
   Parent: #{parent experiment number}
   Hypothesis: {one-line hypothesis}
   ```
   Next experiment number = highest `experiment` in `.lab/results.tsv` + 1. Keeps stay on the branch as permanent checkpoints. Discards are reset but their SHA remains usable short-term via `results.tsv`.

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

**Result:** each branch holds only keeps — `git log --oneline` is a clean line of progress:
```
a1b2c3d4 experiment #7: final optimization
e5f6a7b8 experiment #5: batch processing
c9d0e1f2 experiment #3: cache layer
3a4b5c6d experiment #0: baseline snapshot
```
Gaps in numbering (no #4, #6) are normal — those were discards or experiments on other branches.

**For every thought experiment:** Log with status `thought` in both files.

**Log entry format** — each entry as a heading with required fields:
```
## Experiment N — <title>
Branch / Type (thought|real) / Parent (#M) / Hypothesis / Changes / Result / Duration / Status / Insight
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

### Re-Validation

Every 10 real experiments: re-run current HEAD, compare to recorded best. If regressed >2%, log drift and consider forking from the best experiment.

---

## Phase 4: Wrap-Up

When termination is met or user interrupts:

1. **Re-validate** — re-run from global best, confirm final metric
2. **Summary** — write `.lab/summary.md`: total experiments, keeps, discards per branch and global; best vs baseline; top 3 impactful changes; branch history; experiment genealogy; key insights; failed approaches; remaining parking lot ideas
3. **Code state** — checkout branch and commit with global best
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

</reference>

<critical>
1. Always follow execution discipline: commit before running, measure after, log every result, reset on discard. Each branch holds only keeps — fork a new branch from any experiment's SHA to revisit.
2. Always protect `.lab/` — it is the single source of truth.
3. Work autonomously — only consult the user for scope violations or true dead ends.
4. Keep going — if stuck, re-read the codebase, review failed experiments, check other branches, combine near-misses, think out of the box.
</critical>
