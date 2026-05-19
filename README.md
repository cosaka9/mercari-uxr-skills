# UXR Skills for Claude Code

Shared Claude Code skills for user research. Install once, run from any project.

## Skills

### `/create-research-plan`
Builds a UXR research plan from a problem statement. Asks narrowing questions, generates hypotheses, and triages each one as data-answerable or UXR-required before designing any surveys or interviews.

**Key feature:** Every hypothesis gets a traffic light:
- **Data can answer this** — skip the research, run the query
- **Data gives a partial signal** — data shows *what*, UXR explains *why*
- **Only UXR can answer this** — design the right method

**Usage:**
```bash
/create-research-plan "Why do sellers list $200+ items on eBay instead of Mercari?"
/create-research-plan https://notion.so/Your-Problem-Brief-Page-ID
```

---

### `/create-survey`
Two-phase workflow for building Sprig surveys:
1. **Problem → Notion spec:** Asks narrowing questions, triages hypotheses (data vs survey), designs questions with rationale blocks, creates a Notion survey spec page for review
2. **Notion spec → Sprig file:** After review/feedback, converts the spec into a Sprig-compatible upload file

**Usage:**
```bash
# Phase 1: Design the survey
/create-survey "Why don't repeat same-seller buyers bundle?"

# Phase 2: Generate Sprig upload file from reviewed spec
/create-survey --sprig https://notion.so/Survey-Spec-Page-ID
```

---

### `/prep-interviews`
End-to-end interview preparation: recruit from Sprig opt-ins, screen user IDs against Looker (status, admin notes, red flags), enrich with behavioral data, rank top 20 candidates by research plan alignment, generate Calendly setup + scheduling email.

**Usage:**
```bash
# From Sprig survey with opt-ins
/prep-interviews /path/to/survey-export.csv --plan https://notion.so/Research-Plan-ID

# From a manual user ID list
/prep-interviews /path/to/user-ids.csv --plan https://notion.so/Research-Plan-ID
```

**Workflow:**
1. Extract opt-in user IDs from Sprig CSV
2. Screen each against Looker: status must be "alive", no red flags in admin notes
3. Enrich with L12M behavioral data (GMV, orders, categories, AOV, tenure)
4. Score & rank top 20 by alignment with research plan segments
5. Output enriched CSV + Calendly setup instructions + ready-to-send scheduling email

---

### `/synthesize-survey`
Analyzes a Sprig CSV export and produces a structured Notion report with statistical analysis, cross-tabs, open-text themes, and actionable findings.

**Works for:**
- Recurring benchmarking surveys (QoQ tracking, pillar scorecards)
- One-off targeted surveys (hypothesis-driven, spec-linked cross-tabs)

**Usage:**
```bash
# Basic — just the CSV
/synthesize-survey /path/to/survey.csv

# With a prior quarter for QoQ comparison
/synthesize-survey /path/to/survey.csv --baseline /path/to/prior.csv

# With a Notion survey spec (pulls hypothesis + cross-tab plan)
/synthesize-survey /path/to/survey.csv --spec https://notion.so/Survey-Spec-Page-ID

# With Looker behavioral data for cohort analysis
/synthesize-survey /path/to/survey.csv --behavior /path/to/looker.csv

# All options
/synthesize-survey /path/to/survey.csv --baseline /path/to/prior.csv --behavior /path/to/looker.csv --spec https://notion.so/Survey-Spec-Page-ID
```

### `/interview-notes-synthesis`
Synthesizes raw UXR interview notes (from a Notion meeting recording) and team debrief comments (from the participant's Slack calendar post in #us-team-uxr-updates) into a standardized interview page and Slack insight drop post.

**Outputs:**
1. **Notion page:** Participant profile callout, interview summary with key quote, pain points ranked by priority (🔴/🟡/🟢), what's working well, market context, feature requests, action items, and strategic recommendations
2. **Slack post:** "Interview Insight Drop" with a casual opening line, key quote, top pain points, and a bonus signal

**Usage:**
```bash
# From a Notion meeting notes page
/interview-notes-synthesis https://notion.so/Meeting-Notes-Page-ID

# From a page ID
/interview-notes-synthesis 34b7fa9ffaef80369c4fe21acd9df9aa
```

**Workflow:**
1. Fetch raw meeting notes from Notion
2. Collect team debrief comments from the participant's calendar post in #us-team-uxr-updates
3. Extract participant profile, pain points, quotes, and market context
4. Synthesize into standardized Notion page format (preserving meeting notes/transcript blocks)
5. Generate Slack insight drop post
6. Present both for user review and feedback
7. Send confirmed Slack post to #us-team-uxr-updates

---

## Install

### Option A: Copy the skill folder (simplest)

```bash
# Clone the repo
git clone https://github.com/cosaka9/mercari-uxr-skills.git /tmp/mercari-uxr-skills

# Copy the skill into your Claude Code skills directory
cp -r /tmp/mercari-uxr-skills/synthesize-survey ~/.claude/skills/synthesize-survey

# Clean up
rm -rf /tmp/mercari-uxr-skills
```

### Option B: Symlink (auto-updates when you pull)

```bash
# Clone to a permanent location
git clone https://github.com/cosaka9/mercari-uxr-skills.git ~/mercari-uxr-skills

# Symlink into Claude Code
ln -s ~/mercari-uxr-skills/synthesize-survey ~/.claude/skills/synthesize-survey
```

Then pull to update:
```bash
cd ~/mercari-uxr-skills && git pull
```

### Verify installation

Open Claude Code and type `/synthesize-survey` — it should appear in the skills list.

## Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed
- Notion MCP server connected (for Notion page output)
- Python 3 available in PATH (for CSV analysis)

## Behavioral data CSV format

If using `--behavior`, the Looker export CSV should have these columns:

| Column | Description |
|---|---|
| `userId` | Mercari user ID (matches Sprig `userId` column) |
| `buyGmv` | L12M buy-side GMV |
| `sellGmv` | L12M sell-side GMV |
| `buyOrders` | L12M buy order count |
| `sellOrders` | L12M sell order count |
| `buyAov` | L12M buy-side AOV |
| `sellAov` | L12M sell-side AOV |
| `topSellingCategory` | Top selling category by GMV |
| `topBuyingCategory` | Top buying category by GMV |
| `accountAgeDays` | Account age in days |

Additional columns are welcome — the skill will use whatever is available.
