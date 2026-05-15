---
name: create-survey
description: "Create a Sprig survey from a problem statement. Phase 1: asks narrowing questions and builds a Notion survey spec. Phase 2: after review, generates a Sprig-compatible upload file. Category: 🔍 User Research"
argument-hint: <problem-statement or Notion URL> [--sprig <notion-spec-url>]
allowed-tools: Read, Bash, Write, Edit, Glob, Grep, WebFetch, WebSearch, mcp__notion__notion-create-pages, mcp__notion__notion-update-page, mcp__notion__notion-fetch, mcp__notion__notion-search
---

# Create Survey

Two-phase workflow for building Sprig surveys:
- **Phase 1** (`/create-survey <problem>`): Narrow the hypothesis, design questions, output a Notion survey spec for review
- **Phase 2** (`/create-survey --sprig <notion-url>`): Convert a reviewed Notion spec into a Sprig upload file

## Usage

```
# Phase 1: Create a survey spec from a problem
/create-survey "Why don't repeat same-seller buyers bundle?"
/create-survey https://notion.so/Research-Plan-Page-ID

# Phase 2: Generate Sprig file from a reviewed spec
/create-survey --sprig https://notion.so/Survey-Spec-Page-ID
```

---

# Phase 1: Problem → Notion Survey Spec

## Step 1: Intake & Narrowing

### 1a. Get the problem
If a Notion URL is provided, fetch it and extract the research context. If text, use it directly.

### 1b. Ask narrowing questions
Before designing any questions, ask the user:

**What decision does this survey drive?**
- "What will you do differently based on the results?"
- "Who is the audience for the findings? What do they need to decide by when?"

**Who are we surveying?**
- "Describe the audience by behavior (not demographics). What filter or trigger defines them?"
- "What moment in the user journey should we catch them? (post-purchase, post-listing, app open, etc.)"

**What do we already know?**
- "What does existing data already tell us about this problem?"
- "Which hypotheses can we validate with a query before asking users?"

**Constraints?**
- "Max questions / time budget? (5 Qs = ~60 sec, 10 Qs = ~120 sec)"
- "Any topics to avoid? (e.g., don't include BNPL, don't mention competitors by name)"
- "Any mutual exclusions with other active surveys?"

### 1c. Triage hypotheses: data vs. survey
For each hypothesis the user wants to test, determine:

- **🟢 Data answers this** — "We can pull this from [table]. Run [query] first."
- **🟡 Data gives partial signal** — "Data shows X is happening but not why. Survey Q[n] covers the why."
- **🔴 Survey needed** — "No behavioral signal for this. Survey Q[n] directly tests it."

Present the triage. Only design survey questions for 🟡 and 🔴 hypotheses.

---

## Step 2: Design Survey Questions

### 2a. Question design rules

**Every question must pass this test:**
- Does it ask about **past behavior or current state**, not hypotheticals?
- Will the answer **change a product decision**?
- Is it **not answerable by data** we already have?

**Red flags — rewrite these immediately:**
- "Would you use X?" → "Have you ever done X? How?"
- "How important is X to you?" → "What did you actually do about X last time?"
- "Would you pay for X?" → "What are you paying for alternatives today?"
- "Do you usually X?" → "Walk me through the last time you X'd"
- Double-barreled questions ("Do you find it easy and useful?")
- Leading questions ("Don't you think X would help?")

**Question type selection:**

| Signal needed | Type | Notes |
|---|---|---|
| Prevalence / distribution | Single-select | Keep options MECE |
| Multiple behaviors / barriers | Multi-select (cap at 3) | Randomize options, anchor "Other" last |
| Priority ranking | Rank order / drag-to-rank | Max 4-5 items |
| Satisfaction / agreement | Rating scale (1-5) | Label endpoints clearly |
| Open exploration | Open text | Use sparingly — 1-2 max per survey |
| Segmentation / gating | Single-select screener | Gates downstream questions via branching |

### 2b. Question flow design

**Structure pattern (from reference surveys):**

1. **Screener / segmenter** (1-2 Qs) — classify the respondent so you can cross-tab everything downstream
2. **Core barrier / behavior questions** (2-4 Qs) — the main thing you're measuring
3. **Follow-up probes** (1-2 Qs) — deeper on the most interesting answer, often branched
4. **Open text** (0-1 Q) — "What's the single biggest thing that..." — forces prioritization
5. **Recruit opt-in** (optional) — "Would you be open to a paid 60-min interview?"

**Branching rules:**
- Only branch on screener questions (not every question)
- Keep max path ≤10 Qs
- Note the branch logic clearly: "If Q1 = X → show Q2a. If Q1 = Y → skip to Q3."

