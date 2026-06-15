> 🤖 **If you are an AI agent:** Read **`AGENT.md`** first — that is your operating manual.

---

# qecopilot (🥝 KIWI) — AI-Native Test Automation Framework

> **Declarative YAML test cases. Zero automation scripts. Driven entirely by AI.**

qecopilot (nick name KIWI 🥝) is an open-source test automation framework where you write tests in plain-language YAML and run them three ways: via the **qecopilot CLI**, the **interactive TUI**, or directly through any **AI agent** (GitHub Copilot, Claude, Codex).

No test scripts. No selectors. No boilerplate. Just describe what to test.

---

## How It Works

1. You write test cases in YAML — steps, expected outcomes, test data
2. qecopilot (or your AI agent) reads the YAML and plans execution
3. The browser is driven through [Playwright MCP](https://github.com/microsoft/playwright-mcp) tools (`browser_navigate`, `browser_click`, etc.)
4. Screenshots, HTML/JSON reports, and a self-updating knowledge graph are written automatically

```
YAML Test Cases  →  qecopilot (Planner → Executor → Validator → Reporter)  →  Reports + Evidence
```

---

## Three Ways to Run

Choose the mode that fits your workflow — all three run the same YAML test cases.

### 1 · qecopilot CLI — direct command execution

```bash
qecopilot run suite smoke
qecopilot run test TC-001
qecopilot validate
```

Best for: **CI/CD pipelines**, automated scripts, quick targeted runs.

---

### 2 · qecopilot TUI — interactive terminal

```bash
qecopilot          # no arguments → launches full-screen TUI
```

On launch, the animated logo appears:

```
         ██████╗  ███████╗
        ██╔═══██╗ ██╔════╝  copilot - 🥝 KIWI
        ██║   ██║ █████╗    ──────────────────────────
        ██║▄▄ ██║ ██╔══╝    AI-Native Test Automation
        ╚██████╔╝ ███████╗  v1.0.0
         ╚══▀▀═╝  ╚══════╝

    Project  project-name  ·  Suite  smoke  ·  Env  qa
    Type help to see commands
```

Full-screen interface with autocomplete, live run progress, command history, and LLM status bar:

```
  ┌─────────────────────────────────────────────────────────────────────┐
  │  Run progress and responses appear here                             │
  ├─────────────────────────────────────────────────────────────────────┤
  │  qe ›  run suite _                                                  │
  │  ↑↓ history  ·  tab complete  ·  PgUp/Dn scroll  ·  ctrl+c exit     │
  ├─────────────────────────────────────────────────────────────────────┤
  │  1 suite  ·  3 tests               ✔ gpt-4o [work]                 │
  └─────────────────────────────────────────────────────────────────────┘
```

Best for: **local development**, exploratory testing, onboarding new team members.

---

### 3 · AI Agent Mode — natural language

Use any AI agent that has CLI or MCP access:

```
# GitHub Copilot CLI
Run the smoke suite

# Claude, Cursor, or any MCP-capable agent
Run suite smoke and show me the failures
```

The agent reads your YAML project, invokes qecopilot (or drives Playwright MCP directly), and streams results back into the conversation.

Best for: **developers who live in their AI agent** and want testing as part of their coding flow.

---

## Prerequisites

### 1 · Install qecopilot runtime

```bash
npm install -g @acr-edge/qecopilot
```

Both `qecopilot` and `qe` (shorthand) are available after install:

```bash
qecopilot --version
qe --version        # same thing
```

### 2 · Configure an LLM

qecopilot requires a reachable LLM to plan and validate test cases. The recommended way is a named global profile — set once, used across all projects.

```bash
qecopilot config profile add work
# Prompts for: provider (openai/anthropic/gemini/ollama/bedrock), model, API key
```

Or use environment variables for a quick start:

```bash
export OPENAI_API_KEY=sk-...
# or
export ANTHROPIC_API_KEY=sk-ant-...
# or
export GEMINI_API_KEY=AIza...
```

Verify the LLM is reachable:

```bash
qecopilot llm status
```

### 3 · Configure Playwright MCP (for AI Agent Mode only)

> **Skip this if you only use CLI or TUI mode** — qecopilot launches Playwright MCP automatically.

#### GitHub Copilot CLI

Create `~/.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "type": "local",
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--browser", "msedge"],
      "env": {},
      "tools": ["*"]
    }
  }
}
```

> **Windows:** Use `msedge` (pre-installed). **macOS/Linux:** Use `"chrome"` instead.

Or add interactively:
```
/mcp add
# Server Name: playwright
# Server Type: Local (STDIO)
# Command: npx @playwright/mcp@latest --browser msedge
```

Verify:
```
/mcp show playwright
```

#### Claude (Claude Code / Claude Desktop)

Add to `.claude/settings.local.json` in your project root:
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--browser", "msedge"]
    }
  }
}
```

#### GitHub Copilot Cloud Agent

No setup needed — Playwright MCP is built in.

#### OpenAI Codex

Configure per your Codex deployment environment.

---

## Quick Start

```bash
# 1. Scaffold a new project from the qecopilot template
qecopilot init my-app-tests
cd my-app-tests

