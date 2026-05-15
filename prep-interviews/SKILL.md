---
name: prep-interviews
description: "End-to-end interview prep: recruit from Sprig opt-ins or cohort, screen user IDs against Looker data, rank top 20 candidates, generate scheduling email with Calendly link. Category: 🔍 User Research"
argument-hint: <sprig-csv or user-id-list> [--plan <notion-research-plan-url>] [--spec <notion-survey-spec-url>]
allowed-tools: Read, Bash, Write, Edit, Glob, Grep, WebFetch, mcp__notion__notion-create-pages, mcp__notion__notion-update-page, mcp__notion__notion-fetch, mcp__notion__notion-search
---

# Prep Interviews

End-to-end interview preparation workflow: from recruit pool to scheduled interviews.

## Usage

```
# From a Sprig CSV with opt-in responses
/prep-interviews /path/to/survey-export.csv --plan https://notion.so/Research-Plan-ID

# From a manual user ID list
/prep-interviews /path/to/user-ids.csv --plan https://notion.so/Research-Plan-ID

# With a survey spec (to pull cohort/segment definitions)
/prep-interviews /path/to/survey-export.csv --spec https://notion.so/Survey-Spec-ID
```

---

## Step 1: Define the Recruit Pool

### 1a. Determine the source

**If Sprig CSV provided:**
- Parse the CSV (same format as synthesize-survey: row 1 = column IDs, row 2 = labels, rows 3+ = data)
- Find the recruit opt-in question (typically the last Q, with "Yes → email" response pattern)
- Extract all user IDs where the opt-in response = "Yes" or contains an email address
- Also extract their survey responses — these inform candidate ranking later

**If user ID list provided:**
- Read the CSV expecting at minimum a `userId` column
- Any additional columns (segment, cohort, survey responses) are used for ranking

### 1b. Pull research context

**If `--plan` provided:**
- Fetch the Notion research plan page
- Extract: target segments, interview goals, recruiting criteria, category targeting, segment definitions
- This drives the candidate ranking in Step 3

**If `--spec` provided:**
- Fetch the Notion survey spec page
- Extract: audience definition, trigger, segment definitions from cross-tab plan
- Use survey responses to classify candidates into segments

### 1c. Ask the user to confirm
Before querying data, confirm:
- "I found [n] opt-in users from the survey. The research plan targets [segment descriptions]. Does this look right?"
- "Any specific segments you want to prioritize or exclude?"
- "What's the interview format? (duration, incentive amount, study topic for the email)"

---

## Step 2: Screen & Enrich User Profiles

### 2a. Pull behavioral data

For each user ID in the recruit pool, pull from Looker (via the uxr-user-profile script or direct query):

```bash
bun /Users/chiaki/.claude/plugins/cache/mercari-us/superjunior/1.5.1/skills/uxr-user-profile/scripts/user-profile.ts --user-id=[ID]
```

If the script returns a 403, tell the user to run `gcloud auth application-default login` then retry.

**Required data points per user:**

| Field | Source | Purpose |
|---|---|---|
| User status | `users.status` | **MUST be "alive" — reject all others** |
| Admin notes | `users.admin_notes` | Screen for red flags (see 2b) |
| User type | Buy orders > 0 AND sell orders > 0 = Overlapper; buy only = Buyer; sell only = Seller | Segment classification |
| Buy GMV (L12M) | `orders.buyer_id` filter | Activity level |
| Sell GMV (L12M) | `orders.seller_id` filter | Activity level |
| Buy orders (L12M) | Count | Volume |
| Sell orders (L12M) | Count | Volume |
| Buy AOV | GMV / orders | Price tier signal |
| Sell AOV | GMV / orders | Price tier signal |
| Top 3 selling categories | `orders.category_0` grouped | Category alignment |
| Top 2 buying categories | `orders.category_0` grouped | Category alignment |
| Active listings count | `listings` explore | Seller scale |
| Account age | `users.created_at` | Tenure |
| Conversion rate | Sell orders / listings count | Seller efficiency |

### 2b. Red flag screening

**Automatically reject users with ANY of these:**
- User status != "alive" (banned, suspended, deactivated)
- Admin notes containing: "excessive cancellations", "fraud", "suspension", "banned", "policy violation", "chargeback abuse"
- Admin notes containing: "return abuse", "multiple warnings", "restricted"

**Flag but don't auto-reject (note for researcher):**
- High cancellation rate (> 15% of orders cancelled)
- High return rate (> 10% of orders returned)
- Admin notes with any other content (show the note text for manual review)

### 2c. Generate enriched CSV

Output a CSV file with all screened candidates:

```
userId, email, userStatus, userType, buyGmvL12m, sellGmvL12m, buyOrders, sellOrders, buyAov, sellAov, topSellingCat1, topSellingCat2, topSellingCat3, topBuyingCat1, topBuyingCat2, activeListings, accountAgeDays, conversionRate, adminNotes, redFlag, surveySegment, recruitScore
```

**Key columns:**
- `redFlag`: "REJECTED: [reason]" or "FLAGGED: [reason]" or "CLEAR"
- `surveySegment`: which research plan segment they best fit (e.g., "D. Bundle laddering buyer", "C. Overlapper")
- `recruitScore`: ranking score from Step 3

Save as: `[study-name]-recruit-pool-[date].csv`

---

## Step 3: Rank & Recommend Top 20

### 3a. Scoring criteria

Score each CLEAR candidate (skip REJECTED) on a 0-100 scale based on alignment with the research plan:

**Segment fit (0-40 pts)**
- Matches a target segment from the research plan = 40
- Partially matches (e.g., right user type but wrong category) = 20
- Doesn't match any target segment = 0

**Category alignment (0-20 pts)**
- Top categories match the research plan's category targets = 20
- Partial overlap = 10
- No overlap = 0

**Activity level (0-20 pts)**
- High activity (top quartile of GMV/orders in the pool) = 20
- Medium activity = 10
- Low activity = 5
- Inactive (0 orders L12M) = 0

**Diversity bonus (0-10 pts)**
- Adds a user type not yet represented in top 20 = 10
- Adds a category not yet represented = 5

**Survey response signal (0-10 pts)**
- Gave interesting/detailed open-text responses = 10
- Selected responses that indicate unique perspective = 5
- Standard responses = 0

### 3b. Recommend top 20

Sort by recruitScore descending. Present the top 20 as a table:

| Rank | User ID | Type | Segment | Top Categories | L12M GMV | Why this user |
|---|---|---|---|---|---|---|
| 1 | [ID] | Overlapper | D. Bundle laddering | Pokemon, LEGO | $4,200 buy / $6,800 sell | High-GMV overlapper in heritage collectibles, entered via bundles |
| 2 | ... | ... | ... | ... | ... | ... |

The "Why this user" column is a 1-sentence reason tied to the research plan goals.

### 3c. Confirm selection with user
Ask: "Here are the top 20 candidates. Want to adjust the list before I draft the scheduling email?"

Also note:
- How many per segment (e.g., "8 Segment D, 5 Segment C, 4 Segment E, 3 unclassified")
- Any gaps (e.g., "No fashion handbag buyers in the top 20 — want me to pull in lower-ranked candidates from that category?")
- Any flagged users that made the top 20 (show the flag reason)

---

## Step 4: Create Calendly Event

### 4a. Ask for Calendly details

Confirm with the user:
- Study topic (for the event title)
- Duration (30 / 45 / 60 min)
- Incentive amount ($50 / $75 / $100)
- Participant type label (buyers / sellers / buyers and sellers)
- Study goal (1 sentence for the email)
- Date range for availability
- Whether they already have a Calendly event or need to create one

### 4b. Provide Calendly setup instructions

If the user needs to create a new Calendly event:

```
Calendly Event Setup:
- Event name: "Mercari User Interview: [STUDY TOPIC]"
- Duration: [DURATION] minutes
- Location: Google Meet (auto-generate link)
- Description: "A [DURATION]-minute conversation with a Mercari researcher. No prep needed — just show up and share your experience."
- Availability: [DATE RANGE]
- Max invitees: 20
- Buffer: 15 min between interviews
- Confirmation page: Custom message — "You're all set! You'll receive a calendar invite with the Google Meet link shortly."
```

Ask the user for the Calendly URL once created.

---

## Step 5: Draft Scheduling Email

### 5a. Generate the email

Using the confirmed details from Step 4:

```
Subject: Mercari User Interview: [STUDY TOPIC] ([DURATION] Minutes)

Body:

Thank you for letting us know you're interested in participating in a user research session with Mercari!

We're excited to hear from some of our most active [PARTICIPANT TYPE]. Your feedback will directly help us [STUDY GOAL] on Mercari.

What to expect:
- A [DURATION]-minute online video call via Google Meet
- A conversation with a Mercari researcher about how you use the app
- No preparation needed — simply show up and use Mercari the way you normally do
- No right or wrong answers; we want to learn from your genuine experience

Thank-you gift:
You'll receive a [$INCENTIVE AMOUNT] gift card as a thank-you for your time and insights.

Schedule your interview:
Please select a time that works best for you: [CALENDLY LINK]

After you book, you'll receive a confirmation email with the Google Meet link and all necessary details.

Thanks again for being part of the Mercari community — we look forward to speaking with you!
```

### 5b. Output the email + recipient list

1. Print the email subject and body (ready to copy-paste)
2. Print the list of email addresses from the top 20 (comma-separated, ready to paste into BCC)
3. Note any candidates who opted in but didn't provide an email (suggest reaching out via Mercari messaging if available)

---

## Step 6: Summary & Next Steps

Print a summary:

```
Recruit pool: [n] opt-ins from survey
Screened: [n] passed, [n] rejected (red flags), [n] flagged (review needed)
Top 20 selected: [segment breakdown]
Email ready: [subject line]
Calendly: [URL]

Next steps:
1. Send the scheduling email to the top 20
2. If < 5 book within 3 days, send to the next 10 candidates
3. Before each interview, run `/prep-interview-user [user-id]` for the full user profile + custom script guidance
```

---

## Quality Standards

### User safety screening
- **NEVER include banned, suspended, or deactivated users** — check user status first, always
- **NEVER skip admin notes screening** — red flags protect both the researcher and the user
- If Looker access fails, do NOT proceed with unscreened users. Tell the user to fix access first.

### Recruit diversity
- Don't over-index on a single segment — aim for representation across all target segments
- Include at least 1 user from each target category if the pool allows
- Flag if the top 20 is homogeneous (e.g., all overlappers, all high-GMV)

### Privacy
- User IDs and emails are PII — don't write them to Notion pages
- The enriched CSV stays local (not uploaded to Notion or GitHub)
- Don't include admin notes content in any shared output — only note "FLAGGED" or "REJECTED"

### Email quality
- Keep the email warm but professional
- Don't oversell the incentive — lead with "your feedback will help"
- Don't mention specific features or changes being tested
- Don't use the word "interview" in the subject line if doing usability testing (use "feedback session" instead)
