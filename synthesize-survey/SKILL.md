---
name: synthesize-survey
description: "Synthesize Sprig survey CSV exports into structured Notion reports with distributions, cross-tabs, themes, and optional QoQ/behavioral analysis. Works for recurring benchmarking surveys and one-off targeted surveys. Category: 🔍 User Research"
argument-hint: <csv-path> [--baseline <prior-csv>] [--behavior <looker-csv>] [--spec <notion-url>]
allowed-tools: Read, Bash, Write, Edit, Glob, Grep, WebFetch, mcp__notion__notion-create-pages, mcp__notion__notion-update-page, mcp__notion__notion-fetch, mcp__notion__notion-search
---

# Synthesize Survey

Analyze a Sprig survey CSV export and produce a structured Notion report with statistical analysis, cross-tabs, open-text themes, and actionable findings.

## Usage

```
/synthesize-survey /path/to/survey.csv
/synthesize-survey /path/to/survey.csv --baseline /path/to/prior_quarter.csv
/synthesize-survey /path/to/survey.csv --behavior /path/to/looker_export.csv
/synthesize-survey /path/to/survey.csv --spec https://notion.so/Survey-Spec-Page-ID
```

**Arguments:**
- `<csv-path>` (required) — Path to the Sprig CSV export
- `--baseline <csv>` (optional) — Prior period CSV for QoQ/comparison deltas
- `--behavior <csv>` (optional) — Looker behavioral data CSV with userId + metrics for cohort analysis
- `--spec <notion-url>` (optional) — Notion survey spec page. If provided, the skill pulls the hypothesis, cross-tab plan, and design constraints directly from the spec instead of inferring them.

---

## Step 1: Parse Arguments & Detect Survey Type

### 1a. Parse CLI arguments
Extract the CSV path and optional flags. If `--spec` is provided, fetch the Notion page to get:
- Survey hypothesis / objective
- Pre-defined cross-tab plan
- Design constraints (e.g., "do NOT include BNPL")
- Question intent annotations (the `>` blockquote rationale under each question)

### 1b. Read & parse the CSV
Use Python (via Bash) for all CSV analysis. The Sprig export format is:
- **Row 1:** Column IDs (machine names like `Q1_Response`, `Q3_Theme_1`, `Q4_Choice_3`)
- **Row 2:** Human-readable labels (question text and choice labels)
- **Rows 3+:** Response data

```bash
python3 << 'PYEOF'
import csv, json, sys, re
from collections import defaultdict, Counter

with open("PATH_TO_CSV", "r") as f:
    reader = csv.reader(f)
    col_ids = next(reader)
    col_labels = next(reader)
    rows = list(reader)

# ... analysis code ...
PYEOF
```

### 1c. Auto-detect question types
Scan column IDs to classify each question. The pattern is always `Q{n}_{suffix}`:

| Column pattern | Type | Analysis |
|---|---|---|
| `Q{n}_Value` exists | **Likert/numeric scale** | Mean, distribution, % at each level |
| `Q{n}_Choice_1`, `Q{n}_Choice_2`, ... | **Multi-select** | % selected per choice, sorted desc |
| `Q{n}_Theme_1`, `Q{n}_Theme_2`, ... | **Open-text with themes** (Sprig AI-coded) | Theme frequency, representative quotes |
| `Q{n}_Response` only, text values | **Open-text free response** | Cluster into themes, pull quotes |
| `Q{n}_Response` only, numeric values | **Single-select or scale** | Distribution |
| `Q{n}_Response` with values like "Yes"/"No"/"Clicked" | **Gate/screener** | % yes/no, note downstream gating |

Also extract from `Q{n}_Response_Other` columns (free text for "Other" selections).

### 1d. Compute basic metadata
- Survey name (from `surveyName` column)
- Total responses (row count)
- Completions (rows where `completedAt` is not empty)
- Completion rate
- Date range (min/max `createdAt`)
- Platform split (from `os` column: iOS / Android / Web)
- Question count

