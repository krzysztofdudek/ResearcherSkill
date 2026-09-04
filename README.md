<img width="1376" height="768" alt="Gemini_Generated_Image_5z6cxx5z6cxx5z6c" src="https://github.com/user-attachments/assets/02c3c4cd-fe2a-4fcd-b414-29bd84f5a741" />

# Researcher Skill

**One file. Your AI coding agent becomes a scientist.**

[![Latest Release](https://img.shields.io/github/v/release/krzysztofdudek/ResearcherSkill)](https://github.com/krzysztofdudek/ResearcherSkill/releases/latest)
[![License](https://img.shields.io/github/license/krzysztofdudek/ResearcherSkill)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/krzysztofdudek/ResearcherSkill)](...)
[![GitHub Discussions](https://img.shields.io/badge/Discussions-Join-181717?logo=github&logoColor=white)](https://github.com/krzysztofdudek/ResearcherSkill/discussions)

Install as a Claude Code plugin, or drop `skills/researcher/SKILL.md` into Codex, Cursor, or any agent that reads markdown skills. The agent designs experiments, tests hypotheses, discards what fails, keeps what works — 30+ experiments overnight while you sleep.

## Install

### Claude Code plugin (recommended)

Two slash commands inside Claude Code — first registers this repo as a marketplace, second installs the plugin from it:

```
/plugin marketplace add krzysztofdudek/ResearcherSkill
/plugin install researcher@researcher-marketplace
```

Run `/reload-plugins` to activate it (or restart Claude Code), then trigger the skill with `/researcher` or by asking the agent to run a research loop on something.

To upgrade later: `/plugin marketplace update researcher-marketplace` then `/plugin install researcher@researcher-marketplace` again.

### GitHub Copilot CLI plugin

The same repo is also a [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli) marketplace. Register it, then install the plugin:

```
copilot plugin marketplace add krzysztofdudek/ResearcherSkill
copilot plugin install researcher@researcher-marketplace
```

To upgrade later: `copilot plugin update researcher`. The same skill body powers both Claude Code and Copilot — trigger it the same way.

### Codex CLI plugin

Codex reads the same skill. Register this repo as a marketplace, then install:

```
codex plugin marketplace add krzysztofdudek/ResearcherSkill
codex plugin install researcher@researcher-marketplace
```

To upgrade later: `codex plugin marketplace upgrade researcher-marketplace`. Or drop the single file into `~/.agents/skills/researcher/SKILL.md` (user-level) or `.agents/skills/researcher/SKILL.md` (project-level).

### Cursor plugin

Cursor auto-discovers the skill from the plugin manifest at the repo root. Install it locally:

```
git clone https://github.com/krzysztofdudek/ResearcherSkill.git
ln -s "$(pwd)/ResearcherSkill" ~/.cursor/plugins/local/researcher
```

Then reload Cursor (**Developer: Reload Window**). Or drop the single file into `~/.cursor/skills/researcher/SKILL.md` (user-level) or `.cursor/skills/researcher/SKILL.md` (project-level).

### Single-file drop-in (any agent)

The canonical skill body is `skills/researcher/SKILL.md` in this repo (one file, ~300 lines, frontmatter-tagged). Copy it into your agent's skill directory:

- **Claude Code (user-level):** `~/.claude/skills/researcher/SKILL.md`
- **Claude Code (project-level):** `.claude/skills/researcher/SKILL.md` in your repo
- **Codex / other agents:** wherever your tool reads skills or instructions from (consult its docs)

Trigger with `/researcher` (Claude Code) or by asking the agent to enter "researcher mode".

## What it looks like running

> ### Experiment b4 — READ/WRITE phase separation
> **Branch:** research/graph-protocol-optimization · **Parent:** #b1 · **Type:** real
>
> **Hypothesis:** Agents read architectural rules but treat them as optional. Separating the instruction into a READ phase ("load constraints first") and a WRITE phase ("now implement") with a guard ("if you haven't done READ, stop") should improve compliance.
> **Changes:** restructured agent rules into explicit READ/WRITE phases, added structural guard
> **Result:** 7.04/10 (was 1.82 baseline, 5.91 best) — **new best**
> **Status:** keep
>
> **Insight:** Every attempt to add verification checklists regressed. What worked was changing the structure, not adding steps. Agents respond to framing, not policing.

- b0: baseline (no special instructions): 1.82/10. keep.
- b1: reframe rules as "constraints, not suggestions": 5.91. keep.
- b2: exhaustive checklist: regression. discard.
- b3: lightweight checkpoint: regression. discard.
- b4: READ/WRITE separation + structural guard: **7.04**. **keep.**
- b5: contractual "implement or document exception": regression. discard.
- b6: JIT re-reading: 5.23, evaluator disagreement. interesting.
- b7: mandatory pattern-triggered re-reading: 1.4. **regression below baseline.** discard.

*Real experiment from optimizing [Yggdrasil](https://github.com/krzysztofdudek/Yggdrasil) agent rules. The skill works on any codebase.*

**Same loop, different problems:**
- `npm run build` takes 40s → agent gets it to 18s
- prompt returns wrong format 30% of the time → agent gets it to 3%
- API p99 is 200ms → agent finds the bottleneck and cuts it to 80ms
- document parser misses edge cases → agent improves match rate from 74% to 91%

## How it works

The agent interviews you about what to optimize, sets up a lab on a git branch, and works autonomously. Thinks, tests, reflects. Commits before every experiment, reverts on failure, logs everything.

It detects when it's stuck and changes strategy. Forks branches to explore different approaches. Keeps going until you stop it or it hits a target. Resume where you left off across sessions.

Generalizes [autoresearch](https://github.com/karpathy/autoresearch) beyond ML. Works on any problem where you can measure a result — code, configs, prompts, documents.

All experiment history lives in an untracked `.lab/` directory. Git manages code. `.lab/` manages knowledge.

**Want the full walkthrough?** Read the [guide](GUIDE.md). It walks through a complete example from start to finish.

## FAQ

**How is this different from autoresearch?**
Autoresearch's core loop is universal, but the repo is wired to `train.py`, `val_bpb`, and GPU training. To use it on something else you'd rewrite the setup. This gives you that loop ready to go for any codebase.

**When would I use this instead of ML?**
It's not instead of ML. ML is one possible domain. This works on anything where the agent can try things, measure, and iterate. Code, scripts, documents, configs. Slow builds, flaky tests, API latency, prompt accuracy.

**How does it measure success for non-ML code?**
Whatever you can measure. Test pass rate, benchmark output, type check errors, build time. You set it up in the discovery phase. The agent asks what to measure and how. If you can run a command and get a number, that's your metric. For cases where there's no command to run, the agent scores against a qualitative rubric you define together.

**How does convergence detection work?**
The agent checks a table of signals after every experiment. If it sees 5+ failures in a row, a metric plateau, or the same area modified too many times, it knows to change approach. Some signals are advisory (consider pivoting), others are hard guardrails (you must pivot). Details in the [guide](GUIDE.md).

**Can it improve itself?**
Sort of. The skill was optimized using the skill itself. A research document about how LLMs process instructions (attention decay, primacy/recency, instruction budgets) was used as criteria, and the agent ran the loop against its own prompt. Not fully recursive, but the loop was: research → skill → use skill to improve skill.

**Can't I just ask Claude to build this from the autoresearch repo?**
You can try. This saves you the work and includes things autoresearch doesn't have: thought experiments, non-linear branching, convergence detection, qualitative metrics, and session resume.

## License

MIT

## The Yggdrasil family

Four tools, one thesis: **make an AI coding agent prove correctness, stage by stage** — because "done" isn't done. Each is a checkpoint at a different point in the pipeline, where the agent has to show its work before it continues.

| Tool | Stage | What it makes the agent prove |
|---|---|---|
| **[Ratatoskr](https://github.com/krzysztofdudek/RatatoskrSkill)** | request → intent | Keeps the agent talking to you in plain words, not code, so you can follow what it's doing. |
| **[Urd](https://github.com/krzysztofdudek/UrdSkill)** | intent → code | When the spec is ambiguous, it consults the source of truth and asks instead of guessing. |
| **[Yggdrasil](https://github.com/krzysztofdudek/Yggdrasil)** | code → architecture | Every change satisfies the rules that govern it, checked before the agent moves on. |
| **Researcher** (this one) | code → measured result | Point it at a metric and it runs experiments, hypotheses kept and discarded. |

Two more sit alongside the chain rather than inside it. **[Horde](https://github.com/krzysztofdudek/Horde)** doesn't own a stage — it's what you add when a mission needs more than one agent to move through all four at once, holding every agent it raises to the same standards. **[Grain](https://github.com/krzysztofdudek/Grain)** reads the conventions a codebase actually practices instead of the ones someone declared — the same seam as Yggdrasil from the other side. Paused since 2026-09-02: the engine works, but the question that decides whether it's worth using — does it change what an agent ships — came back not demonstrated.

---

<div align="center">
  <img src="yggdrasil.svg" alt="Yggdrasil" width="150" />
  <br/><br/>
  <a href="https://github.com/krzysztofdudek/ResearcherSkill/discussions">
    <img src="https://img.shields.io/badge/Discussions-Join-181717?logo=github&logoColor=white" alt="GitHub Discussions" />
  </a>
  <br/>
  <sub>Questions? Open a discussion on GitHub.</sub>
</div>
