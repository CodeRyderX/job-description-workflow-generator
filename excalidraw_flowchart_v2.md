---
name: excalidraw-automation-workflows
description: Generate Excalidraw automation workflow flowcharts from Upwork job descriptions. Produces a single .excalidraw file containing one or more workflow diagrams side by side, each labeled with its platform name and using platform-specific node names — ready to transfer directly to the target platform.
scripts: []
---

# Excalidraw Automation Workflow Generator

Convert Upwork job descriptions into platform-accurate Excalidraw workflow diagrams. Each diagram maps directly to a real automation platform using that platform's exact node/module names. If the job requires multiple workflows, all are placed **side by side in a single `.excalidraw` file**, each with a clear platform label header.

---

## When to Use

- User pastes an Upwork job description involving automation, integrations, or workflows
- User asks for a workflow diagram, automation map, or proposal flowchart
- User mentions a specific platform (n8n, Make.com, Zapier, GHL, Pabbly, etc.)
- User wants to visualize what they'd build before writing a proposal

---

## Tool

```bash
npx @swiftlysingh/excalidraw-cli create --inline "DSL" -o output.excalidraw
```

---

## DSL Syntax Reference

| Syntax | Element | Use For |
|--------|---------|---------|
| `[Label]` | Rectangle | Automation nodes / actions / steps |
| `{Label?}` | Diamond | Conditional logic / filters / routers |
| `(Label)` | Ellipse | Trigger / Start / End |
| `[[Label]]` | Database | Data stores, CRMs, spreadsheets |
| `->` | Arrow | Flow connection |
| `-> "label" ->` | Labeled Arrow | Conditional branch label |
| `-->` | Dashed Arrow | Optional or fallback path |

### Directives

| Directive | Description | Example |
|-----------|-------------|---------|
| `@direction` | Flow direction | `@direction TB` or `@direction LR` |
| `@spacing` | Node spacing in pixels | `@spacing 60` |

---

## Step 1: Analyze the Job Description

### A. Platform Detection
- Is a specific platform named? (n8n, Make.com, Zapier, Go High Level, Pabbly, ActivePieces, etc.)
- If no platform is named, infer from context:
  - Marketing/CRM automation → GHL or ActiveCampaign
  - Developer-friendly / self-hosted → n8n
  - Non-technical / simple → Make.com or Zapier
- **Always label each workflow with its platform name**

### B. Workflow Count
Decide how many workflows to draw **inside the single file**:
- One workflow = one linear or branching diagram
- Multiple workflows when:
  - The job describes clearly separate automations (different triggers, different purposes)
  - "Also need..." introduces a distinct process
  - Different trigger types appear (e.g., form submit + scheduled report + webhook)
- Each workflow gets its own diagram placed **side by side** in the canvas
- Each diagram is headed by a bold platform label: e.g., `=== n8n: Lead Intake Workflow ===`

### C. Focus: Automation Nodes Only
**Only map what happens inside the automation tool.** Do NOT include:
- Real-world steps like "confirm tools with client", "install software", "get API keys"
- Consulting or onboarding steps
- Human decisions outside the automation

Every node should represent something that happens **inside the platform** — a trigger, an action, a filter, a data operation.

---

## Step 2: Platform Node Libraries

Use these exact names when labeling workflow nodes.

### n8n
**Triggers:** `Webhook`, `Schedule Trigger`, `Gmail Trigger`, `Google Sheets Trigger`, `Airtable Trigger`, `HTTP Request (Polling)`
**Actions:** `HTTP Request`, `Set`, `Code`, `IF`, `Switch`, `Merge`, `Split In Batches`, `Wait`, `Send Email (Gmail)`, `Google Sheets`, `Airtable`, `Notion`, `Slack`, `OpenAI`, `Postgres`, `MySQL`, `Send Email (SMTP)`, `Respond to Webhook`, `Function`, `Aggregate`, `Filter`

### Make.com
**Triggers:** `Watch Records`, `Watch New Emails`, `Watch Rows`, `Webhook (Custom)`, `Scheduler`
**Actions:** `Send an Email`, `Create a Record`, `Update a Record`, `Search Records`, `HTTP - Make a Request`, `JSON - Parse JSON`, `Tools - Set Variable`, `Router`, `Filter`, `Iterator`, `Array Aggregator`, `OpenAI - Create a Completion`, `Google Sheets - Add a Row`, `Airtable - Create a Record`, `Slack - Send a Message`, `Notion - Create a Page`

### Zapier
**Triggers:** `Trigger: [App] - [Event]`, `Webhook by Zapier`, `Schedule by Zapier`
**Actions:** `Action: [App] - [Event]`, `Filter by Zapier`, `Paths by Zapier`, `Formatter by Zapier`, `Delay by Zapier`, `Code by Zapier`, `Looping by Zapier`, `Storage by Zapier`

