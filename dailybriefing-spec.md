# Skill Specification: dailybriefing

**Version:** 1.0
**Created:** 2026-03-28
**Updated:** 2026-03-28
**Status:** Draft

---

## Overview

Generates and emails Mike Cushing's daily morning briefing — a curated HTML digest covering top general news, AI news, tech news, and local Ventura County stories. This skill codifies the existing cron job that runs daily at 6:30am PT and sends to mbcushing@gmail.com. As a skill it can also be invoked on-demand at any time via `\dailybriefing`.

---

## Trigger

- **Shortcut:** `\dailybriefing`
- **Trigger phrases:** "send my morning briefing", "run my daily briefing", "give me my briefing", "morning briefing"
- **Scheduled:** Daily cron at 6:30am PT (job id: `5e909f25-0454-4e09-96b6-e9b99cdd1c23`) — sends to mbcushing@gmail.com automatically
- **Arguments:** None required. Optional: `--to personal|work` to override recipient (default: personal/mbcushing@gmail.com)

---

## Actions

### GENERATE AND SEND

**When:** Triggered manually or by the 6:30am cron job

**Process:**
1. Use `web_search` to find today's **4–5 top general news** headlines (major US/world news)
2. Use `web_search` to find today's **3–4 AI news** stories (research, product launches, industry, policy)
3. Use `web_search` (query: `site:vcstar.com`) AND `web_fetch` on `https://vcstar.com` for **3–4 local Ventura County** stories
4. Use `web_search` for **2–3 notable tech** stories (excluding AI, which is covered separately)
5. Compose a clean, scannable HTML email
6. Write HTML to `/tmp/morning_briefing.html` using `exec`
7. Send via `gog gmail send -a maikclawing@gmail.com --to mbcushing@gmail.com --subject '...' --body-html "$(cat /tmp/morning_briefing.html)" --no-input`
8. Confirm: "Morning briefing sent to mbcushing@gmail.com ✅"

**Output:** HTML email sent to mbcushing@gmail.com (or work email if specified)

---

## Email Format

```
Subject: ☀️ Morning Briefing – [Day, Month Date, Year]

Sections:
  🌎 Top News        — 4-5 stories, US/world
  🤖 AI              — 3-4 stories
  💻 Tech            — 2-3 stories (no AI overlap)
  📍 Local (Ventura County) — 3-4 stories from vcstar.com

Each story:
  - Bold headline as clickable link
  - 1-2 sentence summary (scannable, not an essay)
  - New line: Source: sitename.com

Footer: Delivered by mAIk · Powered by OpenClaw
```

---

## Inputs & Outputs

| Parameter | Type | Required | Description |
|---|---|---|---|
| `--to` | string | No | Override recipient: `personal` (mbcushing@gmail.com) or `work` (mike.cushing@patagonia.com). Default: personal |

**Output formats:**
- HTML email via gog gmail send
- Chat confirmation message

---

## External Integrations

- **web_search**: News discovery for all four sections
- **web_fetch**: Direct scrape of vcstar.com for local news
- **gog gmail** (`maikclawing@gmail.com`): Email delivery
- **exec**: Writing HTML to `/tmp/morning_briefing.html`

---

## Examples

### Example 1: Manual on-demand trigger

**Input:**
```
\dailybriefing
```

**Output:**
```
Morning briefing sent to mbcushing@gmail.com ✅
```
(HTML email lands in inbox with all four sections)

### Example 2: Send to work email

**Input:**
```
\dailybriefing --to work
```

**Output:**
```
Morning briefing sent to mike.cushing@patagonia.com ✅
```

### Example 3: Scheduled cron run (6:30am PT)

Runs automatically. No user input. Output announced to Discord #reasoning.

---

## Edge Cases & Constraints

- If vcstar.com is unreachable, fall back to `web_search` with `site:vcstar.com OR ventura county news` query
- If gog gmail fails with token error, report the failure with instructions to re-run `gog auth login --account maikclawing@gmail.com`
- HTML must be real rendered HTML — use `--body-html` not `--body-file` (plain text only)
- Write HTML to file first, then pass via `$(cat /tmp/morning_briefing.html)` — do NOT inline multi-KB HTML as a shell argument
- Keep stories scannable — this is a morning briefing, not a newsletter. 1-2 sentences per story max.
- Do not duplicate AI stories in the Tech section

---

## Notes

- This skill documents the existing cron job (id: `5e909f25-0454-4e09-96b6-e9b99cdd1c23`) and makes it invocable on-demand
- The cron job has a 300-second timeout; the `make` action should preserve this in the schedule
- Occasional timeouts occur when web_search returns slow results — this is expected and the cron will retry the next day
- The `--to work` flag is an enhancement over the current cron (which always sends to personal email)
