# UXR Survey Skills for Claude Code

Shared Claude Code skills for analyzing Sprig survey exports. Install once, run from any project.

## Skills

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

## Install

### Option A: Copy the skill folder (simplest)

```bash
# Clone the repo
git clone https://github.com/cosaka9/uxr-survey-skills.git /tmp/uxr-survey-skills

# Copy the skill into your Claude Code skills directory
cp -r /tmp/uxr-survey-skills/synthesize-survey ~/.claude/skills/synthesize-survey

# Clean up
rm -rf /tmp/uxr-survey-skills
```

### Option B: Symlink (auto-updates when you pull)

```bash
# Clone to a permanent location
git clone https://github.com/cosaka9/uxr-survey-skills.git ~/uxr-survey-skills

# Symlink into Claude Code
ln -s ~/uxr-survey-skills/synthesize-survey ~/.claude/skills/synthesize-survey
```

Then pull to update:
```bash
cd ~/uxr-survey-skills && git pull
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