### 1e. Detect survey mode
Based on question count and structure:
- **Benchmarking mode** (15+ questions, multiple pillars, scale questions across domains): Full scorecard output with pillar groupings
- **Targeted mode** (3-10 questions, focused topic): Hypothesis-driven output with cross-tab analysis

---

## Step 2: Compute Core Statistics

Run all computations in Python. For each question, based on detected type:

### Likert/Scale questions
- Mean score (1 decimal)
- Score distribution: count and % at each level (1 through 5)
- Top-2 box (% scoring 4 or 5) and Bottom-2 box (% scoring 1 or 2)
- n (respondents who answered)

### Multi-select questions
- % of respondents who selected each choice, sorted descending
- n (total respondents for this question)
- Flag any notable "Other" free-text responses (group similar ones)
- If the question spec says "select up to 3", note the cap

### Open-text with themes (Sprig AI-coded)
- Theme frequency: count each `Q{n}_Theme_*` column where value = 1
- Sort by frequency descending
- Pull 2-3 representative `Q{n}_Response` quotes for each top theme
- Note: Sprig theme coding methodology may differ between surveys — flag this in output

### Open-text (uncoded)
- Read all non-empty responses
- Cluster into 5-8 themes using your own judgment
- For each theme: count, 2-3 representative quotes, and a 1-sentence "so what"
- Flag quotes that directly reference product features or competitors

### Single-select / Screener questions
- % for each response option
- n respondents
- Note if this question gates downstream questions (e.g., "Have you ever sold?" gates selling questions)

---

## Step 3: Cross-Tab Analysis

### 3a. If `--spec` provided: follow the spec's cross-tab plan
The survey spec defines specific cross-tabs (e.g., "Q1 awareness x Q2 barriers"). Execute each one:
- For each cross-tab pair, compute a contingency table
- Highlight cells where a segment over- or under-indexes by >=8pp vs the overall population
- Include n per cell so the reader can judge reliability
- Add a 1-sentence interpretation for each cross-tab

### 3b. If no spec: auto-detect useful cross-tabs
- Cross-tab every multi-select/scale question against every screener/gate question
- Cross-tab the first question (often PMF or screener) against all others
- Flag any segment that over-indexes by >=8pp on any choice
- Limit to the 5-8 most insightful cross-tabs (skip trivially similar ones)

### 3c. Behavioral cross-tabs (if `--behavior` provided)
The behavior CSV should have columns joinable on `userId`:
- Expected: `userId`, `buyGmv`, `sellGmv`, `buyOrders`, `sellOrders`, `buyAov`, `sellAov`, plus any category or tenure columns
- Classify users: **Overlapper** (buy+sell), **Buyer-only**, **Seller-only**, **Inactive** (no orders)
- For interesting survey cohorts (low scores, specific selections), compute behavioral profile:
  - Mean/median GMV, AOV, order count
  - User type distribution (overlapper/buyer/seller)
  - Over-indexed categories or behaviors
- Cross-tab survey responses by behavioral segments (e.g., whale buyers vs light buyers)

### 3d. Event properties cross-tabs
Check the `eventProperties` column — Sprig often passes server-side context (e.g., `is_bundle`, `category_0`, `order_value`). If present:
- Parse the JSON
- Cross-tab survey responses against these event properties
- This is especially valuable for targeted surveys where the trigger carries context

---

## Step 4: QoQ / Baseline Comparison (if `--baseline` provided)

### 4a. Parse the baseline CSV
Use the same question detection logic from Step 1.

### 4b. Match questions between current and baseline
Match by `Q{n}_Question_Text` content (not by Q number, since question order may shift between quarters).

### 4c. Compute deltas
For each matched question:
- **Scale questions:** Absolute change in mean (e.g., 3.4 → 3.52 = +0.12)
- **Multi-select:** Percentage point change per choice (e.g., 43% → 52.5% = +9.5pp)
- **Distribution shifts:** Change in top-2 box and bottom-2 box

