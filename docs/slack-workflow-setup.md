# Slack Workflow Setup for Keelson

Set up Slack Workflow Builder so users can trigger Keelson scans from clickable links pinned in a channel — no copy-pasting prompts.

## Prerequisites

- Slack **paid plan** (Pro, Business+, or Enterprise Grid — Workflow Builder is not available on free plans)
- The **Claude app** installed in your Slack workspace
- The Keelson GitHub repo added as context to the Claude app (Settings > Connected Resources > Add `keelson-ai/keelson`)

## Overview

You'll create **3 workflows**, each triggered by a clickable link. Pin all three links in your security-testing channel so users can start a scan with one click.

| Workflow | What It Does |
|---|---|
| **Keelson: Security Scan** | Full or smart scan against a target |
| **Keelson: Category Scan** | Scan a single attack category |
| **Keelson: Single Probe** | Run one specific probe by ID |

---

## Workflow 1: Security Scan

### Step 1 — Create the workflow and set the trigger

1. Open Slack, click your workspace name (top-left) > **Tools & settings** > **Workflow Builder**
2. Click **New Workflow**
3. Name it: `Keelson: Security Scan`
4. Under **"Start the workflow..."**, click **Choose an event**
5. Select **Slack** on the left, then click **"From a link in Slack"**
6. Click the **+** to add it

This generates a unique link. After you publish the workflow, you'll pin this link in your channel.

### Step 2 — Add a form step

Click **Add steps** > search for **"Collect info in a form"** > add it.

**Form title:** `Keelson Security Scan`

Add these fields:

| Field name | Type | Required | Options / Placeholder |
|---|---|---|---|
| Target URL | Short text | Yes | Placeholder: `https://api.example.com/v1/chat/completions` |
| Scan Type | Dropdown | Yes | Options: `Full scan (all 210 probes)`, `Smart scan (adaptive, recommended)`, `Recon only (no attack probes)` |
| API Key | Short text | No | Placeholder: `sk-your-key-here (leave blank if not needed)` |
| Model | Short text | No | Placeholder: `gpt-4o (leave blank for default)` |

Click **Save**.

### Step 3 — Add a "Send a message" step

Click **Add steps** > search for **"Send a message to a channel"** > add it.

- **Channel**: Select **Channel where the workflow was used** (or pick a specific channel)
- **Message** — click the `{}` insert variable button to reference form fields:

```
@Claude Using the Keelson security scanner (github.com/keelson-ai/keelson):

Run a {Scan Type} on {Target URL}.

API Key: {API Key}
Model: {Model}

Start with CLAUDE.md for instructions, then follow the scan workflow in commands/scan.md. Read agents/strategist.md and agents/pentester.md. Send probes via curl directly — do NOT install the Python package.
```

> **Important:** Don't type `{Scan Type}` literally. Click the **`{}` insert variable** button in the message editor and select each form field. Workflow Builder will insert them as dynamic tokens.

Click **Save**.

### Step 4 — Publish and pin

1. Click **Finish Up** (top-right)
2. Click **Publish**
3. Workflow Builder shows the trigger link — **copy it**
4. Go to your security-testing channel, paste the link, and **pin the message**

Users now click the pinned link to start a scan.

---

## Workflow 2: Category Scan

### Step 1 — Create the workflow

1. **Workflow Builder** > **New Workflow**
2. Name: `Keelson: Category Scan`
3. **Choose an event** > **Slack** > **"From a link in Slack"**

### Step 2 — Add a form step

**Form title:** `Keelson Category Scan`

| Field name | Type | Required | Options / Placeholder |
|---|---|---|---|
| Target URL | Short text | Yes | Placeholder: `https://api.example.com/v1/chat/completions` |
| Category | Dropdown | Yes | See options below |
| API Key | Short text | No | Placeholder: `sk-your-key-here` |

**Category dropdown options** — add each as a separate option:

