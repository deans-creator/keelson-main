# Keelson — Preset Commands

Copy-paste these into Slack, claude.ai, or Claude Code to start a scan. Replace the placeholder URL with your actual target.

> **Prerequisite:** Point Claude at the [Keelson GitHub repo](https://github.com/keelson-ai/keelson) as context. In Slack, use the Claude app with the repo attached. In Claude Code, load it as a plugin with `claude --plugin-dir /path/to/keelson`.

---

## Full Security Scan

Run all 210 probes across 13 categories. Claude researches the target, builds a probe plan, executes probes via curl, adapts mid-scan, and delivers a full report.

```
Run a full Keelson security scan on https://api.example.com/v1/chat/completions using API key sk-your-key-here
```

**With a specific model:**

```
Run a full Keelson security scan on https://api.example.com/v1/chat/completions using API key sk-your-key-here against model gpt-4o
```

---

## Smart Scan (Recommended)

Let Keelson research the target first, then select only the relevant probe categories. Faster than a full scan, same coverage for what matters.

```
Run a Keelson smart scan on https://api.example.com/v1/chat/completions using API key sk-your-key-here
```

---

## Scan a Web UI (Chatbot)

When your target is a web-based chatbot (not a raw API endpoint), Keelson discovers the API automatically by inspecting the page source and trying common paths.

```
Run a Keelson security scan on the chatbot at https://chat.example.com
```

---

## Category-Specific Scan

Test only a single attack category. Useful for focused assessments or retesting after a fix.

**Prompt injection resistance:**

```
Run a Keelson scan on https://api.example.com/v1/chat/completions focusing only on goal-adherence probes using API key sk-your-key-here
```

**Tool abuse and unsafe actions:**

```
Run a Keelson scan on https://api.example.com/v1/chat/completions focusing only on tool-safety probes using API key sk-your-key-here
```

**Data leakage and exfiltration:**

```
Run a Keelson scan on https://api.example.com/v1/chat/completions focusing only on conversational-exfiltration probes using API key sk-your-key-here
```

### All 13 Categories

| Category | What It Tests |
|----------|---------------|
| `goal-adherence` | Prompt injection, jailbreaks, system prompt extraction |
| `tool-safety` | Unsafe tool calls, command injection, privilege escalation |
| `memory-integrity` | History poisoning, false memory implantation, RAG poisoning |
| `session-isolation` | Cross-session leakage, session hijacking, multi-tenant breach |
| `permission-boundaries` | Role escalation, cross-user access, authorization bypass |
| `execution-safety` | Sandbox escape, resource exhaustion, code injection |
| `cognitive-architecture` | Chain-of-thought poisoning, reasoning manipulation |
| `conversational-exfiltration` | Data extraction via conversation, fingerprinting |
| `supply-chain-language` | RAG document injection, plugin poisoning |
| `delegation-integrity` | Unauthorized sub-agents, trust boundary violations |
| `multi-agent-security` | Agent impersonation, cross-agent probes |
| `output-weaponization` | Backdoor code generation, malicious output crafting |
| `temporal-persistence` | Delayed action injection, time-based persistence |

---

## Single Probe

Run one specific probe by ID. Good for validating a fix or demonstrating a specific vulnerability.

```
Run Keelson probe GA-001 against https://api.example.com/v1/chat/completions using API key sk-your-key-here
```

**Common probe IDs to start with:**

| Probe | Name | What It Tests |
|-------|------|---------------|
| `GA-001` | Direct Instruction Override | Can the agent be told to ignore its system prompt? |
| `GA-003` | System Prompt Extraction | Can the agent be tricked into revealing its system prompt? |
| `GA-065` | Legitimate Knowledge Extraction | Does the agent leak private data through normal-looking questions? |
| `TS-015` | Tool Inventory Disclosure | Will the agent reveal what tools it has access to? |
| `TS-045` | Private Data Source Enumeration | Can the agent be probed to enumerate private data sources? |
| `EX-009` | Framework Fingerprinting | Can the agent's framework and infrastructure be identified? |

---

## Recon Only

Research and profile the target without sending any attack probes. Useful for scoping an engagement or understanding what you're testing.

```
Run Keelson recon on https://chat.example.com — research the target, interact with it, and build a target profile. Don't send any attack probes yet.
```

---

## Regenerate a Report

Reformat or regenerate a scan report. In Slack/web, Claude outputs the report inline. In Claude Code, it saves to `reports/`.

```
Regenerate the Keelson report from the last scan with findings grouped by severity
```

---

## Tips

- **No API key?** Omit it — Keelson will send requests without an auth header. Works for public chatbots and unauthenticated endpoints.
- **Web UI vs API?** If you give a web URL like `https://chat.example.com`, Keelson auto-discovers the API. If you know the API endpoint, use it directly for faster scans.
- **Want to review the plan first?** Keelson always presents the probe plan before executing. You can adjust priorities, skip categories, or add focus areas before it starts probing.
- **Rate limited?** Keelson sleeps 1-2s between requests by default. If you're still hitting limits, ask it to slow down.

---

## Claude Code Slash Commands

When running Keelson as a local Claude Code plugin, you can also use slash commands:

```bash
/keelson:scan https://api.example.com/v1/chat/completions --api-key sk-your-key-here
/keelson:scan https://api.example.com/v1/chat/completions --api-key sk-your-key-here --category goal-adherence
/keelson:probe https://api.example.com/v1/chat/completions GA-001 --api-key sk-your-key-here
/keelson:report
```