### 4d. Statistical significance
With ~1,000 responses per metric:
- Scale questions: changes >=±0.10 are statistically significant
- Percentage metrics: changes >=±4pp are statistically significant
- For smaller n (< 200): widen thresholds to ±0.20 and ±8pp
- Always note the n when flagging significance

### 4e. Build scorecard
Generate a comparison table:

| Metric | Prior | Current | Change | Sig? |
|---|---|---|---|---|
| [Question summary] | [value] | [value] | [delta] | [Yes/No] |

Color-code: green for improvement, red for decline, gray for within noise.

---

## Step 5: Synthesize Findings

Before writing the Notion page, synthesize the raw statistics into findings. This is the most important step — don't just report numbers, interpret them.

### 5a. Identify the 3-6 key findings
For each finding:
1. **Title:** A clear, opinionated statement (not "Q3 results" but "Timing is the #1 barrier, not awareness")
2. **Evidence:** The specific data that supports it (with n and %)
3. **So what:** Why this matters for the product
4. **Action:** A concrete next step (not "investigate further" but "ship a nudge prompt when buyer purchases from a repeat seller")

### 5b. Write TL;DR
3-4 bullets, executive-level:
- Lead with the most surprising or actionable finding
- Include the key number that makes it stick
- End with the strategic implication

### 5c. Flag open questions
What does the data NOT answer? What follow-up research or analysis would close the gap?

---

## Step 6: Generate Notion Page

Create the page in the US Documents database: `collection://30a7fa9f-faef-80e5-9ee7-000b6f11ae2b`

### Page structure — Targeted survey (3-10 questions)

```
# [Survey Name] {color="purple_bg"}

<callout icon="📋" color="gray_bg">
  **Conducted:** [date range] | **n = [total] responses ([completions] completions)** | **Method:** [trigger + audience from spec or CSV metadata]
</callout>

## Hypothesis & Objective
[From spec if provided, otherwise ask the user]

---

## TL;DR {color="blue_bg"}
- [3-4 bullet executive summary]

---

## Key Findings {color="gray_bg"}

### 1. [Finding title — opinionated, not just "Q1 Results"]
[Narrative: data + context + so what]
**Action:** [Concrete next step]

### 2. [Finding]
...

### 3-6. [Additional findings]

---

## 📊 Question-by-Question Results {color="purple_bg"}

### Q1: [Question text]
*[Question type] | n = [count]*

[Distribution table or choice breakdown]

<callout icon="💡" color="gray_bg">
  [1-2 sentence interpretation]
</callout>

### Q2: [Question text]
...

---

## 🔀 Cross-Tab Analysis {color="purple_bg"}

### [Cross-tab title, e.g., "Awareness × Barriers"]
[Contingency table with over-indexed cells highlighted]

<callout icon="💡" color="gray_bg">
  [Interpretation]
</callout>

...

---

## 📝 Open Text Themes {color="purple_bg"} (if applicable)
[Theme table with counts and representative quotes]

---

## 🔬 Behavioral Deep Dive {color="purple_bg"} (if --behavior provided)
[Cohort profiles, GMV tables, behavioral cross-tabs]

---

## ❓ Open Questions
- [What the data doesn't answer]
- [Suggested follow-up]

---

## Methodology
- Survey platform: Sprig
- Trigger: [from metadata/spec]
- Sample: [n] responses, [completions] completions ([rate]%)
- Platform: [iOS/Android/Web split]
- [Statistical significance note if applicable]
```

### Page structure — Benchmarking survey (15+ questions, with QoQ)

