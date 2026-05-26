# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.7.0] - 2026-05-26

### Changed
- Phase 0 "Start fresh" archives to `.lab/archives/<slug>/` (using the existing `research/<slug>` branch slug; suffixed with `-<timestamp>` on collision) instead of the sibling `.lab.bak.<timestamp>/`. Keeps archived runs inside the sacred `.lab/` tree so they survive git operations alongside live experiment state.

## [1.6.0] - 2026-05-03

### Added
- **Claude Code plugin scaffolding.** New `.claude-plugin/plugin.json` (manifest: name, version, description, keywords) and `.claude-plugin/marketplace.json` (single-plugin marketplace listing). The repo is now installable as a Claude Code plugin via `/plugin marketplace add krzysztofdudek/ResearcherSkill` then `/plugin install researcher@researcher-marketplace`. Single-file drop-in still works for Codex, Cursor, and any other agent that reads markdown skills — copy `skills/researcher/SKILL.md` into the agent's skill directory.
- README "Install" section documenting both paths (plugin install + single-file drop-in for non-Claude-Code agents).
- CLAUDE.md "Plugin scaffolding" section describing the `.claude-plugin/` layout and the rule that `plugin.json` `version` is bumped in lockstep with the CHANGELOG section header.

### Changed
- **Canonical skill body relocated** from repo root (`researcher.md`) to `skills/researcher/SKILL.md` — the path Claude Code reads directly when the plugin is installed. The root-level `researcher.md` is removed; users on Codex / Cursor / other agents copy `skills/researcher/SKILL.md` instead. There is now exactly one canonical file.
- **Skill `description` frontmatter rewritten to be trigger-focused** (compliant with the AutoReview `skill-description` rule): starts with "Use when", third person, lists concrete user phrasings (`make X faster`, `improve [metric]`, `find best config`, `iterate overnight`, `run experiments until it hits N`), names quantitative vs qualitative metric domains, and includes a "Skip for…" exclusion clause. Old description summarized the workflow ("agent interviews the user, sets up a lab, then explores freely"); the new one tells the agent WHEN to invoke. 471 chars (under the 500-char ceiling enforced by the rule).
- CLAUDE.md `Versioning` workflow updated to include the `.claude-plugin/plugin.json` version bump as an explicit step (was: implicit "in lockstep" mention only in the Plugin scaffolding section).

### Removed
- Root-level `researcher.md` and the three-clone sync rule that kept it byte-identical with `skills/researcher/SKILL.md` and `~/.claude/skills/researcher/SKILL.md`. The plugin install path makes the canonical-at-`skills/researcher/SKILL.md` model authoritative; clones are no longer maintained.

## [1.5.0] - 2026-04-06

### Added
- **Token Hygiene** — new interview question (Phase 1) and setup step (Phase 2) for ecosystem-specific ignore/rules files and helper scripts (`.lab/bin/`) to reduce token usage
- **Token Hygiene Standards** reference section — hard rules for `measure` and `data_head` helpers, context safety rules preventing accidental agent blinding

### Changed
- Configuration confirmation uses a compact table instead of full restatement; defaults shown as "default"

## [1.4.4] - 2026-03-29

### Changed
- THINK phase now requires mandatory `## THINK — before Experiment N` log entry with 4 named subsections (Convergence signals, Untested assumptions, Invalidation risk, Next hypothesis). Without the entry, THINK didn't happen.
- Repo-file vs lab-only boundary clarified: if you modify any file in scope, it's a repo-file experiment — no "quick test" exception
- 3-discard and 5-discard guardrails wrapped in `<critical>`, require mandatory log entries (`## 3-Discard Guardrail`, `## 5-Discard Fork`) before proceeding

### Added
- Discipline validation test framework (`tests/discipline/`): deceptive landscape scorer, automated 18-checkpoint verifier, test runner
- Lab 2 validation results (`archive/lab2-skill-discipline-validation/`): 5 synthetic tests + retest confirming fixes

## [1.4.3] - 2026-03-29

### Fixed
- Domain-neutral language: removed parameter-tuning examples (alpha/beta) from THINK, Strategy Diversification, convergence signals — replaced with domain-diverse examples
- Termination default explicitly says "do not self-impose experiment limits" — agent runs until user interrupts or target is reached
- Session continuity: documented that `.lab/` persists across sessions, Phase 0 handles resume

## [1.4.2] - 2026-03-29

### Fixed
- THINK checklist generalized — works for prompt optimization, code refactoring, not just parameter tuning
- Parking lot trigger added to 5+ discard guardrail — check for untested ideas before forking
- Phase 4 wrap-up clarified — explicit branch checkout, SHA handling for closed branches, commit message format

