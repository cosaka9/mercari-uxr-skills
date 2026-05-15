---
name: create-research-plan
description: "Create a UXR research plan from a problem statement. Asks questions to narrow hypotheses, separates what data can answer from what needs UXR, and outputs a structured plan with surveys, interviews, and deliverables. Category: 🔍 User Research"
argument-hint: [problem-statement or Notion URL]
allowed-tools: Read, Bash, Write, Edit, Glob, Grep, WebFetch, WebSearch, mcp__notion__notion-create-pages, mcp__notion__notion-update-page, mcp__notion__notion-fetch, mcp__notion__notion-search
---

# Create UXR Research Plan

Build a structured research plan from a problem statement. The skill separates what can be answered through existing data from what requires primary research, then designs the UXR to fill only the gaps data can't close.

## Philosophy

**Data first, UXR second.** Every hypothesis gets triaged:
- **🟢 Data can answer this** — point to the query, dashboard, or analysis that resolves it
- **🟡 Data gives a partial signal** — describe what data shows and what's still ambiguous
- **🔴 Only UXR can answer this** — design the right method (survey, interview, usability test)

This prevents wasting research cycles on questions a SQL query could answer in 10 minutes.

---

## Step 1: Intake & Problem Framing

### 1a. Get the problem statement
If the user provides a Notion URL, fetch it for context. If they provide a text description, use that.

### 1b. Ask narrowing questions
Before writing anything, ask the user 3-5 questions to sharpen the research scope. Good narrowing questions:

**About the problem:**
- "What decision will this research drive? Who makes that decision and by when?"
- "What do you already know? What data have you already looked at?"
- "What's your current best guess for why this is happening?"

**About the audience:**
- "Who specifically are we studying? Can you describe the behavioral segment, not a demographic?"
- "How would you find these people in your data? What query or filter defines them?"

**About constraints:**
- "What's the timeline? When does the decision need to be made?"
- "What methods are available? (Sprig surveys, interviews, usability tests, etc.)"
- "Are there any known constraints? (e.g., don't include BNPL, don't study X segment)"

**About existing evidence:**
- "Has anyone studied this before? Where are the prior findings?"
- "What related data exists? (dashboards, prior surveys, behavioral logs)"

Do NOT proceed until you have clear answers to at least: (1) the decision being made, (2) the audience, and (3) the timeline.

### 1c. Frame the problem statement
Write a 2-3 sentence problem statement that includes:
- What's happening (the observed gap or behavior)
- Why it matters (revenue, retention, strategy)
- What we don't know (the specific unknowns this research will resolve)

Confirm the problem statement with the user before proceeding.

---

## Step 2: Generate & Triage Hypotheses

### 2a. Generate hypotheses
From the problem statement and context, generate 4-8 testable hypotheses. Each hypothesis should be:
- **Specific** — names a mechanism, not just a correlation
- **Falsifiable** — you can describe what data or evidence would disprove it
- **Actionable** — if true, it points to a specific product action

Format:
> **H1: [Hypothesis statement]**
> If true → [product action]
> If false → [what changes]

### 2b. Triage each hypothesis: data vs. UXR

For each hypothesis, determine:

**🟢 Data can answer this** — The hypothesis can be confirmed or rejected with existing behavioral data, logs, or metrics.
- Name the specific data source, query, or analysis
- Example: "Pull L12M orders where seller_id has listings on both Mercari and eBay (via cross-platform ID match) — if >60% of $200+ items are listed on eBay first, H1 is supported"

**🟡 Data gives a partial signal** — Data can show *what* is happening but not *why*.
- Describe what data reveals and what remains ambiguous
- Example: "We can see that 8,300 buyers/month make 2+ non-bundle purchases from the same seller — but data can't tell us if the barrier is awareness, friction, timing, or risk perception"

**🔴 Only UXR can answer this** — The hypothesis involves motivations, perceptions, decision-making, or experiences that aren't captured in behavioral data.
- Name the right UXR method and why
- Example: "Why sellers choose eBay over Mercari for $500+ items requires in-context interviews — the decision factors (trust, buyer pool, fees, habit) aren't in our data"

Present this triage to the user. Ask: "Does this split look right? Any hypotheses you'd move between categories?"

### 2c. Data validation plan
For all 🟢 and 🟡 hypotheses, write a concrete data validation plan:
- What to query (table, filters, dimensions)
- What result would confirm or reject the hypothesis
- Who should run it and by when

This becomes a pre-research checklist. The UXR plan only covers what survives data triage.

---

## Step 3: Design the Research Plan

### 3a. Determine methods
Map each 🔴 (and remaining 🟡) hypothesis to the right method:

| Signal needed | Method | When to use |
|---|---|---|
| Prevalence / distribution | **Survey** (Sprig) | "How many?", "Which is most common?", ranking priorities |
| Motivation / decision process | **Interview** (1:1, 45 min) | "Why?", "How did you decide?", journey reconstruction |
| Usability / friction | **Usability test** | "Can they do X?", "Where do they get stuck?" |
| Competitive landscape | **Competitor analysis** | "How do others solve this?", positioning gaps |

### 3b. Structure the plan
Organize into tracks if the research covers multiple domains. Each track should have:

**For surveys:**
- Audience definition (behavioral, not demographic)
- Trigger event (when in the user journey)
- Sample target and field window
- Key questions mapped to hypotheses
- Design constraints
- Link to survey spec (or note that one needs to be created)

**For interviews:**
- Segment definitions with recruiting criteria (behavioral filters)
- Number of interviews per segment and why
- Category or cohort targeting
- Key probes mapped to hypotheses
- What behavioral data to pull on each participant before the interview
- Discussion guide (or note that one needs to be created)