```
# [Survey Name] [Quarter] {color="purple_bg"}

<callout icon="📋" color="gray_bg">
  **Conducted:** [date] | **n = [total] responses ([completions] completions)** | **Method:** [description]
</callout>

## Objective
[Survey objective]

---

<callout icon="📊" color="gray_bg">
  **Statistical significance note:** [Sample size context, threshold explanation]
</callout>

---

## 📊 Scorecard {color="gray_bg"}
[QoQ comparison table with color-coded changes]

---

## TL;DR {color="blue_bg"}
- [3-4 bullets]

## Key Findings & Actions {color="gray_bg"}

### 1. [Finding]
[Narrative + data]
**Action:** [Next step]

...

---

## [Pillar sections with deep dives in <details> toggles]

### [Pillar Name] {color="purple_bg"}
[Score + distribution + key callouts]

<details>
<summary>[Distribution detail]</summary>
  [Tables and charts]
</details>

...

---

## 📝 Open Text Themes {color="purple_bg"}
[Theme table: theme | count | representative quote | cross-question signal]

---

## 🔬 Behavioral Cohort Deep Dive {color="purple_bg"} (if --behavior)
[Full cohort analysis: definitions, GMV, cross-tabs, intersections]

---

## Methodology & Validation
[Sample, trigger, platform split, significance thresholds, validation notes]
```

### Notion formatting rules
- Use `{color="purple_bg"}` for major section headers
- Use `{color="gray_bg"}` for subsection headers
- Use `{color="blue_bg"}` for TL;DR and strategic callouts
- Use `<callout icon="💡" color="gray_bg">` for per-question insights
- Use `<callout icon="🎯" color="blue_bg">` for action items
- Use `<callout icon="📋" color="gray_bg">` for metadata
- Use `<details><summary>` for detailed data that most readers will skip
- Use `<table header-row="true" fit-page-width="true">` for data tables
- Color-code table cells: `color="green_bg"` for improvements, `color="red_bg"` for declines, `color="yellow_bg"` for small moves, `color="gray_bg"` for flat/noise
- Use `<columns>` for side-by-side layouts (narrative left, table right)

---

## Step 7: Confirm & Output

1. **Print the Notion page URL** so the user can open it immediately
2. **Print the TL;DR** in the terminal (3-4 lines)
3. **Flag any data quality issues** (low n on a question, high drop-off, suspicious patterns)
4. If behavioral data was provided, **flag cohorts worth recruiting** for follow-up interviews (e.g., "24 users are both scam-fearful AND detractors — good Wave 3 recruit list")
5. If a survey spec was provided, **note which cross-tabs from the spec were completed** and which couldn't be done (e.g., missing server-side data)

---

## Quality Standards

These standards apply to ALL output from this skill:

### Statistical rigor
- Always report n alongside percentages
- Flag when n < 30 for any segment ("directional only")
- Don't over-interpret small differences — note when changes are within sampling noise
- For cross-tabs, only highlight segments that differ from baseline by >=8pp

### Writing voice
- Plain, direct language. No consultant-speak or McKinsey jargon.
- Short sentences. Concrete mechanisms.
- No double em-dashes.
- Findings should be opinionated: "Timing is the barrier, not awareness" not "Several barriers were identified"
- Always connect data to product decisions: "so what?" for every finding

### Open-text handling
- Pull actual user quotes — they're the most persuasive part of the report
- Escape special characters in quotes for Notion markdown
- Attribute quotes anonymously (no user IDs in the report body)
- Group similar "Other" responses and report the cluster, not individual responses

### Behavioral data
- When joining on userId, report match rate (e.g., "1,178 of 1,751 respondents matched to L12M behavioral data")
- Flag inactive users (no L12M orders) separately — their survey responses are valid but their behavioral profile is different
- Use user type labels: Overlapper, Buyer-only, Seller-only, Inactive

---

## Error Handling

- If the CSV is empty or has no data rows: tell the user and stop
- If question detection fails (unexpected column format): print the column names and ask the user to clarify
- If `--baseline` CSV has no matching questions: warn and proceed without QoQ
- If `--behavior` CSV has no userId column: warn and proceed without behavioral analysis
- If `--spec` Notion page can't be fetched: warn and proceed without spec context
- If eventProperties column contains JSON: attempt to parse; if malformed, skip with warning
- If a question has <10 responses: flag as "insufficient data" and skip detailed analysis
