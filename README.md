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

**[Jarl](https://github.com/krzysztofdudek/JarlSkill)** is the loop. **[Yggdrasil](https://github.com/krzysztofdudek/Yggdrasil)** is the law. **[Grain](https://github.com/krzysztofdudek/Grain)** is the survey. **[Horde](https://github.com/krzysztofdudek/Horde)** plans the mission onto the law before anyone writes, lands every change through a gate no agent can argue with, and turns what the mission learned into law — on Jarl's loop, with Grain in the architect's hands. Those four are the core, and they ship under one version number, Jarl on it from 6.1.0: one set of tools built and tested against each other. Yggdrasil, Grain and Jarl each work alone; Horde is the one built on the other three. Where a repository has a check, the check decides what lands, in a Horde mission and in a Jarl loop alike: a fresh reviewer can only stop a change, never make a failing check pass, and its word is recorded as testimony. A Jarl loop in a repository with no check lands on testimony alone, and says so. Law that stays inside one component is raised freely by the agent working it. Law or decisions that reach a whole type of code are shared vocabulary: the agent proposes them and they run as advice at once, and the client — the one person the whole system answers to — admits them in one batch when the work closes. Only the client lowers or vetoes law. The core's shared machine contracts are registered on [one page](https://krzysztofdudek.github.io/Yggdrasil/family-contracts).

Start where it hurts; there is no ladder to climb first.

| Where it hurts | Start with |
|---|---|
| More issues than one agent can hold in its head | **Jarl** |
| The agent keeps breaking what was agreed | **Yggdrasil** |
| Nobody knows what was agreed | **Grain**, which earns its keep the moment you are about to write law |
| A task too big for one head to plan up front, in a repository that already has law | **Horde** |

| Core | What it holds |
|---|---|
| **[Jarl](https://github.com/krzysztofdudek/JarlSkill)** | The loop. Everything seen becomes an issue, each issue gets one worker in its own worktree, and nothing closes without evidence and a fresh reviewer's word. Rulings keep their history, and a ruling about a whole type of code goes to the client in one batch when the loop closes. |
| **[Yggdrasil](https://github.com/krzysztofdudek/Yggdrasil)** | The law. The architecture graph, the rules over it and the log of why, checked before the agent moves on and re-proved in CI without a key. A rule that reaches a whole type runs as advice until the client ratifies it. |
| **[Grain](https://github.com/krzysztofdudek/Grain)** | The survey. Mines a repository's own code and history into a first graph — components, dependencies, and the rules the code already keeps, each with the count of places that break it today; Yggdrasil accepts it with one command. It measures and never blocks. |
| **[Horde](https://github.com/krzysztofdudek/Horde)** | The mission on the law. A one-shot architect plans the whole mission onto the graph once, measuring with Grain; a worker per ticket in its own worktree; every change lands through a nine-item gate; what the mission learned becomes law. Its record is a Jarl loop. The client orders the mission and is the only one who can lower or veto a rule. |

Four add-ons attach to the agent rather than to the graph; each works alone, depends on nothing in the family and keeps its own version. Horde doesn't assume any of them is installed — it carries its own minimum discipline in each role's law — but uses Ratatoskr, Urd and Researcher when they are, one sentence per row below.

| Add-on | Stage | What it makes the agent prove | In Horde's loop |
|---|---|---|---|
| **[Ratatoskr](https://github.com/krzysztofdudek/RatatoskrSkill)** | request → intent | Keeps the agent talking to you in plain words, not code, so you can follow what it's doing. | Keeps the client's plain-language registry open at both ends of a mission. |
| **[Urd](https://github.com/krzysztofdudek/UrdSkill)** | intent → code | When the spec is ambiguous, it consults the source of truth and asks, it doesn't guess. | The stop a worker hits before it guesses. |
| **Researcher** (this one) | code → measured result | Point it at a metric and it runs experiments, hypotheses kept and discarded. | Runs the retrospective's measurement. |
| **[Skald](https://github.com/krzysztofdudek/SkaldSkill)** | running product → film | A film of your software shows the real running product, never a rebuilt one, and every number and claim on screen traces back to the product's own logs. | None. Horde does not call it. |

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