### 2c. Cross-tab plan
For each pair of questions, note the planned cross-tabs:
- What segment × what outcome = what insight
- Include server-side event properties if the trigger carries context (order value, category, etc.)

### 2d. Design constraints
Capture any constraints:
- Topics to exclude
- Max question count / time
- Mutual exclusions with other surveys
- Hard cap (1 response per user)

---

## Step 3: Create Notion Survey Spec Page

Create in US Documents database: `collection://30a7fa9f-faef-80e5-9ee7-000b6f11ae2b`

**Required properties:**
- `Team&PJ`: `["https://www.notion.so/2b07fa9ffaef807cbe3bfe33bdd8fdc9"]` (US UXR)
- `Category`: `["Survey Spec"]`
- `Status`: `"Draft"`

**Page structure:**

```
## Survey #[N] — [Title]
Part of [link to research plan if applicable]

---

## Decisions this survey drives {color="gray_bg"}

<table fit-page-width="true" header-row="true">
<tr>
<td>**Decision**</td>
<td>**How this survey answers it**</td>
<td>**Audience**</td>
</tr>
<tr>
<td>[Decision 1]</td>
<td>[Specific mechanism]</td>
<td>[Who needs this answer]</td>
</tr>
</table>

## Setup {color="gray_bg"}

<table fit-page-width="true" header-row="true">
<tr>
<td>**Field**</td>
<td>**Detail**</td>
</tr>
<tr><td>Audience</td><td>[Behavioral definition]</td></tr>
<tr><td>Trigger</td><td>[Event + conditions]</td></tr>
<tr><td>Sample</td><td>[Target n]</td></tr>
<tr><td>Max path</td><td>[N Qs (~X sec)]</td></tr>
<tr><td>Hard cap</td><td>1 response per user for entire field window</td></tr>
</table>

> **Why this audience?** [1-2 sentence rationale for the targeting choice]

---

## Questions {color="gray_bg"}

### Q1 — [Short label] *(question type)*

"[Question text]"

- [Option 1]
- [Option 2]
- [Option 3]
- Other (open text)

> [Rationale: why this question, what it tests, how it connects to the hypothesis]

### Q2 — [Short label] *(question type, randomized, ≤3)*

"[Question text]"

- [Options...]

> [Rationale]

*If [condition]:*

### Q2-follow — [Follow-up label] *(question type)*

"[Follow-up question text]"

...

### Q[N] — [Label] *(open text, required)*

"[Open text question — forces prioritization]"

### Q[N+1] — Recruit opt-in (optional)

"We're talking with a small number of [audience] next month — would you be open to a paid 60-min interview?"

- Yes → email field
- No

---

## Branching {color="gray_bg"}

```javascript
Q1 → Q2 → [if condition: Q2-follow] → Q3 → Q4
Q4 ─┬─ [condition A] → Q5 → Q6
    └─ [condition B] → Q6
```

---

## Key cross-tabs {color="gray_bg"}

- **Q1 × Q2** → [what this reveals]
- **Q1 × Q3** → [what this reveals]
- **Q2 × server-side [event property]** → [what this reveals]
...

---

## Data validation checklist {color="gray_bg"}

<table fit-page-width="true" header-row="true">
<tr>
<td>**Hypothesis**</td>
<td>**Data check**</td>
<td>**Status**</td>
</tr>
<tr>
<td>[H1 that data can answer]</td>
<td>[Query / dashboard to run]</td>
<td>🟢 / 🟡 / ⬜ Not started</td>
</tr>
</table>
```

---

## Step 4: Output & Next Steps

1. **Print the Notion page URL**
2. **Print a summary:** survey title, audience, question count, max path time, key hypotheses being tested
3. **Tell the user:** "Review the spec, get feedback, then run `/create-survey --sprig [notion-url]` to generate the Sprig upload file"

---

# Phase 2: Notion Spec → Sprig Upload File

Triggered by: `/create-survey --sprig <notion-url>`

## Step 1: Fetch the Notion spec

Fetch the survey spec page. Extract:
- Survey title
- Audience, trigger, sample, field window
- All questions with types, options, branching, randomization
- Design constraints

## Step 2: Convert to Sprig format

Generate a `.txt` file in Sprig's upload format:

```
SURVEY TITLE: [Title from spec]

SURVEY GOAL OR OBJECTIVE: [Pull from "Decisions this survey drives" table]

AUDIENCE: [From Setup table]
TRIGGER: [From Setup table]
SAMPLE TARGET: [From Setup table]
FIELD WINDOW: [From Setup table]
HARD CAP: [From Setup table]

DESIGN CONSTRAINTS:
- [Any constraints from the spec]

---

1. [Question text]? [Question Type]
   - Option 1
   - Option 2
   - Option 3
   - Other (open text)
   SKIP LOGIC: None
   RANDOMIZE OPTIONS: No
   REQUIRED: Yes

2. [Question text]? [Question Type]
   - Option 1
   - Option 2
   - Option 3
   SKIP LOGIC: None
   RANDOMIZE OPTIONS: Yes (anchor "Other" last)
   REQUIRED: Yes

2a. [Branched follow-up question]? [Question Type]
   - Option 1
   - Option 2
   SKIP LOGIC: Only shown if Q2 = "[condition]"
   RANDOMIZE OPTIONS: No
   REQUIRED: Yes

...
```