### Go High Level (GHL)
**Triggers:** `Form Submitted`, `Appointment Scheduled`, `Tag Added`, `Contact Created`, `Pipeline Stage Changed`, `Inbound Webhook`, `Email Opened`, `Link Clicked`
**Actions:** `Send Email`, `Send SMS`, `Add Tag`, `Remove Tag`, `Create Contact`, `Update Contact`, `Add to Pipeline`, `Move Pipeline Stage`, `Create Opportunity`, `Wait`, `If/Else`, `Go To`, `Create Task`, `Assign User`, `Send Internal Notification`, `Webhook`

### Pabbly Connect
**Triggers:** `Trigger App + Event`, `Webhook`
**Actions:** `Action App + Event`, `Router`, `Filter`, `Iterator`, `API Request`, `Text Formatter`, `Date/Time Formatter`, `Number Formatter`

---

## Step 3: Build the DSL

### Multiple Workflows = Multiple Diagram Blocks
Use `@workflow` sections to separate diagrams. Each section produces a workflow that will be placed side by side in the final canvas. Begin each with a descriptive title and platform name.

```
@workflow "=== n8n: Lead Intake Workflow ==="
@direction TB
@spacing 60

(Webhook) -> [Set: Extract Fields] -> [IF: Has Email?]
[IF: Has Email?] -> "yes" -> [Google Sheets: Add Row] -> [Send Email (Gmail)] -> (Done)
[IF: Has Email?] -> "no" -> [Slack: Notify Team] -> (End)

@workflow "=== Make.com: Daily Report Workflow ==="
@direction TB
@spacing 60

(Scheduler) -> [Google Sheets - Get Rows] -> [Iterator] -> [HTTP - Make a Request] -> [Array Aggregator] -> [Send an Email] -> (Done)
```

### Node Naming Rules
| Node Type | Format | Example |
|-----------|--------|---------|
| Trigger | Platform trigger name | `(Webhook)`, `(Form Submitted)` |
| Action | `[Platform Node: Detail]` | `[Google Sheets: Add Row]`, `[Send SMS]` |
| Condition | `{Short Question?}` | `{Has Email?}`, `{Tag Exists?}` |
| Data Store | `[[Name]]` | `[[CRM Contacts]]`, `[[Google Sheet]]` |
| End | `(Done)` or `(End)` | |

Keep labels short (2–5 words). Use imperative verbs: "Add Row", "Send Email", "Parse JSON".

---

## Step 4: Reasoning Process

### 1. Map the Happy Path First
What does the automation do when everything works? Build that spine first.

### 2. Add Decision Points
Look for: "if", "when", "depending on", "based on", conditional logic, success/failure of API calls.

### 3. Add Error / Fallback Branches
External calls (APIs, webhooks, emails) can fail. Add:
- `{Success?}` after HTTP requests
- Fallback paths to logging or notifications

### 4. Add Loops Where Needed
- Batch processing: `Split In Batches` (n8n) / `Iterator` (Make)
- Retry logic: loop back to the failing node with a `Wait`

### 5. Decide: One or Multiple Workflows?
Ask: do these share the same trigger and flow as one sequence, or are they separate automations that happen independently? If separate → separate workflow blocks in the DSL.

---

## Common Automation Patterns

### Lead Capture → CRM + Notification (GHL)
```
@workflow "=== GHL: Lead Capture Workflow ==="
@direction TB
@spacing 60

(Form Submitted) -> [Create Contact] -> [Add Tag: New Lead] -> [Add to Pipeline] -> [Send Email: Welcome] -> [Wait: 1 Day] -> [Send SMS: Follow Up] -> (Done)
```

### AI Content Pipeline (n8n)
```
@workflow "=== n8n: AI Content Pipeline ==="
@direction TB
@spacing 60

(Schedule Trigger) -> [Google Sheets: Get Row] -> [OpenAI: Generate Content] -> {Quality OK?}
{Quality OK?} -> "yes" -> [Notion: Create Page] -> [Slack: Notify Team] -> (Done)
{Quality OK?} -> "no" -> [Google Sheets: Flag Row] -> (End)
```

### Multi-Step Scrape + Store (Make.com)
```
@workflow "=== Make.com: Scrape & Store Workflow ==="
@direction TB
@spacing 60

(Scheduler) -> [HTTP - Make a Request: Scrape] -> {Got Data?}
{Got Data?} -> "no" -> [Send an Email: Alert] -> (End)
{Got Data?} -> "yes" -> [JSON - Parse JSON] -> [Iterator] -> [Airtable - Create a Record] -> [Array Aggregator] -> (Done)
```

