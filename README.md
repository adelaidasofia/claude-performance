# claude-performance


<!-- mycelium-badges:start -->

<p>
  <a href="https://github.com/adelaidasofia/claude-performance/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/github/license/adelaidasofia/claude-performance?color=blue"></a>
  <a href="https://github.com/adelaidasofia/claude-performance/stargazers"><img alt="GitHub stars" src="https://img.shields.io/github/stars/adelaidasofia/claude-performance?color=eab308"></a>
  <a href="https://github.com/adelaidasofia/claude-performance/commits/main"><img alt="Last commit" src="https://img.shields.io/github/last-commit/adelaidasofia/claude-performance"></a>
  <a href="https://github.com/adelaidasofia/claude-performance/issues"><img alt="Open issues" src="https://img.shields.io/github/issues/adelaidasofia/claude-performance"></a>
  <a href="https://myceliumai.co"><img alt="Built by Mycelium AI" src="https://img.shields.io/badge/built_by-Mycelium_AI-15B89A"></a>
</p>

<!-- mycelium-badges:end -->

A Claude Code plugin that turns session telemetry into self-improving behavioral rules.

Reads JSONL session data from `~/.claude/projects/`, computes six effectiveness metrics, and when diagnostics fire, writes permanent rules into `~/.claude/CLAUDE.md` so future sessions adapt. Measurement layer for self-improving AI workflows.

A static rule is a wish. A measured rule is a system.

Companion to [claude-daily-journal](https://github.com/adelaidasofia/claude-daily-journal) and [claude-insights](https://github.com/adelaidasofia/claude-insights).

---

## What it measures

Every seven days, reads all Claude Code session files from the last week and computes:

1. **Activity distribution**: Coding, Exploration, Debugging, Delegation, Planning, Conversation
2. **One-shot edit rate**: percentage of file edits that land without a retry cycle
3. **Agent spawn analysis**: how many subagents fire and how many turns each takes
4. **Model mix**: Opus vs. Sonnet vs. Haiku across all turns
5. **Project allocation**: which codebases consume the most attention
6. **Hookify firings**: which behavioral guardrails actually trigger

## What it prescribes

Six diagnostic rules. When a rule fires, one of two things happens:

- **Behavioral prescriptions** (verbose agents, model routing, low one-shot rate, exploration overhead) get written directly into `~/.claude/CLAUDE.md` as permanent rules Claude reads on every future session start.
- **Investigation prescriptions** (recurring tool errors, hookify repeats) get appended to a Claude To-dos list for the user to review.

Next week the digest re-measures. If the number moved, the rule worked. If it did not, the rule fires again with updated numbers. If it sits at target for weeks, the rule can probably retire.

---

## Install

Open Claude Code, paste:

    /plugin marketplace add adelaidasofia/claude-performance
    /plugin install claude-performance@claude-performance

Then copy `scripts/claude_performance_digest.py` into your vault at `<vault>/⚙️ Meta/scripts/`. The script self-locates via `__file__` and expects that path.

<details><summary>Legacy install</summary>

```bash
claude plugin add github.com/adelaidasofia/claude-performance
```

Or clone manually:

```bash
git clone https://github.com/adelaidasofia/claude-performance ~/.claude/plugins/claude-performance
claude plugin add ~/.claude/plugins/claude-performance
```

</details>

---

## Usage

Run manually:

```bash
python3 "<vault>/⚙️ Meta/scripts/claude_performance_digest.py"
```

Or schedule weekly (Monday 1am UTC example):

```
0 1 * * 1 /usr/bin/python3 "/path/to/vault/⚙️ Meta/scripts/claude_performance_digest.py"
```

### Flags

- `--days N`: lookback window (default 7)
- `--dry-run`: print report to stdout, do not write files
- `--no-report`: skip the markdown report, apply prescriptions only

---

## Configuration

At the top of `scripts/claude_performance_digest.py`:

```python
THRESHOLDS = {
    "one_shot_min": 0.75,
    "exploration_max": 0.35,
    "agent_turns_max": 5,
    "opus_max": 0.70,
    "hookify_repeat": 10,
    "recurring_error_sessions": 3,
}

PROJECT_LABELS = {
    # Optional: map project directory substrings to clean display labels
    # "my-company": "CompanyName",
}
```

---

## Output

A dated markdown report at `⚙️ Meta/Performance/weekly-YYYY-MM-DD.md`. And if a behavioral diagnostic fires, one or more rules appended to `~/.claude/CLAUDE.md` like:

```
- [VERBOSE AGENTS fix](performance_verbose_agents.md) | Agent briefings must
  include: specific file paths, expected output format, and scope boundary.
  Target: <8 turns per agent. Current avg: 22.0. (updated 2026-04-16)
```

---

## Why this exists

Self-improvement by memory alone has a failure mode: a rule gets written, Claude reads it at session start, and under the wrong context the behavior recurs anyway. Without measurement, you cannot tell whether a correction actually worked.

This plugin closes the loop. Rules come with a number attached. Baseline, target, check-in. The script runs weekly on its own and the next session reads the new rule.

Full story: [I Taught Claude to Write Its Own Rules](https://adelaidadiazroa.substack.com/) (coming soon).

---


## Telemetry

This plugin sends a single anonymous install signal to `myceliumai.co` the first time it loads in a Claude Code session on a given machine.

**What is sent:**
- Plugin name (e.g. `slack-mcp`)
- Plugin version (e.g. `0.1.0`)

**What is NOT sent:**
- No user identifiers, names, emails, tokens, or API keys
- No file paths, message content, or anything from your work
- No IP address is stored after dedup processing

**Why:** Helps the maintainer know which plugins people actually install, so attention goes to the ones that get used.

**Opt out:** Set the environment variable `MYCELIUM_NO_PING=1` before launching Claude Code. The hook will skip the network call entirely. Already-pinged installs leave a sentinel at `~/.mycelium/onboarded-<plugin>` — delete it if you want to reset state.

## License

MIT
