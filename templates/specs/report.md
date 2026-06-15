# report.html — Content Contract
# Written to runs/<run-id>/report.html after every run.
# Copied (alongside report.json) to reports/latest/ and reports/history/<run-id>/.

---

## How to Build This Report

The reporter MUST follow this construction sequence:

**Step 1 — Read source data**
Read `runs/<run-id>/execution.json` in full before writing a single line of HTML.
This file contains every TC, action, step, assertion, and screenshot path.

**Step 2 — Embed screenshots as base64**
For every screenshot path found in `execution.json` steps:
1. Read the PNG file from disk at the given path (relative to the project root)
2. Base64-encode the raw bytes
3. Replace the path with `data:image/png;base64,<encoded-bytes>` in the HTML `<img>` tag
⛔ NEVER use a file path (`src="artifacts/..."`) in the final HTML — the file will not
   be accessible when viewed from `reports/latest/` or shared externally.

**Step 3 — Build steps table from execution.json**
For each TC → for each action in `actions[]` → for each entry in `steps[]`:
one `<tr>` in the Steps table. Column mapping:
- `#` → running step index across all actions in the TC (1, 2, 3 …)
- `Description` → `step.description`
- `Tool` → `step.tool`
- `Status` → `step.result` (✅ PASS / ❌ FAIL / ⚠️ WARN)
- `Duration` → `step.duration_ms` formatted per Duration rules (`null` → `—`)
- `Healed` → `step.healed === true` → `✓ healed`, else blank
- `Screenshot` → base64 `<img>` if `step.screenshot` is set, else blank

**Step 4 — Build assertions table from execution.json**
For each TC → for each entry in `expected[]` (or `assertions[]`):
one `<tr>` in the Assertions table. Column mapping:
- `#` → assertion index
- `Expected` → `assertion.description`
- `Result` → `assertion.status` → `✓ PASS` (green) or `✗ FAIL` (red)
- `Actual observed state` → `assertion.evidence`

**Step 5 — PASS collapsed, FAIL open**
- `<details>` with NO `open` attribute → TC is collapsed by default (PASS)
- `<details open>` → TC is expanded by default (FAIL or SKIP)
⛔ Do NOT add `open` to PASS test cases.

---

## Self-Contained Rule

`report.html` MUST be fully self-contained:
- No external CDN links — all styles in an inline `<style>` block.
- All screenshots embedded as base64 data URIs — see Step 2 above.
- The file MUST render correctly when viewed from `reports/latest/` where no
  screenshot folder exists alongside it.

---

## Duration Format Rules

Apply everywhere duration is shown:

| Range        | Format        | Example  |
|--------------|---------------|----------|
| < 60 s       | `Ns` / `N.Ns` | `45.2s`  |
| 60 s–3599 s  | `Nm Ns`       | `1m 23s` |
| ≥ 3600 s     | `Nh Nm`       | `1h 02m` |

---

## 1. Header

Suite name · Run ID · Environment · Browser · LLM provider/model · generated timestamp
Overall status badge: `✅ PASSED` or `❌ FAILED` — coloured green / red

---

## 2. KPI Bar

Six cards, one each — read all values from `report.json`:

| Card | Source field |
|------|-------------|
| Total | `total` |
| Passed | `passed` |
| Failed | `failed` |
| Skipped | `skipped` |
| Pass Rate | `pass_rate` |
| Total Duration | `duration_seconds` → formatted per Duration rules |

---

## 3. Per-Test-Case Section

One `<details>`/`<summary>` block per TC — read from `execution.json`:

| TC status | `<details>` attribute | Sub-sections rendered?      |
|-----------|----------------------|-----------------------------|
| PASS      | *(none — collapsed)* | yes                         |
| FAIL      | `open`               | yes                         |
| SKIP      | `open`               | no — show `skip_reason` only |

Summary row (always visible in `<summary>`): TC-ID · title · status badge · duration

Expanding reveals three sub-sections:

### Steps

One row per step entry from `execution.json` → TC → action → `steps[]`:

| # | Description | Tool | Status | Duration | Healed | Screenshot |
|---|-------------|------|--------|----------|--------|------------|

- **Status**: `✅` PASS · `❌` FAIL · `⚠️` WARN (from `step.result`)
- **Duration**: `step.duration_ms` formatted per Duration rules; `null` → `—`
- **Healed**: `✓ healed` if `step.healed === true`; blank otherwise
- **Screenshot**: `<img src="data:image/png;base64,..." style="max-width:300px;cursor:pointer" onclick="this.style.maxWidth=this.style.maxWidth?'':'300px'">` (base64 encoded from disk). Blank if no screenshot.

### Assertions

One row per entry from `execution.json` → TC → `expected[]`:

| # | Expected | Result | Actual observed state |
|---|----------|--------|-----------------------|

- **Result**: `✓ PASS` (green) or `✗ FAIL` (red)
- **Actual observed state**: `assertion.evidence`

### Defects *(render only if TC has defects)*

One row per defect — read from `execution.json` → TC → `defects[]`:
- `BUG-NNN` · severity badge (`CRITICAL` / `HIGH` / `MEDIUM` / `LOW`)
- Expected: *one-line summary*
- Actual: *one-line summary*