### Zapier Multi-Path by Source
```
@workflow "=== Zapier: Inbound Lead Router ==="
@direction TB
@spacing 60

(Trigger: Typeform - New Entry) -> [Paths by Zapier]
[Paths by Zapier] -> "Source = Ads" -> [Action: HubSpot - Create Contact] -> [Action: Slack - Send Message] -> (Done)
[Paths by Zapier] -> "Source = Organic" -> [Action: Mailchimp - Add Subscriber] -> (Done)
```

---

## Execution Command

For a single workflow:
```bash
npx @swiftlysingh/excalidraw-cli create --inline "$(cat <<'EOF'
@direction TB
@spacing 60

(Webhook) -> [Set: Extract Fields] -> [IF: Has Email?]
[IF: Has Email?] -> "yes" -> [Google Sheets: Add Row] -> [Send Email (Gmail)] -> (Done)
[IF: Has Email?] -> "no" -> [Slack: Notify Team] -> (End)
EOF
)" -o .tmp/workflow_name.excalidraw
```

For multiple workflows (all in one file, side by side):
```bash
npx @swiftlysingh/excalidraw-cli create --inline "$(cat <<'EOF'
@workflow "=== n8n: Workflow One ==="
@direction TB
@spacing 60

(Webhook) -> [Step A] -> [Step B] -> (Done)

@workflow "=== Make.com: Workflow Two ==="
@direction TB
@spacing 60

(Scheduler) -> [Step X] -> [Step Y] -> (Done)
EOF
)" -o .tmp/workflow_name.excalidraw
```

---

## Output Format

Always output as a single `.excalidraw` file. Multiple workflows appear **side by side** in the canvas, each with a labeled header. The file can be:
- Opened at https://excalidraw.com via File > Open
- Edited and rearranged freely in Excalidraw
- Exported to PNG/SVG from Excalidraw

---

## File Naming Convention

Use descriptive snake_case based on the job:
- `lead_intake_crm_sync.excalidraw`
- `ai_content_pipeline.excalidraw`
- `multi_platform_lead_router.excalidraw`

All outputs go to `.tmp/` directory.

---

## Output Summary to User

After generating, always tell the user:
1. **File:** `.tmp/filename.excalidraw`
2. **How to open:** https://excalidraw.com → File → Open
3. **Workflows included:** a brief table listing each workflow name, its platform, and what it does

| Workflow | Platform | Purpose |
|----------|----------|---------|
| Lead Intake | GHL | Captures form submissions, tags contact, starts email sequence |
| Daily Report | Make.com | Pulls sheet data, aggregates, emails summary |

---

## Worked Example

### Input: Upwork Job Description

> "I need someone to set up two automations in n8n. First: when a new lead fills out a Typeform, add them to Airtable, send a welcome email via Gmail, and notify our Slack channel. Second: every morning at 8am, pull all leads added yesterday from Airtable, generate a summary report using OpenAI, and email it to our team."

### Analysis

**Platform:** n8n (explicitly stated)
**Workflow count:** 2 (clearly separate — different triggers, different purposes)
- Workflow 1: Typeform trigger → Airtable + Gmail + Slack
- Workflow 2: Schedule trigger → Airtable → OpenAI → Gmail

### DSL

```
@workflow "=== n8n: Workflow 1 — Lead Intake ==="
@direction TB
@spacing 60

(Webhook: Typeform) -> [Set: Extract Lead Fields] -> [Airtable: Add Record] -> [Send Email (Gmail): Welcome] -> [Slack: Notify Channel] -> (Done)

@workflow "=== n8n: Workflow 2 — Daily Lead Report ==="
@direction TB
@spacing 60

(Schedule Trigger: 8AM Daily) -> [Airtable: Get Yesterday's Leads] -> [Split In Batches] -> [OpenAI: Summarize Leads] -> [Merge] -> [Send Email (Gmail): Team Report] -> (Done)
```

### Execution Command

```bash
npx @swiftlysingh/excalidraw-cli create --inline "$(cat <<'EOF'
@workflow "=== n8n: Workflow 1 — Lead Intake ==="
@direction TB
@spacing 60

(Webhook: Typeform) -> [Set: Extract Lead Fields] -> [Airtable: Add Record] -> [Send Email (Gmail): Welcome] -> [Slack: Notify Channel] -> (Done)

@workflow "=== n8n: Workflow 2 — Daily Lead Report ==="
@direction TB
@spacing 60

(Schedule Trigger: 8AM Daily) -> [Airtable: Get Yesterday's Leads] -> [Split In Batches] -> [OpenAI: Summarize Leads] -> [Merge] -> [Send Email (Gmail): Team Report] -> (Done)
EOF
)" -o .tmp/n8n_lead_workflows.excalidraw
```

### Output Summary

| Workflow | Platform | Purpose |
|----------|----------|---------|
| Lead Intake | n8n | Typeform → Airtable → Gmail welcome → Slack alert |
| Daily Lead Report | n8n | Scheduled pull from Airtable → AI summary → team email |
