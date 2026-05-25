<img width="1376" height="768" alt="Gemini_Generated_Image_5z6cxx5z6cxx5z6c" src="https://github.com/user-attachments/assets/02c3c4cd-fe2a-4fcd-b414-29bd84f5a741" />

# Researcher Skill

**One file. Your AI coding agent becomes a scientist.**

[![Latest Release](https://img.shields.io/github/v/release/krzysztofdudek/ResearcherSkill)](https://github.com/krzysztofdudek/ResearcherSkill/releases/latest)
[![License](https://img.shields.io/github/license/krzysztofdudek/ResearcherSkill)](LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/krzysztofdudek/ResearcherSkill)](...)
[![Discord](https://img.shields.io/badge/Discord-Join%20Community-5865F2?logo=discord&logoColor=white)](https://discord.gg/SZTbgsH8Wm)

Install as a Claude Code plugin, or drop `skills/researcher/SKILL.md` into Codex, Cursor, or any agent that reads markdown skills. The agent designs experiments, tests hypotheses, discards what fails, keeps what works — 30+ experiments overnight while you sleep.

## Install

### Claude Code plugin (recommended)

Two slash commands inside Claude Code — first registers this repo as a marketplace, second installs the plugin from it:

```
/plugin marketplace add krzysztofdudek/ResearcherSkill
/plugin install researcher@researcher-marketplace
```

Restart Claude Code (or run `/plugin reload`) and trigger the skill with `/researcher` or by asking the agent to run a research loop on something.

To upgrade later: `/plugin marketplace update researcher-marketplace` then `/plugin install researcher@researcher-marketplace` again.

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

## See also

**[Liaison](https://github.com/krzysztofdudek/LiaisonSkill)** — when the optimization target comes from a non-technical stakeholder in business or product terms (not a measurable metric yet). A five-phase dialogue protocol captures intent, holds consent gates, and produces a concrete target you can then point researcher at.

**[Be Precise](https://github.com/krzysztofdudek/BePreciseSkill)** — once you have a plan, this stops the agent from silently filling gaps. Pairs with researcher when an experiment plan needs faithful execution rather than agent-guided improvisation.

**[Yggdrasil](https://github.com/krzysztofdudek/Yggdrasil)** — architecture rules in Markdown your agent can't ignore. A reviewer verifies every change and feeds violations back into the agent's loop before it moves on. Useful when research outcomes need to land in a codebase under enforced rules.

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