# 2. Set up your environment
cp .env.example .env
# Edit .env: set BASE_URL and credentials for your app

# 3. Fill in application knowledge (optional but recommended)
# Edit knowledge/modules.json   — UI elements, button labels, field types
# Edit knowledge/entities.json  — data models, ID formats
# Edit knowledge/workflows.json — business workflows

# 4. Write your first test case
# Copy templates/testcase.yml → testcases/<feature>/TC-001.yml

# 5. Run it
qecopilot run suite smoke
# or launch TUI:
qecopilot
```

---

## CLI Reference

| Command | Description |
|---|---|
| `qecopilot run suite <name>` | Run all test cases in a named suite |
| `qecopilot run test <id>` | Run a single test case by ID |
| `qecopilot compile` | Pre-compile all test plans (AOT) — faster, deterministic runs |
| `qecopilot compile --suite <name>` | Pre-compile a specific suite only |
| `qecopilot compile --force` | Force recompile even if plans are up to date |
| `qecopilot runs [--limit <n>]` | List recent runs |
| `qecopilot validate` | Validate all YAML assets in the project |
| `qecopilot report latest` | Show path to the latest run report |
| `qecopilot report show <run-id>` | Show path to a specific run's report |
| `qecopilot llm status` | Check LLM provider health |
| `qecopilot mcp status` | Check browser adapter status |

### Global LLM Profile Commands

| Command | Description |
|---|---|
| `qecopilot config profile add <name>` | Add or replace a named LLM profile |
| `qecopilot config profile list` | List all profiles |
| `qecopilot config profile use <name>` | Set the active profile |
| `qecopilot config profile remove <name>` | Remove a profile |

Profiles are stored in `~/.qecopilot/config.json` and shared across all projects.

### TUI Commands (inside `qecopilot` interactive session)

| Command | Description |
|---|---|
| `run suite <name>` | Run a suite with live progress |
| `run test <id>` | Run a single test case |
| `list suites` | Show all suites |
| `list tests [<suite>]` | Show test cases |
| `validate` | Validate YAML assets |
| `config` | Show current LLM and project config |
| `help` | Show all available commands |
| `exit` / Ctrl+C | Exit |

### Exit Codes

| Code | Meaning |
|---|---|
| 0 | All tests passed |
| 1 | One or more tests failed |
| 2 | Config / YAML validation error |
| 3 | LLM provider unreachable |
| 4 | Application under test unreachable |

---

## Running with AI Agents (Copilot CLI)

Every new Copilot CLI session requires two setup commands:

```
/allow-all
```
Grants all tool, file, and URL permissions — without this, Copilot prompts for approval on every browser action.

```
/mcp show playwright
```
Verifies the Playwright MCP server is active. You should see `browser_navigate`, `browser_click` etc.

**Recommended flow every session:**
```
cd /path/to/my-app-tests
gh copilot

/allow-all
/mcp show playwright

Run the smoke suite
```

> `/allow-all` resets on exit — run it at the start of every new session.

---

## Framework Structure

```
qecopilot/
├── AGENT.md                   ← AI agent operating manual (read this first)
├── README.md
├── .env                       ← Runtime secrets (gitignored)
├── .env.example               ← Template — copy to .env
│
├── configs/
│   ├── ai-test-config.yml     ← Global settings (browser, suite, environment, artifacts)
│   ├── qa.yml                 ← QA environment (base_url, api_url, flags)
│   ├── uat.yml                ← UAT environment
│   └── prod.yml               ← Production environment
│
├── testcases/                 ← Test case definitions (YAML)
│   └── <feature>/TC-NNN.yml
│
├── actions/                   ← Reusable action libraries (YAML)
│   ├── action-catalog.yml     ← Master index of all actions
│   └── common/
│       ├── auth.yml           ← Login / logout
│       └── navigation.yml     ← Navigation actions
│
├── testdata/
│   ├── users.yml              ← User personas and credentials
│   ├── entities.yml           ← Domain entities
│   └── environments.yml       ← Environment URLs and flags
│
├── knowledge/                 ← Application knowledge graph (auto-updated each run)
│   ├── modules.json           ← UI elements, button labels, field types per module
│   ├── entities.json          ← Data models, ID formats, field constraints
│   ├── workflows.json         ← Business workflows and status transitions
│   ├── api_catalog.json       ← REST API endpoints
│   └── learning-log.json      ← Cross-run learning history
│
├── suites/
│   └── smoke.yml              ← Test suite definitions
│
├── templates/                 ← Blank templates for creating new assets
│   ├── testcase.yml
│   ├── action.yml
│   └── defect.md
│
├── pipelines/
│   └── execution-pipeline.yml
│
├── runs/                      ← Auto-generated run evidence (gitignored)
└── reports/                   ← Quick-access reports (gitignored)
    ├── latest/
    └── history/
