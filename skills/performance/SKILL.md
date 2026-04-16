---
name: performance
description: Use this skill when the user says /performance, /digest, asks for a weekly Claude performance report, or wants to review how their Claude Code sessions have been going. Reads JSONL session files from ~/.claude/projects/, computes six effectiveness metrics (activity distribution, one-shot edit rate, subagent turn count, model mix, project allocation, hookify firings), fires diagnostic rules when thresholds breach, and writes behavioral rules into ~/.claude/CLAUDE.md so future sessions adapt. Closes the measurement loop for self-improving AI workflows.
version: 1.0.0
---

# Performance Digest

Measurement layer for Claude Code. Turns session telemetry into prescriptive behavioral rules that Claude reads on the next session start.

A static rule is a wish. A measured rule is a system.

---

## What it does

Every seven days, reads all Claude Code session files from the last week and computes:

- **Activity distribution**: Coding, Exploration, Debugging, Delegation, Planning, Conversation
- **One-shot edit rate**: percentage of file edits that landed without a retry cycle
- **Agent spawn analysis**: how many subagents fired and how many turns each took
- **Model mix**: Opus vs. Sonnet vs. Haiku across all turns
- **Project allocation**: which codebases consumed the most attention
- **Hookify firings**: which behavioral guardrails actually triggered

Then runs six diagnostic rules. When a rule fires, one of two things happens:

1. **Behavioral prescriptions** (verbose agents, model routing, low one-shot rate, exploration overhead) are written directly to `~/.claude/CLAUDE.md` as permanent rules Claude reads on future session starts.
2. **Investigation prescriptions** (recurring errors, hookify repeats) are appended to a Claude To-dos list for the user to review.

Next week the digest re-measures. If the number moved, the rule worked. If it did not, the rule fires again with updated numbers.

---

## Setup

Expected vault layout (follows `ai-brain-starter` conventions):

```
<vault-root>/
  ⚙️ Meta/
    Performance/            (reports land here)
    Claude To-dos.md        (prescriptions go here)
    scripts/
      claude_performance_digest.py
```

Copy `scripts/claude_performance_digest.py` into your vault at the matching path. The script self-locates via `__file__` and expects to sit at `<vault>/⚙️ Meta/scripts/`.

Optional: edit the `PROJECT_LABELS` dict at the top of the script to map project directory substrings to clean display labels.

Optional: tune the `THRESHOLDS` dict. Defaults are calibrated for daily power users.

---

## Usage

Run manually:

```bash
python3 "<vault>/⚙️ Meta/scripts/claude_performance_digest.py"
```

Or schedule weekly. Examples:

Cron (Monday 1am UTC, adjust offset for your timezone):

```
0 1 * * 1 /usr/bin/python3 "/path/to/vault/⚙️ Meta/scripts/claude_performance_digest.py"
```

Or via the `scheduled-tasks` Claude Code plugin, if installed.

---

## Flags

- `--days N`: lookback window (default 7)
- `--dry-run`: print report to stdout, do not write files
- `--no-report`: skip the markdown report, apply prescriptions only

---

## What the output looks like

A dated markdown file at `⚙️ Meta/Performance/weekly-YYYY-MM-DD.md`:

```
# Performance Digest: 2026-04-09 to 2026-04-16

**767 sessions** across 3 projects. 20,822 assistant turns. 773 files edited.

## Activity Distribution
| Category     | Turns | %  |
| Coding       | 1392  | 7% |
| Exploration  | 3062  | 15%|
| Debugging    | 2719  | 13%|
| Delegation   | 324   | 2% |
| Planning     | 2292  | 11%|
| Conversation | 13033 | 52%|

## One-Shot Edit Rate
**83%** (773 files edited, 129 required retries) - on target

## Agent Spawns
**324 agents spawned**. Average turns per agent: **22.0**

## Diagnostics
- **VERBOSE AGENTS**: Avg agent turns is 22.0 (target: <5). Review agent
  prompts: add file paths, expected output format, and clear scope.

## Trending
*Baseline week. No prior data for comparison.*
```

And a behavioral rule written into `~/.claude/CLAUDE.md`:

```
- [VERBOSE AGENTS fix](performance_verbose_agents.md) | Agent briefings must
  include: specific file paths, expected output format, and scope boundary.
  Target: <8 turns per agent. Current avg: 22.0. (updated 2026-04-16)
```

---

## Why this exists

Self-improvement by memory alone has a failure mode: a rule gets written, Claude reads it at session start, and under the wrong context the behavior recurs anyway. Without measurement, you cannot tell whether a correction actually worked. You can only tell that you wrote the rule.

This skill closes the loop. Rules come with a number attached. Baseline, target, check-in. Next week the measurement tells you whether the rule is alive, needs revising, or can retire.

---

## Integration

If you use the `/weekly` skill from `claude-insights`, the weekly review auto-surfaces the most recent performance digest in its report. The two skills compose: `/weekly` tells you what happened in your life, `/performance` tells you what happened in your AI workflow.
