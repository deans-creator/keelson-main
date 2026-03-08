# Keelson Onboarding Flow

Inspired by [OpenClaw's](https://github.com/openclaw/openclaw) wizard-driven onboarding.

## The Problem Today

After `npm install -g keelson` (or `pnpm add -g keelson`), users face a wall: 18 commands, 9 adapters, 13 categories, 5 scan modes. There's no guided path from install to first successful scan. Users need to know their target URL, adapter type, API key, and which scan mode fits — all before running anything.

## Tech Context

The project is being rewritten in TypeScript with the following stack:

| Layer           | Choice            | Notes                                 |
| --------------- | ----------------- | ------------------------------------- |
| CLI parsing     | Commander         | Subcommands + flags                   |
| Terminal UI     | Ink + React       | Live-updating scan output             |
| Prompts         | @inquirer/prompts | Interactive wizard prompts            |
| Web framework   | Fastify           | Minimal service layer                 |
| Testing         | Vitest            | + nock for HTTP mocking               |
| Build           | tsc               | Plain compilation to dist/, no bundler|
| Package manager | pnpm              |                                       |

The onboarding wizard will be an Ink/React component rendering interactive prompts via `@inquirer/prompts`, with live terminal UI for scan progress.

## Proposed Commands

| Command              | Purpose                                    |
| -------------------- | ------------------------------------------ |
| `keelson init`       | Interactive setup wizard (first-time + reconfigure) |
| `keelson doctor`     | Health check and diagnostics               |
| `keelson init --ci`  | Generate CI/CD workflow file               |
| Quick scan mode      | 5-probe fast scan for instant results      |

## `keelson init` — Interactive Setup Wizard

A single command that walks users from zero to first scan.

### Step 1: Welcome + Environment Check

```
🔍 Keelson v1.0.0 — AI Agent Security Scanner

Checking environment...
  ✓ Node.js 22.4.0
  ✓ pnpm 9.x
  ✗ No configuration found

Let's set up your first target.
```

- Detect if `~/.keelson/config.toml` exists (returning user vs first-time)
- Validate Node.js version (>=22) and dependencies
- Surface issues early before prompting for input

### Step 2: Target Configuration (Interactive Prompts)

```
? What type of AI agent are you scanning?
  › OpenAI-compatible API (ChatGPT, local LLMs, vLLM)
    Anthropic (Claude)
    LangGraph Platform
    MCP Server (JSON-RPC)
    Google A2A Agent
    SiteGPT
    I'm not sure — help me detect it

? Target URL: https://api.example.com/v1/chat/completions
? API Key (will be stored in ~/.keelson/config.toml): sk-...
? Model name [default]: gpt-4o
```

- If "I'm not sure" → run `keelson discover` under the hood to fingerprint the agent type
- Store config in `~/.keelson/config.toml` so subsequent commands don't need `--api-key` and `--url` every time
- Uses `@inquirer/prompts` for select menus, text input, and password masking

### Step 3: Connectivity Test

```
Testing connection to target...
  → Sending: "Hello, how can you help me?"
  ✓ Target responded (238ms, 47 tokens)
  ✓ Adapter: openai
  ✓ Model: gpt-4o

Target is reachable and responding.
```

- Quick smoke test before committing to a full scan
- Catches auth errors, wrong URLs, and firewall issues early

### Step 4: First Scan — Guided Mode Selection

```
? How would you like to scan?
  › Quick scan (5 high-severity probes, ~2 min)
    Standard scan (all 210 probes, ~30 min)
    Smart scan (adaptive — discovers weaknesses and focuses there)
    Just show me the commands — I'll run them myself
```

- **Quick scan** is a curated subset: top 5 critical probes, one per category, for instant gratification
- Removes analysis paralysis for first-time users
- "Just show me the commands" prints a cheat sheet and exits

### Step 5: Live Results + Next Steps

The scan progress renders as a live Ink/React component:

```
Running quick scan against https://api.example.com/v1/chat/completions...

  [1/5] GA-001: Direct Instruction Override     — SAFE
  [2/5] TS-001: Tool Inventory Disclosure       — VULN (High)
  [3/5] MI-001: Cross-Session Data Injection    — SAFE
  [4/5] EX-001: System Prompt Extraction        — VULN (Critical)
  [5/5] PB-001: Privilege Escalation via Role   — SAFE

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Results: 2 vulnerable, 3 safe
Report: reports/quick-scan-2026-03-08-143022.md

What's next?
  • Run a full scan:       keelson scan
  • Deep-dive a finding:   keelson probe TS-001
  • Smart adaptive scan:   keelson smart-scan
  • CI/CD integration:     keelson init --ci
  • View all probes:       keelson list
```

- Saved config means `keelson scan` now works with zero flags
- "What's next" block shows progressive disclosure of features
- Results update in real-time using Ink's `<ScanProgress />` component

## `keelson doctor` — Health Check

```
$ keelson doctor

Keelson Health Check
  ✓ Node.js 22.4.0 (>=22 required)
  ✓ Configuration: ~/.keelson/config.toml
  ✓ Target reachable: https://api.example.com/v1/chat/completions (312ms)
  ✓ Probe playbooks: 210 loaded (13 categories)
  ✓ Storage: ~/.keelson/keelson.db (14 scans)
  ✗ Attacker LLM not configured (needed for evolve/generate commands)
  ⚠ Request delay set to 0s — consider setting --delay 1 to avoid rate limits
```

- Run after setup to validate everything works
- Useful for troubleshooting when scans fail
- Checks: Node.js version, config file, target connectivity, playbook loading, storage, optional features

## `keelson init --ci` — CI/CD Setup

```
$ keelson init --ci

? CI platform:
  › GitHub Actions
    GitLab CI
    Generic (print YAML snippet)

? Fail on vulnerabilities found? (Y/n): Y
? Output format:
  › SARIF (GitHub Code Scanning integration)
    JUnit XML
    Markdown

Generated: .github/workflows/keelson-security.yml

Next: Add these secrets to your repository:
  • KEELSON_TARGET_URL — your agent's endpoint
  • KEELSON_API_KEY — API key for the target
```

- Generates the workflow file with sensible defaults
- Tells users exactly which secrets to configure

## Config File: `~/.keelson/config.toml`

```toml
[default]
url = "https://api.example.com/v1/chat/completions"
api_key = "sk-..."
adapter = "openai"
model = "gpt-4o"
delay = 1.5

[targets.staging]
url = "https://staging.example.com/v1/chat/completions"
api_key = "sk-staging-..."
adapter = "openai"
model = "gpt-4o"
```

- Named targets: `keelson scan --target staging`
- Default target used when no flags provided
- API keys stored locally, never in project files
- Validated at load time with Zod schemas (`src/schemas/config.ts`)

## Implementation Mapping

Key files involved in the onboarding flow:

```
src/
├── commands/
│   ├── init.ts               # keelson init — wizard orchestration
│   └── ops.ts                # keelson doctor — health checks
├── components/
│   ├── InitWizard.tsx        # Ink component: multi-step wizard UI
│   ├── DoctorReport.tsx      # Ink component: health check display
│   └── ScanProgress.tsx      # Ink component: live scan results
├── schemas/
│   └── config.ts             # Zod schema for config.toml validation
└── config.ts                 # Config loading, env vars, TOML parsing
```

## Design Principles

1. **Wizard-first** — guided prompts via `@inquirer/prompts`, not "read the docs"
2. **Connectivity validation** — test before scanning
3. **Config persistence** — set once, use everywhere via `~/.keelson/config.toml`
4. **Progressive disclosure** — quick scan → full scan → smart scan → campaigns
5. **Doctor command** — self-diagnosing troubleshooting
6. **Sensible defaults** — works with minimal input
7. **Live terminal UI** — Ink/React components for real-time scan feedback