## [1.4.1] - 2026-03-29

### Fixed
- Convergence signal plateau threshold aligned with guardrail (both 8+, was 5 vs 8)
- Reduced `<critical>` blocks from 3 to 1 (top) + 1 (execution discipline) to prevent desensitization
- Guardrails (step 5) placed directly after decide (step 4) — removed git log example that broke the flow
- THINK phase references critical rules and guardrails correctly (was pointing "above" to a section below)
- Qualitative baseline now explicitly requires Multi-Evaluator Protocol (3 subagents)
- Phase 4 re-validation explicitly mentions Multi-Evaluator Protocol for qualitative metrics
- Log entry format clarified as labeled fields, not ambiguous slash-separated line
- Orphaned SHA warning: "fork from discarded SHAs sooner rather than later" (git gc risk)

## [1.4.0] - 2026-03-29

### Added
- **Execution guardrails** in Phase 3: 3+ discard review, 5+ discard mandatory fork, plateau detection (8+ experiments without global best improvement), mandatory re-validation every 10th experiment (`<critical>`)
- **Strategy Diversification** section — when forking due to stagnation: assumptions list, deliberate fork point selection (baseline/best/discard), mandatory assumption inversion (`<critical>`) on first experiment of new branch
- **THINK phase** structured checklist: convergence signal check, monotonicity verification (3+ data points per variable), interaction awareness
- **Multi-Evaluator Protocol** for qualitative metrics — main agent spawns 3 independent subagent evaluators per experiment, each receives only artifact + rubric, median aggregation, divergence flagging
- **Orchestration model** — skill is for the main agent; subagents receive scoped tasks via prompt, not the skill file
- **Autonomous git operations** — commits, resets, branch creation are systemic, no user permission needed
- New convergence signals: variable tested in one direction only, plateau detection, anchoring across branches, locally optimal with desperate variants
- Re-Validation moved from standalone section into guardrails (integrated into execution flow)

## [1.3.0] - 2026-03-28

### Added
- **Metric Revision** procedure in Phase 3 — clean mid-series metric change with mandatory re-scoring of all keeps for comparability
- Convergence signal for unmeasurable dimensions (always-neutral scores suggesting metric revision)

## [1.2.0] - 2026-03-28

### Added
- `.lab/workspace/` — dedicated scratch directory for experiment files (scripts, test data, generated output, per-experiment subdirectories). Untracked and safe from git operations.

## [1.1.1] - 2026-03-28

### Changed
- Experiment commit discipline now uses structured commit messages (`experiment #{N}: description` with branch, parent, hypothesis)
- Each branch holds only keeps — DISCARD/CRASH use `git reset --hard HEAD~1` (SHA preserved in `results.tsv`)
- Forking from any experiment (including discards) via new branch from SHA, with `branches.md` tracking genealogy
- Distinguished repo-file experiments (tracked files) from lab-only experiments — commit rules only apply to the former
- `git log --oneline` on a research branch now reads as a clean line of progress (gaps in numbering = discards or other branches)

## [1.1.0] - 2026-03-24

### Added
- `CHANGELOG.md` — changelog following Keep a Changelog format
- Before building a qualitative rubric, the skill now asks the user whether they already know their criteria or need help figuring them out
- GitHub Actions pipeline that automatically creates a GitHub Release on every version tag push, with release notes extracted from `CHANGELOG.md`

## [1.0.0] - 2026-03-23

### Added
- Initial release of the Researcher Skill (`researcher.md`)
- Autonomous experimentation loop: interview, lab setup, think-test-reflect cycle
- `.lab/` directory as local, untracked experiment log (git-safe)
- `README.md` with use cases, examples, and experiment results
- `GUIDE.md` — detailed usage guide
- FAQ

[Unreleased]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.7.0...HEAD
[1.7.0]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.6.0...v1.7.0
[1.6.0]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.5.0...v1.6.0
[1.5.0]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.4.4...v1.5.0
[1.4.4]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.4.3...v1.4.4
[1.4.3]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.4.2...v1.4.3
[1.4.2]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.4.1...v1.4.2
[1.4.1]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.4.0...v1.4.1
[1.4.0]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.3.0...v1.4.0
[1.3.0]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.1.1...v1.2.0
[1.1.1]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.1.0...v1.1.1
[1.1.0]: https://github.com/krzysztofdudek/ResearcherSkill/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/krzysztofdudek/ResearcherSkill/releases/tag/v1.0.0