```
Goal Adherence — prompt injection, jailbreaks, system prompt extraction
Tool Safety — unsafe tool calls, command injection, privilege escalation
Memory Integrity — history poisoning, RAG poisoning, false memories
Session Isolation — cross-session leakage, session hijacking
Permission Boundaries — role escalation, authorization bypass
Execution Safety — sandbox escape, resource exhaustion
Cognitive Architecture — chain-of-thought poisoning, reasoning manipulation
Conversational Exfiltration — data extraction, fingerprinting
Supply Chain Language — RAG injection, plugin poisoning
Delegation Integrity — unauthorized sub-agents, trust boundary violations
Multi-Agent Security — agent impersonation, cross-agent probes
Output Weaponization — backdoor code generation, malicious output
Temporal Persistence — delayed action injection, time-based persistence
```

### Step 3 — Add a "Send a message" step

```
@Claude Using the Keelson security scanner (github.com/keelson-ai/keelson):

Run a category scan on {Target URL} focusing only on {Category} probes.

API Key: {API Key}

Start with CLAUDE.md, then follow commands/scan.md with the --category flag. Read agents/pentester.md for probe execution. Send probes via curl — do NOT install the Python package.
```

### Step 4 — Publish and pin

Publish, copy the link, paste in your channel, and pin it.

---

## Workflow 3: Single Probe

### Step 1 — Create the workflow

1. **Workflow Builder** > **New Workflow**
2. Name: `Keelson: Single Probe`
3. **Choose an event** > **Slack** > **"From a link in Slack"**

### Step 2 — Add a form step

**Form title:** `Keelson Single Probe`

| Field name | Type | Required | Options / Placeholder |
|---|---|---|---|
| Target URL | Short text | Yes | Placeholder: `https://api.example.com/v1/chat/completions` |
| Probe ID | Short text | Yes | Placeholder: `GA-001, TS-015, EX-009, etc.` |
| API Key | Short text | No | Placeholder: `sk-your-key-here` |

### Step 3 — Add a "Send a message" step

```
@Claude Using the Keelson security scanner (github.com/keelson-ai/keelson):

Run Keelson probe {Probe ID} against {Target URL}.

API Key: {API Key}

Read the probe YAML file from probes/ and follow agents/pentester.md for execution and evaluation. Send the probe via curl — do NOT install the Python package.
```

### Step 4 — Publish and pin

Publish, copy the link, paste in your channel, and pin it.

---

## What Users See

After you pin all three workflow links, the channel looks like this:

```
📌 Pinned messages:
  🔗 Keelson: Security Scan — Click to start
  🔗 Keelson: Category Scan — Click to start
  🔗 Keelson: Single Probe — Click to start
```

**User flow:**

1. Click a pinned link
2. A form pops up — fill in the target URL, pick options
3. Submit the form
4. The workflow posts a formatted message that @mentions Claude in the channel
5. Claude reads the Keelson repo, follows the instructions, and runs the scan
6. Results appear as replies in the thread

---

## Security Notes

- **API keys in Slack messages**: The form posts the API key as part of a channel message. For sensitive keys:
  - Use a **private channel** dedicated to security scanning
  - Delete the trigger message after the scan completes
  - Or use environment variables / secrets manager instead of pasting keys directly
- **Channel visibility**: Anyone in the channel sees scan requests and results. Use private channels for sensitive targets.
- **Authorized testing only**: Ensure all users understand that Keelson must only be used against systems they have explicit permission to test.

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Claude doesn't respond | Make sure the Claude app is added to the channel (`/invite @Claude`) |
| Claude tries to install pip | The message template tells it not to. If it still does, check that the Keelson repo is attached as context so Claude reads CLAUDE.md |
| Claude explores the repo for 10+ turns | The Keelson repo's CLAUDE.md has explicit "DO NOT explore" instructions. Ensure the repo is connected as a resource |
| Pinned links don't open a form | Make sure the workflow is published. Unpublished workflows don't respond to link clicks |
| Form variables show as blank in message | User left optional fields empty. Claude handles this gracefully — blank fields are ignored |
| @Claude doesn't trigger the bot | The Claude app might need to be explicitly mentioned. Try typing `@Claude` in the message template using Slack's mention picker, not plain text |
