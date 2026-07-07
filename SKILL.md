---
name: "social-content-planner"
description: "Research-driven monthly social media content calendar planner. Onboards a client, researches trending topics in their niche, runs human approval loops, and delivers a fully scheduled calendar ready for content production."
---

# Social Media Content Planner — SKILL.md

## What This Skill Does

Takes a client from zero to a fully researched, human-approved monthly content calendar. The agent:

1. Runs a structured onboarding questionnaire to understand the client's niche, audience, content types, and goals
2. Builds a Google Spreadsheet with dedicated tabs for research, content planning, and the calendar
3. Researches trending topics in the client's niche using web search + YouTube Data API
4. Runs a two-gate human approval loop (topic angles → content list) before finalizing the calendar
5. Produces a ready-to-use monthly calendar the production team can execute against
6. On the monthly loop, re-runs research and generates a new month without re-onboarding

The spreadsheet is the single source of truth. The human manager approves via conversation — the agent handles all sheet editing.

---

## Prerequisites

Before running this skill, ensure:

- **Google Sheets access** — Google Sheets API v4 via a service account, API key, or Maton connection if using OpenClaw infrastructure
- **YouTube Data API key** — free tier via Google Cloud Console (10,000 units/day). Required for research phase. If unavailable, the agent falls back to web search only.
- **Communication channel** — Slack channel or any messaging surface for the approval loop. For public/standalone use, approval happens in-conversation.

---

## Trigger

Use this skill when asked to:
- Plan a month of social media content for a client or brand
- Build a content calendar
- Research trending topics in a niche for content planning
- Run the monthly content planning loop for an existing client
- Onboard a new client for social content planning

---

## Phase 0 — Client Onboarding

Run this once per new client. If a `Client Config` tab already exists in the client's spreadsheet, skip to Phase 1.

### Step 1: Run the Questionnaire

Ask the client these questions one at a time (or in a grouped batch if they prefer). Be conversational — not a form dump.

---

**Q1 — Brand basics**
> "What's the name of the brand or company we're planning content for?"

**Q2 — Niche and industry**
> "In one or two sentences, describe what this brand does and who it serves. What industry or niche is it in?"

**Q3 — Target audience**
> "Who is the ideal person watching or reading this content? Describe them: age range, profession or lifestyle, what problems they have, what they care about."

**Q4 — Brand voice and tone**
> "How should this brand sound? Pick any that apply, or describe it in your own words:
> - Educational / expert
> - Casual and relatable
> - Motivational / high energy
> - Professional / authoritative
> - Entertaining / personality-driven
> - Something else?"

**Q5 — Content types**
> "What kind of content are we planning? Select all that apply:
> - Short-form video (~60 seconds) — posts to TikTok, Instagram Reels, Facebook Shorts, YouTube Shorts, LinkedIn
> - Long-form video (5+ minutes) — YouTube
> - Blog / articles — website and LinkedIn
> - Image or carousel posts — Instagram, LinkedIn, Facebook
> - Something else?"

**Q6 — Platform confirmation**
> (Auto-populate based on Q5 answers, then confirm)
> "Based on your content types, we'll be planning for: [list platforms]. Does that look right, or do you want to add or remove any platforms?"

**Q7 — Posting frequency**
> "How many pieces of content do you want to publish per month? You can give a total, or break it down by content type (e.g., 8 short videos + 2 articles)."

**Q8 — Competitors and inspiration**
> "Name 3–5 accounts, creators, or brands that are doing content you admire or compete with. They don't have to be in your exact niche — just accounts that set the standard for what good looks like."

**Q9 — Off-limits**
> "Are there any topics, angles, or types of content that are off-limits for this brand? Anything we should never touch?"

**Q10 — Primary goal**
> "What's the main thing you want this content to accomplish? Pick one:
> - Build brand awareness
> - Generate leads or sales
> - Educate the audience
> - Build community and engagement
> - Establish authority in the niche
> - Something else?"

---

### Step 2: Create the Spreadsheet

After the questionnaire is complete, create a new Google Spreadsheet named:
`[Brand Name] — Content Planner`

