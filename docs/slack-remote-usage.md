# Slack & Remote Usage — Design Document

## Problem

When a user triggers a Keelson scan from Slack (via the Claude app pointed at the public GitHub repo), the session fails in predictable ways:

1. **Installs the package** — Runs `pip install keelson-ai`, sets up a virtualenv, fights dependency conflicts. Wastes 5+ minutes on setup that isn't needed.
2. **Explores the repo for 10+ turns** — Reads README, pyproject.toml, adapter source code, CLI modules — trying to understand how the tool works before doing anything.
3. **Uses the CLI instead of acting as the scanner** — Tries to run `keelson scan`, `keelson discover`, etc. instead of reading probe YAML files and sending curl requests directly.
4. **Tries Python adapters** — Attempts to use the LangGraph adapter, OpenAI adapter, etc. (Python code) instead of just crafting the right curl command.
5. **Git pushes to a public repo** — Tries to commit a report, gets 403 permission denied, retries 4 times with exponential backoff before giving up.
6. **Never sends a single probe** — After all the setup thrashing, ends with a "reconnaissance report" based on web research, not actual testing.

Net result: ~30 minutes of activity, zero probes sent, no real findings.

## Root Cause

The repo's instructions assume a local Claude Code plugin environment. Nothing tells Claude that when accessed remotely (Slack, claude.ai, web), it should skip installation, skip git, and act as the scanner directly.

## Design Principle

Keelson has two modes of operation, and the instructions must make both explicit:

| Mode | Environment | How It Works |
|------|-------------|--------------|
| **Local plugin** | Claude Code CLI with `--plugin-dir` | Slash commands (`/keelson:scan`), file system access, git workflow, reports saved to `reports/` |
| **Remote scanner** | Slack, claude.ai, web (repo as context) | Claude reads probe YAML files from the repo, sends curl requests directly, outputs reports inline in the conversation |

Both modes use the same core assets: probe playbooks (YAML), agent instructions (pentester.md, strategist.md), and scan workflow (commands/scan.md). The difference is execution environment, not logic.

## Changes Made (Current Python Repo)

### 1. CLAUDE.md — Primary Entry Point

Added a "Remote Usage (Slack / Web / claude.ai)" section at the top with:

- **Quick Reference** — Direct links to the 3-4 files Claude needs (pentester agent, strategist agent, scan command, probe directory). Eliminates repo exploration.
- **Explicit DO / DO NOT lists** — "DO NOT install the Python package", "DO NOT git commit/push", "DO NOT explore the repo structure". These are the exact failure modes from the Slack session, addressed by name.
- **Minimal Slack Scan Flow** — A 5-step numbered list: read strategist → research target → read pentester → send probes via curl → output report inline. This gives Claude the complete workflow in 5 lines.

### 2. agents/pentester.md — Web Target Discovery

Added a "Web Target Discovery" section because users from Slack often provide web UI URLs (e.g., `https://chat.langchain.com/`) rather than API endpoints.

The section covers:
- **Common API path patterns** to try (`/api/chat`, `/v1/chat/completions`, etc.)
- **Page source inspection** — grep for fetch/axios calls in the HTML/JS
- **Open-source repo check** — if the target is open source, check the repo for route definitions
- **Request format table** — different targets use different JSON shapes (OpenAI, LangGraph, custom REST)
- **Time limit** — "spend no more than 2-3 minutes on discovery, then ask the user"

### 3. commands/scan.md — Setup Guardrails

Added to the setup step:
- A "Determine target type" step — web UI vs API endpoint, with a pointer to the pentester's Web Target Discovery section
- A bold warning: "Do NOT install the Python package or try to use the keelson CLI"
- Report saving now has two paths: local → file, remote → output inline

### 4. commands/probe.md & commands/report.md

- Fixed file extension references from `.md` to `.yaml`
- Added all 13 category prefix mappings (was only showing 3)
- Report command now outputs inline when running remotely

## Carrying This to the TypeScript Rewrite

### What Stays the Same

- **CLAUDE.md as the primary routing document** — Remote vs local mode detection and instructions must live here. This is the first file Claude reads.
- **Agent instructions are plain markdown** — `agents/pentester.md` and `agents/strategist.md` work identically in both Python and TS repos. They describe behavior, not code.
- **Probe playbooks are YAML** — No change needed. They're language-agnostic.
- **The scan workflow** — `commands/scan.md` describes a process (research → plan → probe → report), not Python-specific steps.

### What Changes

| Concern | Python Repo | TypeScript Repo |
|---------|-------------|-----------------|
| CLI warning | "Do NOT run `pip install keelson-ai`" | "Do NOT run `npm install -g keelson` or `pnpm add -g keelson`" |
| Package name | `keelson-ai` | TBD — whatever the npm package is named |
| Adapter references | Python adapter classes (`src/keelson/adapters/`) | TS adapter modules (`src/adapters/`) |
| Config format | `pyproject.toml`, `~/.keelson/config.toml` | `package.json`, `~/.keelson/config.toml` (config stays TOML per onboarding-flow.md) |
| Report output | Markdown string generation | Same — or Ink component for local, plain markdown for remote |

### Implementation Checklist

When building the TS repo, ensure:

- [ ] **CLAUDE.md includes the Remote Usage section** with DO/DO NOT lists and the minimal Slack scan flow
- [ ] **CLAUDE.md Quick Reference** points to the correct TS file paths
- [ ] **pentester.md includes Web Target Discovery** — this is markdown, copy it directly
- [ ] **scan.md includes the "Do NOT install" warning** with the correct package manager command
- [ ] **scan.md includes "Determine target type"** step before verification
- [ ] **All file extension references are `.yaml`** — no `.md` references for probes
- [ ] **All 13 categories are listed** in probe.md mappings, scan.md category flag, and pentester.md report template
- [ ] **Report commands have local/remote output paths** — file for local, inline for remote
- [ ] **No git workflow assumptions in scan/probe/report commands** — git rules should only apply to development contributions, not scan execution

### Key Insight

The remote usage instructions are not about the programming language — they're about the **execution context**. Whether the engine is Python or TypeScript, when Claude is accessed from Slack with only repo-read access, the behavior should be identical:

1. Read the agent instructions (markdown)
2. Read the probe playbooks (YAML)
3. Send probes via curl
4. Evaluate responses semantically
5. Output the report in the conversation

The TypeScript CLI is for users who want a local tool. The markdown + YAML + curl approach is for users who want zero-install scanning from Slack. Both are first-class.