```

---

## Configuration Reference

qecopilot reads `configs/ai-test-config.yml` from the project root. The LLM section is not in this file — it comes from your global profile or environment variables (see Prerequisites above).

```yaml
framework:
  name: "My Application Test Suite"   # your project name
  version: "1.0"
  spec: "AGENT.md"

execution:
  mode: mcp                        # mcp only (v1)
  browser: msedge                  # msedge | chrome | firefox | webkit
  headless: false                  # true for CI, false for local
  maximize: true
  timeout_seconds: 30
  page_load_timeout_seconds: 15
  retry_failed: 1                  # retry a failed step once before marking FAIL
  slow_mo_ms: 0                    # ms pause between actions (0 = no delay)

environment:
  target: qa                       # matches configs/<target>.yml
  config: configs/qa.yml

suite:
  name: smoke                      # matches suites/<name>.yml
  file: suites/smoke.yml

artifacts:
  base_path: runs/
  screenshots: true
  videos: false
  traces: false
  screenshot_on_action_boundary: true
  screenshot_on_failure: true

mcp:
  snapshot_before_action: true
  snapshot_after_action: false
  tool_timeout_seconds: 10

reporting:
  formats: [json, html]
  output_path: reports/
  copy_to_latest: true
  history_retention_runs: 50
```

### Environment file (`configs/qa.yml`)

```yaml
environment:
  name: qa
  base_url: "${env.QA_BASE_URL}"
  api_url: "${env.QA_API_URL}"
```

> **Single-environment project model:** one qecopilot project maps to one target environment. If you need to run the same automation against UAT, staging, or another environment, scaffold a **separate project folder** for that environment instead of switching envs at runtime. This keeps compiled plans, healing history, reports, and knowledge clean and easy to maintain.

---

## Writing Test Cases

Copy `templates/testcase.yml` to `testcases/<feature>/TC-NNN.yml`:

```yaml
test_case:
  id: TC-001
  title: "Create a new record"
  feature: "Records"
  priority: high
  persona: admin

  test_data:
    user: users.admin1

  steps:
    - Login as Admin
    - Navigate to Records
    - Create Record
    - Save

  step_params:
    Create Record:
      name: "${testdata.entities.record1.name}"

  expected:
    - "Record is created successfully"
    - "Record appears in the list"
```

---

## Writing Actions

Copy `templates/action.yml` to `actions/<category>/<name>.yml`:

```yaml
actions:
  - action_id: create_record
    name: "Create Record"
    description: "Opens the new record form and fills required fields"
    parameters:
      - name: name
        required: true
    steps:
      - description: "Click the New button"
        tool: browser_click
        params:
          element: "New Record button"
      - description: "Type record name"
        tool: browser_type
        params:
          element: "Name input"
          text: "${params.name}"
      - description: "Screenshot of filled form"
        tool: browser_take_screenshot
```

Register it in `actions/action-catalog.yml`:

```yaml
actions:
  - id: create_record
    file: actions/records/create-record.yml
```

---

## Test Results

**Quick access — `reports/latest/`** (and `reports/history/<run-id>/`):

| File | Contents |
|---|---|
| `report.html` | Human-readable pass/fail summary with screenshots |
| `report.json` | Machine-readable run summary |

**Full evidence — `runs/<run-id>/`**:

| Path | Contents |
|---|---|
| `artifacts/screenshots/` | Step screenshots named `<TC-ID>-<tool>-<step>.png` |
| `context/` | Resolved environment, test data, and execution plan used for the run |
| `defects/BUG-NNN.md` | Auto-generated defect report per failure |
| `ai-reasoning/` | Agent decision logs (planner, executor, validator, reporter) |
| `execution.log` | One-line-per-step audit log of every MCP tool call |
| `knowledge-updates.json` | UI element discoveries made during this run |

---

## Supported AI Agents

| Agent | Playwright MCP | Notes |
|---|---|---|
| qecopilot CLI / TUI | ✅ Auto-launched | No MCP setup needed — qecopilot manages it |
| GitHub Copilot CLI | ✅ Manual config | See Prerequisites above |
| GitHub Copilot Cloud Agent | ✅ Built-in | No setup needed |
| Claude Code / Claude Desktop | ✅ Via settings.local.json | See Prerequisites above |
| OpenAI Codex | ✅ Via deployment config | Configure per environment |

---

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.