### Type mapping (Notion spec → Sprig format)

| Spec notation | Sprig type |
|---|---|
| single-select | `[Multiple Choice - Single]` |
| multi-select | `[Multiple Choice - Multi]` |
| multi-select, ≤3 | `[Multiple Choice - Multi] (max 3 selections)` |
| rating scale (1-5) | `[Rating Scale]` with `Scale:` line |
| NPS | `[NPS]` |
| rank order / drag-to-rank | `[Rank Order]` with `Items to rank:` prefix |
| matrix | `[Matrix]` with `Rows:` and `Columns:` sections |
| open text | `[Open Text]` with optional `Placeholder text:` |
| MaxDiff | `[MaxDiff]` with `Sets:` section |

### Branching conversion
- Notion spec uses `> Analysis note:` or inline branching descriptions
- Convert to: `SKIP LOGIC: If "[response]", go to Q[n]. If "[other response]", skip to Q[m].`

### Randomization conversion
- Notion spec says "randomized" in the question type annotation
- Convert to: `RANDOMIZE OPTIONS: Yes`
- If "Other" is an option: `RANDOMIZE OPTIONS: Yes (anchor "Other" last)`
- Default: `RANDOMIZE OPTIONS: No`

### Multi-select caps
- Notion spec says "≤3" or "pick up to 3"
- Convert to: `[Multiple Choice - Multi] (max 3 selections)`

### Required/Mandatory
- Default all questions to `REQUIRED: Yes`
- Open text follow-ups and recruit opt-ins: `REQUIRED: No`

## Step 3: Output

1. **Write the file** to the same directory as the working directory: `[survey-title-slugified]-sprig.txt`
2. **Print the file path** so the user can upload it to Sprig
3. **Print a quick checklist:**
   - Total questions: [n]
   - Max path: [n] Qs (~[time] sec)
   - Branches: [list branch points]
   - Randomized questions: [list]
   - Open text questions: [list]
4. **Remind:** "Upload this to Sprig via AI Study Builder. After uploading, verify the branching logic in Sprig's visual editor — complex branches sometimes need manual adjustment."

---

# Quality Standards

### Survey length
- 5 Qs ≈ 60 sec (ideal for in-flow surveys)
- 7-8 Qs ≈ 90-100 sec (max for most audiences)
- 10 Qs ≈ 120 sec (only for highly motivated audiences like power sellers)
- Beyond 10 Qs: push back. Completion rate drops sharply.

### Question quality
Every question must be:
- **Behavioral** — asks about what people did or do, not what they would do
- **Decision-driving** — the answer changes what gets built
- **Not data-answerable** — if you can query it, don't survey it
- **Single-barreled** — one concept per question
- **Not leading** — no "Don't you think..." or "Wouldn't it be great if..."

### Option design
- Options must be **MECE** (mutually exclusive, collectively exhaustive)
- Always include "Other (open text)" on multi-select and single-select
- Randomize options when order might bias responses (anchor "Other" and "None" last)
- For multi-select, cap at 3 selections to force prioritization
- For rank order, max 4-5 items (beyond that, respondents satisfice)

### Open text questions
- Max 1-2 per survey
- Frame as forced prioritization: "What's the **single biggest** thing that..."
- Don't use open text for things you could enumerate in a multi-select

### Branching
- Only branch on clear segmentation questions (not every answer)
- Keep total max path under 10 Qs regardless of branches
- Note the full path tree in the spec so reviewers can trace every route
- Don't create branches that produce segments smaller than n=30 (not analyzable)

### Audience targeting
- Define by **behavior** (what they did), not demographics
- Include the trigger event and any qualifying conditions
- Note mutual exclusions with other active surveys
- Hard cap at 1 response per user unless explicitly designing for longitudinal

### Rationale blocks
- Every question in the Notion spec gets a `>` rationale block explaining:
  - Why this question (what hypothesis it tests)
  - Why this question type (why multi-select vs rank vs open text)
  - How it connects to other questions (cross-tab plan)

### Writing voice
- Question text: conversational, second person ("you"), plain language
- Options: short phrases, not sentences
- No jargon, no internal terminology, no product feature names the user wouldn't recognize
- Test readability: would a non-expert user understand every word?
