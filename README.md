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

**Three jobs, one core, in layers.** **[Yggdrasil](https://github.com/krzysztofdudek/Yggdrasil)** is the law: the architecture graph and the rails that hold every change to it. **[Grain](https://github.com/krzysztofdudek/Grain)** surveys the terrain: it mines that graph from a repository's own code and history, so there is a rule-backed map before anyone writes a rule by hand. **[Horde](https://github.com/krzysztofdudek/Horde)** is the software house that builds on the law: zero standing roles, a worker per ticket and a one-shot architect who rules the whole plan once, each ticket refined onto the graph and given a tick. Adoption runs Grain first — install it day zero for a soft, draft-only law that never blocks — then Yggdrasil as the core you keep long term, hard law with proof and CI. Work too big for one agent has two doors. Horde is the door for a mission held to Yggdrasil's law, and the add-on **[Jarl](https://github.com/krzysztofdudek/JarlSkill)** is the lighter door beside it: an issue loop, a worker per issue and evidence before each merge, with no law and no landing gate. From 6.0.0 the core ships as one version; the add-ons keep their own. In the family, law is raised by whichever agent does the work in its own territory, and only the client — the one person the whole system answers to — lowers or vetoes it. The three core repositories' shared machine contracts are registered on [one page](https://krzysztofdudek.github.io/Yggdrasil/family-contracts).

| Core | What it holds |
|---|---|
| **[Yggdrasil](https://github.com/krzysztofdudek/Yggdrasil)** | The law. The architecture graph and the rails that hold every change to it, checked before the agent moves on, re-proved in CI without a key. |
| **[Grain](https://github.com/krzysztofdudek/Grain)** | The terrain survey. Mines a repository's own code and history into a first graph — components, dependencies, and the rules the code already keeps, each with the count of places that break it today; Yggdrasil accepts it with one command. |
| **[Horde](https://github.com/krzysztofdudek/Horde)** | The software house on the law. Zero standing roles: a worker per ticket in its own worktree, refined onto the graph and given a tick by a nine-item merge checklist; a one-shot architect rules the whole plan once; the client orders the mission and is the only one who can lower or veto a rule. |

Four add-ons attach to the agent rather than to the graph, and each works alone. Horde doesn't assume any of them is installed — it carries its own minimum discipline in each role's law — but uses Ratatoskr, Urd and Researcher when they are, one sentence per row below.

| Add-on | Stage | What it makes the agent prove | In Horde's loop |
|---|---|---|---|
| **[Ratatoskr](https://github.com/krzysztofdudek/RatatoskrSkill)** | request → intent | Keeps the agent talking to you in plain words, not code, so you can follow what it's doing. | Keeps the client's plain-language registry open at both ends of a mission. |
| **[Urd](https://github.com/krzysztofdudek/UrdSkill)** | intent → code | When the spec is ambiguous, it consults the source of truth and asks, it doesn't guess. | The stop a worker hits before it guesses. |
| **Researcher** (this one) | code → measured result | Point it at a metric and it runs experiments, hypotheses kept and discarded. | Runs the retrospective's measurement. |
| **[Jarl](https://github.com/krzysztofdudek/JarlSkill)** | issues → merged branch | The agent directs a crew: it files what it sees as issues, gives each issue a worker in its own worktree, and merges a worker's branch only after a fresh reviewer approves it. | None. Jarl runs beside Horde, for work that needs more hands than one agent and no architecture graph. |

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