**For each method, include:**
- Which hypotheses it addresses
- What a "clear signal" looks like (what result = actionable finding)
- What cross-tabs or analysis cuts to plan for

### 3c. Sequencing
Order the research so earlier studies inform later ones:
- Surveys before interviews (quantify first, then probe the "why")
- Data validation before any UXR (don't research what data already answers)
- Avoid audience overlap between concurrent surveys

---

## Step 4: Define Decisions & Deliverables

### 4a. Decisions table
Create a table mapping each decision to the research that answers it:

| Decision | Track | How we answer it |
|---|---|---|
| [Specific decision] | [Track name] | [Method + specific data point] |

Every row should pass the test: "If we get this answer, what will the PM/designer do differently next week?"

### 4b. Deliverables
List concrete outputs with owners and timing:
- Survey reports
- Interview insight drops (Slack)
- Integrated share-outs
- Recommendation documents
- Follow-up research plans

### 4c. Out of scope
Explicitly list what this study does NOT cover and why. This prevents scope creep and sets expectations.

### 4d. What follows
Describe what comes after this research wave — future studies, longitudinal tracking, benchmarking additions.

---

## Step 5: Output

### If creating a Notion page:
Create in US Documents database: `collection://30a7fa9f-faef-80e5-9ee7-000b6f11ae2b`

**Required properties:**
- `Team&PJ`: `["https://www.notion.so/2b07fa9ffaef807cbe3bfe33bdd8fdc9"]` (US UXR)
- `Category`: `["Research"]`
- `Status`: `"Draft"`

**Page structure:**

```
## [Method type] · Owner: [name]

---

## Problem & Hypotheses {color="gray_bg"}

**Problem:** [2-3 sentence problem statement]

**Hypotheses:**

1. **[H1 statement]** *(Method: [survey/interview/data])*

   🟢 DATA CHECK: [what to query] → [what confirms/rejects]

2. **[H2 statement]** *(Method: [survey/interview/data])*

   🟡 PARTIAL: [what data shows] → UXR needed for: [specific gap]

3. **[H3 statement]** *(Method: [survey/interview])*

   🔴 UXR ONLY: [why data can't answer this]

...

---

## Decisions this study drives {color="gray_bg"}

[Decision table]

---

## Context & Background {color="gray_bg"}

<details>
<summary>Full context (click to expand)</summary>
  [Background, prior research, what changed, references]
</details>

---

## Data Validation Checklist {color="gray_bg"}

<table header-row="true" fit-page-width="true">
  [Hypothesis | Data source | Query/analysis | Expected result | Owner | Status]
</table>

---

## Track 1: [Name] {color="gray_bg"}

### 📊 Survey — [Title]
[Survey details table + design constraints]

### 🎤 Interviews — [Title]
[Segment table + recruiting criteria + probes]

---

## Track 2: [Name] {color="gray_bg"}
...

---

## Out of scope {color="gray_bg"}
[What's excluded and why]

---

## Deliverables {color="gray_bg"}
[Deliverables list by track]

---

## What follows {color="gray_bg"}
[Next wave, longitudinal tracking, benchmarking additions]
```

### If outputting as markdown:
Write to a local file and print the path. Same structure as above but in standard markdown.

---

## Quality Standards

These standards apply to ALL research plans produced by this skill.

### Question quality
Every research question must be:
- **Grounded in behavior** — asks about what people *did*, not what they *would do*
- **Decision-driving** — the answer changes what the team builds or prioritizes
- **Not answerable by data alone** — if a query could answer it, flag it as 🟢 and move it to the data checklist

### Red flags to catch
If any of these appear in the plan, rewrite them:
- "Would you use this?" → replace with "Tell me about the last time you needed to do X"
- "Is this a good idea?" → replace with "How do you currently solve this?"
- "What features do you want?" → replace with "What's hard about how you do this now?"
- "Do you usually experience this?" → replace with "Walk me through the last time this happened"
- Hypothetical scenarios → replace with past/present behavior probes
- Opinion questions → replace with decision-process questions

### Hypothesis quality
- Each hypothesis names a **mechanism** (not just "users don't like X" but "users route $200+ items to eBay because Mercari lacks visible buyer protection at checkout")
- Each hypothesis has a **falsification condition** (what would disprove it)
- Each hypothesis maps to a **product action** (if true, we build X; if false, we pivot to Y)

### Method selection
- Don't default to interviews for everything — surveys are faster and more scalable for prevalence questions
- Don't use surveys for "why" questions — they'll get post-hoc rationalization, not real motivations
- Match the method to the signal: how many → survey, why → interview, can they → usability test
- Sequence surveys before interviews when both are needed (quantify, then probe)

### Sample and audience
- Define audiences by **behavior** (what they did), not demographics (who they are)
- Include the specific query or filter that identifies the audience in your data
- Note minimum sample sizes: surveys 200+, interviews 5-6 per segment, usability tests 5-8
- Flag when segments overlap between concurrent studies and stagger field windows

### Writing voice
- Plain language. No jargon.
- Short sentences. Concrete mechanisms.
- Hypotheses should read like claims you'd bet on, not corporate hedging.
- "Sellers route $200+ to eBay because..." not "It is hypothesized that sellers may prefer..."

### Competitive claims
- If the plan references competitor features or pricing, verify with a web search first
- Don't assume competitor capabilities — check current state

### Final checklist
Before presenting the plan, verify:
- [ ] Every hypothesis is triaged as 🟢 / 🟡 / 🔴
- [ ] Data validation checklist is concrete (table, filter, expected result)
- [ ] Every UXR question is grounded in behavior, not opinions
- [ ] Every method maps to a specific decision
- [ ] Audiences are defined by behavior with query-able filters
- [ ] Timeline accounts for sequencing dependencies
- [ ] Out of scope is explicit
- [ ] Deliverables have owners and timing
