---
name: interview-notes-synthesis
description: "Synthesize raw UXR interview notes and team debrief comments into a standardized Notion page with participant profile, pain points, strategic recommendations, and a Slack insight drop post. Category: 🔍 User Research"
argument-hint: [Notion meeting notes URL or page ID]
allowed-tools: Read, Bash, Write, Edit, Glob, Grep, WebFetch, WebSearch, AskUserQuestion, mcp__notion__notion-create-pages, mcp__notion__notion-update-page, mcp__notion__notion-fetch, mcp__notion__notion-search, mcp__notion__notion-query-data-sources
---

# Interview Notes Synthesis

Transform raw interview notes (from a Notion meeting recording) and team debrief comments (from the participant's Slack calendar post in #us-team-uxr-updates) into a standardized interview notes page and a Slack insight drop post.

---

## Step 1: Intake — Gather All Source Material

### 1a. Get the source page
The user provides a Notion URL or page ID for the meeting notes page. Fetch it with the `notion-fetch` tool.

### 1b. Extract raw notes content
The source page is typically a Notion meeting recording page. It contains:
- A transcript or recording block at the bottom (this MUST be preserved)
- Raw notes taken during the interview (unstructured text, bullet points, quotes, observations)
- Page properties (Name, Category, Team&PJ, Owner, Status, Date, etc.)

Read and internalize ALL raw content. Pay attention to:
- Direct quotes from the participant (these become key quotes and the "big insight")
- Pain points and frustrations (with specific examples, not general complaints)
- What's working well (positive signals about the platform)
- Feature requests (explicit asks)
- Market/community context (how the participant fits into a broader ecosystem)
- Behavioral data (GMV, order counts, listing counts, frequency, categories)

### 1c. Collect debrief comments from Slack
Before each interview, a calendar post for the participant is shared in **#us-team-uxr-updates**. After the interview, team members who observed or debriefed leave comments under that post with their takeaways, reactions, and observations.

Search Notion (which indexes Slack via connected sources) for the participant's name or user ID in #us-team-uxr-updates to find the calendar post and its thread. Alternatively, ask the user to paste the Slack thread URL or the debrief comments directly.

Debrief comments may include:
- Observer hot takes and first impressions
- Pain points or quotes that stood out to the team
- Connections to existing product work or roadmap items
- Questions raised for follow-up
- Disagreements or alternative interpretations

**Incorporate debrief comments into the synthesis.** They are a first-class input, not an afterthought:
- If a debrief comment highlights a pain point, elevate its priority or add detail
- If a debrief comment connects a finding to a roadmap item, include it in the Action or Strategic Recommendations
- If a debrief comment adds context the interviewer missed, weave it into the relevant section
- If observers disagree on interpretation, note both perspectives

If no debrief comments can be found, ask the user: "I couldn't find debrief comments for this participant in Slack. Want me to proceed without them, or can you paste the thread?"

### 1d. Identify participant profile
From the raw notes and debrief, extract:
- User type: Heavy Buyer, Heavy Seller, Overlapper, Power Seller, etc.
- Collector/seller category (e.g., "Pins Collector", "K-pop Merch", "Fashion Dolls")
- User ID
- Name
- GMV stats (all-time and/or L12M)
- Platform usage (primary vs. secondary platforms)
- Community involvement
- Background and how they found Mercari

If behavioral data is missing from the notes, ask the user if they want you to pull it from Looker or skip.

---

## Step 2: Structure the Notion Page

### 2a. Determine the page title
Format: `[User Type] Category/Niche Participant Name`

Examples:
- `[Heavy Buyer] Pins Collector Sonia Lai`
- `[Heavy Overlapper] Fashion Dolls Angel Sosa`
- `[Power Seller] K-pop Merch Chengyun Li`

### 2b. Preserve existing page properties
Fetch the current page properties and keep ALL of them intact. Update only:
- `Name` → set to the formatted title from 2a
- `Category` → `["UXR Interview Notes"]` (if not already set)
- `Status` → `"New"` (if still in Draft)

Do NOT overwrite: `Team&PJ`, `Owner`, `Created At`, `Date`, or any other existing properties.

### 2c. Build the structured content
The page content MUST follow this exact structure (7 sections). Match the format of the reference page ([Heavy Buyer] Pins Collector Sonia Lai — `3217fa9ffaef802391dac931a5484c67`):

---

**Section 1: Profile Callout**
```
<callout icon="💡" color="gray_bg">
	**[Buyer/Seller/Overlapper] Profile: [Name]**
	- **User ID:** [number]
	- **[Category descriptor]** — [specifics: focus areas, sub-interests]
	- **Collection/Inventory:** [scale description if applicable]
	- **GMV:** $[amount] all-time across [N] orders (~[N] [purchases/sales]/month)
	- **Platform:** [primary platform] (primary), [others] (secondary)
	- **Community:** [groups, channels, social platforms]
	- **[Seller/Buyer secondary role]:** [brief if applicable]
	- **Background:** [how they got into this, how they found Mercari]
</callout>
```

Adapt bullet points to the participant. Not every line applies to every user. Include what's relevant, skip what isn't.

---

**Section 2: Interview Summary**
```
## 📝 Interview Summary
[Single paragraph narrative: who this person is, how they use the platform, what stood out, and what their top ask is. 4-6 sentences. Written in third person, past/present tense.]

**Key Quote:** *"[Most impactful direct quote from the interview]"*
```

Pick the quote that best captures the participant's relationship with the platform or their biggest pain point.

---

**Section 3: Key Pain Points & Findings**
```
## 🔑 Key Pain Points & Findings
### 🔴 High Priority Issues
**1. [Pain point title]**
- [Specific description with examples from the interview]
- [Impact on the user's behavior or workaround they've developed]
- **Action:** [Concrete product recommendation]

**2. [Pain point title]**
- [Details]
- **Action:** [Recommendation]

### 🟡 Medium Priority
**[N]. [Pain point title]**
- [Details]
- **Action:** [Recommendation]

### 🟢 Lower Priority
**[N]. [Pain point title]**
- [Details]
- **Action:** [Recommendation]
```

Rules for pain point classification:
- 🔴 High: Blocks core usage, causes workarounds, or drives users to competitors
- 🟡 Medium: Creates friction but user has a workaround; impacts experience but not retention
- 🟢 Lower: Nice-to-have fixes; aspirational asks; minor annoyances

Number pain points sequentially across all priority levels (1, 2, 3... not restarting at each level).

If a debrief comment elevated or reframed a pain point, reflect that in the priority and include the team's perspective in the description.

---

**Section 4: What's Working Well**
```
## ✅ What's Working Well
- **[Feature/aspect]** — [why it works for this user, with specific example]
- **[Feature/aspect]** — [details]
```

Include 4-8 items. These are signals of what to protect and promote, not just feel-good filler.

---

**Section 5: Market Context**
```
## 📌 [Category] Market Context
- **[Aspect]:** [detail]
- **[Aspect]:** [detail]
```

Section title should name the specific market (e.g., "Enamel Pin Market Context", "K-pop Merch Market Context"). Include:
- Price ranges
- Where this community buys/sells (platforms, social, events)
- Community dynamics (groups, watchlists, ISO culture)
- Creator/brand ecosystem if relevant
- Why Mercari wins or loses in this space

---

**Section 6: Feature Requests**
```
## 💡 Feature Requests
1. [Request]
2. [Request]
```

Simple numbered list. Pull from explicit asks in the interview. If debrief comments surfaced additional feature connections (e.g., "this is related to the bulk edit project Riley is working on"), include a note.

---

**Section 7: Action Items + Strategic Recommendations**
```
## ✅ Action Items
- [ ] [Specific follow-up task]
- [ ] [Another task]

## 📌 Strategic Recommendations
- **[Recommendation title]:** [explanation of why and what to do]
- **[Recommendation title]:** [explanation]
```

Action items are concrete next steps (follow up with X, send gift card, share with Y team).
Strategic recommendations are higher-level product/business takeaways. Incorporate team debrief insights here: if observers connected a finding to a roadmap item or proposed a specific product direction, include it.

---

### 2d. Write to Notion

**CRITICAL: Do NOT use `replace_content`.** It destroys meeting notes blocks (transcript, recording links) permanently.

Use `update_content` with section-by-section `old_str`/`new_str` replacements:

1. First, fetch the current page content to find existing text blocks
2. For **fresh pages** (no formatted sections yet, only meeting notes blocks): look for ANY page-level text blocks outside the meeting notes (empty blocks, stray text, raw notes). Use those as `old_str` anchors in `update_content` to insert all formatted sections. The meeting notes block will remain untouched.
3. For **previously formatted pages** (re-running the skill): match each section heading + content in individual `content_updates` items.
4. Combine all section updates into a single `update_content` call with multiple `content_updates` items.

If truly no page-level anchor exists, alert the user before proceeding.

### 2e. Update page properties
Use `update_properties` to set:
- `Name`: the formatted title
- `Category`: `["UXR Interview Notes"]`
- `Status`: `"New"`

Preserve all other properties.

---

## Step 3: Generate Slack Insight Drop

### 3a. Draft the Slack post
Format the post exactly like this template:

```
✨ Interview Insight Drop — [Category/Topic]

[DYNAMIC opening line — write a fresh, casual 1-liner each time that reflects the actual user type (heavy seller, buyer, overlapper), their specific category, and something specific/surprising from the interview. Never reuse the same phrasing. Make it feel like you're telling a colleague about a conversation you just had.]

**who [she/he/they] [is/are]:**
[1-2 sentences on user background, selling/buying history, scale, and how they operate]

🔑 **the big insight:**
> *"[key quote from the interview]"*

[1-2 sentences expanding on the insight and why it matters for the product]

**top pain points:**

🔴 **#1: [pain point]** — [explanation]. [editorial comment if warranted, e.g., "this is fixable." or "we created this burden."]

🔴 **[pain point]** — [explanation]

🟡 **[pain point]** — [explanation]

🟡 **[pain point]** — [explanation]

💡 **bonus signal:** [any unexpected/strategic finding worth flagging: community dynamics, acquisition channel, behavioral pattern]

full notes here: [Notion URL of the formatted page]
```

Rules:
- Opening line: casual, specific to this interview, never generic. Reference something surprising or concrete.
- Pain points: include 2-3 top ones with emoji priority markers (🔴 high, 🟡 medium). Don't list everything, pick the ones that matter most for the product team.
- Big insight quote: pick the single most impactful quote. It should make someone stop scrolling.
- Bonus signal: something the team wouldn't expect. An acquisition insight, a community behavior, a workaround that reveals a missing feature.
- Tone: casual, direct, conversational. Not a report. Write like you're telling a teammate about a call you just had.
- No double em-dashes (--). Reduce em-dashes generally. Use colons or periods instead.
- Don't use corporate buzzwords: moat, flywheel, leverage, outsized, synergies, unlock, north star, double down, 10x, table stakes, low-hanging fruit, move the needle, wedge, primitives, playbook.
- Don't call collector categories "niche." K-pop, Pokemon TCG, anime, Disney pins are mainstream pop-culture verticals.

### 3b. Present the Slack post to the user
Show the full Slack post draft and ask for feedback. Do NOT send until the user confirms.

---

## Step 4: User Review

### 4a. Present both outputs
Show the user:
1. A link to the formatted Notion page
2. The full Slack post draft

Ask: "Anything you'd like to change in the notes or the Slack post before I send?"

### 4b. Incorporate feedback
If the user requests changes:
- For Notion: use `update_content` with targeted `old_str`/`new_str` replacements
- For Slack: revise the draft and re-present

Repeat until the user confirms both are good.

---

## Step 5: Send Slack Post

### 5a. Send to #us-team-uxr-updates
Once the user confirms, send the Slack post to the `#us-team-uxr-updates` channel.

Use the Bash tool to send via Slack CLI or API:
```bash
# If slack CLI is available:
slack chat send --channel "#us-team-uxr-updates" --text "[message]"

# Or via curl + Slack webhook/API if configured
```

If no Slack integration is available, copy the final post to the clipboard and tell the user to paste it manually.

### 5b. Confirm delivery
Let the user know the message was sent (or provide the final text for manual posting).

---

## Quality Standards

### Content quality
- Every pain point must include a **specific example** from the interview, not a generic complaint
- Every "Action" recommendation must be **concrete and actionable** (not "improve X" but "add Y feature that does Z")
- Quotes must be **actual quotes** from the transcript/notes, not paraphrased or invented
- Market context must reflect **what the participant actually said**, not general knowledge
- Debrief comments must be integrated into the relevant sections, not dumped into a separate "team notes" block

### Behavioral grounding
- Describe what the user **does**, not what they say they'd do
- Include workarounds as evidence of unmet needs
- Reference specific numbers (GMV, order counts, listing counts) when available

### Writing voice
- Plain language, short sentences
- No corporate jargon or AI buzzwords
- Third person for the Notion page, casual first-person-ish for the Slack post
- Em-dashes only when the alternative reads worse; prefer colons and periods

### Preservation
- NEVER destroy meeting notes blocks, transcripts, or recording links
- NEVER use `replace_content` on interview notes pages
- Always verify the meeting notes block survived after writing

### Final checklist
Before presenting to the user, verify:
- [ ] Profile callout has all available data points (User ID, GMV, platform, community)
- [ ] Pain points are priority-ranked with specific examples and actions
- [ ] Debrief comments are integrated into relevant sections (not siloed)
- [ ] At least one direct quote is included
- [ ] Market context section is titled with the specific category
- [ ] Feature requests are pulled from explicit asks (not inferred)
- [ ] Action items include concrete follow-ups
- [ ] Strategic recommendations connect to product decisions
- [ ] Slack post has a unique opening line (not templated)
- [ ] Slack post quote is the most impactful one available
- [ ] Meeting notes / transcript block is intact
- [ ] Page properties are preserved (not overwritten)