Create the following tabs:
- `Client Config` — populated immediately with onboarding answers
- `Research [Month YYYY]` — blank, ready for Phase 1
- `Content List [Month YYYY]` — blank, ready for Phase 3
- `Calendar [Month YYYY]` — blank, ready for Phase 5

Share the sheet with edit access (anyone with the link). Report the sheet URL to the human manager.

### Client Config Tab Structure

| Field | Value |
|---|---|
| Brand Name | |
| Niche / Industry | |
| Target Audience | |
| Brand Voice | |
| Content Types | |
| Platforms | |
| Posts Per Month | |
| Posts Per Month (breakdown) | |
| Competitor / Inspiration Accounts | |
| Off-Limits Topics | |
| Primary Goal | |
| Sheet Created | [date] |
| Last Updated | [date] |

---

## Phase 1 — Research

Run each month. Read `Client Config` for niche, target audience, content types, and competitor accounts before starting.

### Step 1: Web Search Battery

Run all of the following search queries, substituting `[niche]` with the client's niche and `[month year]` with the current planning month:

```
[niche] trending topics [month year]
[niche] trending on Twitter [month year]
[niche] what's blowing up on social media right now
[niche] viral content ideas [year]
[niche] top YouTube videos [month year]
[niche] content ideas for [target audience description]
[niche] most shared articles [month year]
[competitor account name] best performing content
```

Add 2–3 more searches if the niche is specific enough to warrant narrower queries (e.g., a local service business might also search `[city] [niche] trending`).

Collect all raw signals. Don't filter yet.

### Step 2: YouTube Data API Research

If a YouTube Data API key is configured, run the following call:

```
GET https://www.googleapis.com/youtube/v3/search
  ?part=snippet
  &q=[niche keywords]
  &type=video
  &order=viewCount
  &publishedAfter=[30 days ago, ISO 8601]
  &maxResults=25
  &key=[YOUTUBE_DATA_API_KEY]
```

Extract: video titles, channel names, view counts (approximate from metadata). Identify recurring topic patterns and angles across the top results.

If no API key is configured, note this and proceed with web search results only.

### Step 3: Synthesize Into Topic Bank

Combine web search signals + YouTube findings into a clean list of *20–30 topic angles*. For each angle, write:

- **Topic angle** — specific framing, not just a broad subject (e.g., "Why most [niche] businesses are leaving money on the table with X" not just "X")
- **Signal source** — what made this show up (trending search, YouTube views, competitor content, etc.)
- **Content type fit** — which content types this angle works best for (short video, article, long-form, etc.)

Write this list to the `Research [Month YYYY]` tab with columns: `#`, `Topic Angle`, `Signal Source`, `Content Type Fit`, `Status` (default: `Pending Approval`).

---

## Phase 2 — Topic Approval Loop

### Step 1: Post Summary to Channel

Post the topic list to the Slack channel (or in-conversation for standalone use) in this format:

```
Here are the [N] topic angles I've researched for [Brand Name] — [Month].

Review the full list in the sheet: [sheet URL]

Here's a quick summary of the top angles:

1. [Topic angle] — [1-line rationale]
2. [Topic angle] — [1-line rationale]
3. [Topic angle] — [1-line rationale]
... (top 10–12 in the message, full list in the sheet)

Let me know:
- Which ones you want to keep, cut, or adjust
- Any topics you'd add that aren't here
- Any angle adjustments needed
```

### Step 2: Revise Based on Feedback

When the human manager responds:
- Mark rejected angles as `Rejected` in the `Status` column
- Update any angles per feedback
- Add any new angles the manager suggests
- Re-post the updated summary if significant changes were made

### Step 3: Confirm Approval

When the human manager explicitly approves (e.g., "looks good," "approved," "let's go with these"), mark all remaining `Pending Approval` angles as `Approved` in the sheet and confirm in the channel:

> "Topic list approved. Moving on to content list generation."

---

## Phase 3 — Content List Generation

Read the approved topic angles from `Research [Month YYYY]`. Read the `Posts Per Month` target and content type breakdown from `Client Config`.

For each content slot in the month, generate a specific post concept:

| Column | Description |
|---|---|
| # | Slot number |
| Working Title | Specific, compelling title — not generic |
| Content Type | Short video / Long video / Article / Image / etc. |
| Platform(s) | Which platforms this publishes to |
| Hook / Angle | One sentence — the specific frame that makes this worth watching/reading |
| Source Topic | Which approved topic angle this comes from |
| Priority | Suggested publish order (1 = first of month) |
| Status | `Draft` by default |

Fill the `Content List [Month YYYY]` tab completely. Every slot in the month should have a row.

---

## Phase 4 — Content List Approval Loop

Same mechanic as Phase 2.

Post to channel:

```
Content list for [Brand Name] — [Month] is ready for review.

Full list in the sheet: [sheet URL]

Here's the lineup:

[Priority 1] [Working Title] — [Content Type] — [Hook/Angle]
[Priority 2] [Working Title] — [Content Type] — [Hook/Angle]
... (first 8–10 in the message, full list in the sheet)

Total: [N] pieces across [content types]. Let me know any changes.
```

Revise the `Content List` tab based on feedback. Repeat until explicit approval.

On approval, confirm:
> "Content list approved. Building the calendar now."

---

## Phase 5 — Calendar Build

Read the approved `Content List [Month YYYY]` tab. Map each piece of content to a specific publish date.

Rules for calendar distribution:
- Spread content evenly across the month — avoid clustering
- For multi-platform same-content (e.g., one short video → TikTok + IG + FB + YT + LinkedIn), this counts as one content piece, one row, one date
- Vary content types across the week where multiple types are in the mix
- Priority 1 = first publish date, not necessarily Day 1 of the month (account for production time — assume at least 3–5 days lead from calendar confirmation to first publish)

### Calendar Tab Structure

| Date | Day | Working Title | Content Type | Platform(s) | Hook / Angle | Priority | Status |
|---|---|---|---|---|---|---|---|
| YYYY-MM-DD | Monday | | | | | 1 | Planned |

Write the full calendar to the `Calendar [Month YYYY]` tab.

Post to channel:
```
Calendar is built for [Month]. Here's the overview: [sheet URL]

First publish: [date] — [title]
Total pieces: [N]
Content mix: [X short videos, Y articles, etc.]

Review the dates and let me know if anything needs shifting. Otherwise this is ready for production.
```

---

## Phase 6 — Handoff to Production

The Google Spreadsheet is the handoff artifact.

When the human manager is ready to begin content production (script writing, article writing, etc.), they start a new session with the relevant production agent and provide:

> "Here's our content calendar for [Brand Name]: [sheet URL]
> Start with Priority 1 and work down. We're producing [content type]."

No additional file needed. The `Calendar` tab has everything the production agent needs.

---

## Phase 7 — Monthly Loop

When the human manager returns for the next month's planning:

1. Read the existing spreadsheet URL from the client record
2. Read `Client Config` — no re-onboarding needed
3. Check previous month's `Calendar` tab to note topics already covered (avoid immediate repeats)
4. Add three new tabs for the new month: `Research [Month YYYY]`, `Content List [Month YYYY]`, `Calendar [Month YYYY]`
5. Run Phases 1–6 for the new month

Each month builds on the previous one. The sheet accumulates a full history of all content planned for the client.

---

## Quick Reference: Spreadsheet Tab Structure

| Tab Name | Created When | Purpose |
|---|---|---|
| `Client Config` | Onboarding | Stores all client configuration — never deleted |
| `Research [Month YYYY]` | Start of each month | Trending topic bank, approval status |
| `Content List [Month YYYY]` | After topic approval | Specific post concepts, titles, hooks |
| `Calendar [Month YYYY]` | After content list approval | Final dated calendar, production status |

---

## Quick Reference: YouTube Data API Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create or select a project
3. Enable "YouTube Data API v3"
4. Create credentials → API Key
5. Add the key as `YOUTUBE_DATA_API_KEY` in your environment or pass it to the agent at session start

Free quota: 10,000 units/day. A typical research run uses ~100–300 units.

---

## Notes for Public Skill Users

- Replace Maton Google Sheets calls with direct Google Sheets API v4 calls using your own credentials
- Approval loop defaults to in-conversation — no Slack required
- YouTube Data API key is optional but strongly recommended for richer research
- The skill is client-agnostic — run it for any brand in any niche
