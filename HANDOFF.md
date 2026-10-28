# Popular Alien — Handoff Document

**Project:** Popular Alien (popularalien.com)  
**Owner:** Matt Kenyon  
**Last updated:** 2026-10-27  
**Status:** Text desk is automated. Video and some layout items remain manual.

---

## 1. What this is

Popular Alien is a professional-looking news desk that finds high-engagement claims, checks them against the public record, and publishes short fact-checks that expose missing context, contradictions, and unverifiable assertions. It is independent commentary. It is not affiliated with any broadcaster.

The site is published via Grok Build at `popularalien.grok.me` and served on the custom domain `popularalien.com`.

---

## 2. Live URLs

| Purpose | URL |
|---------|-----|
| Custom domain | https://popularalien.com |
| Grok Build site | https://popularalien.grok.me |
| Data source (raw) | https://raw.githubusercontent.com/mkenyon8/popular-alien/main/episodes.json |
| GitHub repo | https://github.com/mkenyon8/popular-alien |

The site loads its story cards from the raw GitHub URL above. A hard-refresh after a file write shows the new card. No republish is required for content updates.

---

## 3. Domain / DNS

- Domain: `popularalien.com` (registered at GoDaddy, NS: ns73/ns74.domaincontrol.com)
- Custom domain is connected in Grok Publish → Domain tab
- Apex A record points at the IP Grok provides (historically 216.250.1.1 or the Vercel-backed IP shown in the Domain tab)
- www is typically a CNAME to the same target
- GoDaddy forwarding was removed so the URL bar stays on the custom domain with proper SSL
- If the domain stops resolving, re-check the Domain tab in Grok Publish and update the A/CNAME records to the exact values Grok shows

---

## 4. Content architecture

All stories live in a single JSON array at:

```
mkenyon8/popular-alien/episodes.json
```

### Schema (required fields)

```json
{
  "id": "YYYYMMDD-slot",
  "slot": "Morning Check | Midday Check | Evening Check | Late Check",
  "aired_at": "ISO-8601 America/Chicago",
  "claim": "quoted claim",
  "outlet": "outlet name",
  "outlet_url": "https://...",
  "read": "2-4 sentence check",
  "verdict": "Matches record | Missing context | Contradicted | Unverifiable",
  "sources": [{"title": "...", "url": "https://..."}],
  "video_url": "",
  "script": "60-90 second narration"
}
```

- `slot` must be exactly one of the four strings above
- `verdict` must be exactly one of the four strings above
- `video_url` is currently always empty (see Gaps below)
- New episodes are prepended (newest first)
- Existing episodes are kept

The site also has a black "Picture to come" hero slot when `video_url` is empty.

---

## 5. Automation (the unattended desk)

Four Grok Automations run daily in America/Chicago. Each one:

1. Scans the last 12 hours of high-engagement political/media claims
2. Picks one claim where the public record and the headline diverge
3. Writes a 60-90 second script
4. Builds a JSON episode object
5. Calls `github___get_file_contents` on `episodes.json` and notes the SHA
6. Prepends the new episode
7. Calls `github___create_or_update_file` with the new SHA and full array
8. Notifies with `POSTED | Slot | claim`

| Clock | Schedule | Status |
|-------|----------|--------|
| popular-alien-morning-check | 06:30 CT daily | Active — writes repo |
| popular-alien-midday-check | 11:30 CT daily | Active — writes repo |
| popular-alien-evening-check | 17:30 CT daily | Active — writes repo |
| popular-alien-late-check | 23:00 CT daily | Active — writes repo |

**Important:** The GitHub connector must stay connected with repo write permission. Gist write is blocked (403 "Resource not accessible by integration") and is no longer used. If a clock fails, the notification includes the error and the episode JSON so it can be retried.

The clocks do **not** write the published site directly. They write the JSON file. The site reads the file on load.

---

## 6. Layout changes (Build chat only)

Any change to layout, colors, nav, footer, pages, or how cards are rendered must go through the Grok Build chat that published `popularalien`, then **Publish Changes**.

### Pending / recent Build pastes

**Move ReFrame to its own page** (requested, not confirmed applied):

```text
Move the ReFrame embed off the home page and onto its own page at /reframe. Remove the floating lower-right panel and the hero ReFrame block from every other page. On the home page, add a prominent link in the nav next to METHOD that says REFRAME, and also add a clear button under the latest-broadcast slot that says "Open ReFrame" and goes to /reframe. Keep the ReFrame page dark, with the same PROF. REFRAME show and scorecard. Do not change colors, the other nav items, the footer, or the episodes URL.
```

**Add source thumbnails and make the home page more visual** (requested, not confirmed applied):

```text
On the home page, for every story card, fetch the og:image from the episode's outlet_url and show it as a full-bleed thumbnail at the top of the card. Make the thumbnail and the claim both link to the outlet_url so the reader can open the source. If the fetch fails or there is no image, show a dark placeholder with the slot name. Keep the verdict chip, date, and read. Make the cards tighter and more visual: larger type on the claim, the thumbnail full-bleed across the card, and stronger contrast between the black header and the white desk. Do not change the nav, footer, colors of the existing elements, or the episodes URL. Do not regenerate the design.
```

After any Build publish, hard-refresh the site.

---

## 7. Remaining gaps (not yet automated)

| Gap | Why | What it takes |
|-----|-----|---------------|
| **Video clips** | No tool in this setup generates a broadcast video, uploads it, and writes the URL into `video_url`. | Manual clip creation + paste of a public video URL into the episode, or a future video-generation pipeline. |
| **ReFrame page** | Layout change | One Build-chat paste + Publish Changes (see above). |
| **Source thumbnails** | Layout change | One Build-chat paste + Publish Changes (see above). Cards currently have no images. |
| **Favicon / tab title** | Cosmetic | Optional Build paste to set title to "Popular Alien" and a simple favicon. |

The text desk (claim → check → JSON → site) is fully automated for all four slots once the clocks are healthy.

---

## 8. How to operate it day-to-day

- Do nothing for the stories. The four clocks write the file. Hard-refresh to see new cards.
- If a clock fails, the email/notification contains the error and the episode JSON. Fix the permission or SHA conflict and re-run, or paste the JSON into the file manually via GitHub.
- For layout or visual changes, open the Build chat that published the site and paste the instruction. Do not ask the automation to change layout.
- To add a video, put a public URL in the episode's `video_url` field and push the file. The hero slot will use it.
- To check the live data: open the raw URL above or view `episodes.json` in the repo.

---

## 9. Key constraints and lessons

- Gist write is permanently blocked on the current GitHub connector scope. Always use the repo file (`github___create_or_update_file`).
- The site must load from the **unpinned** raw URL (`.../main/episodes.json`), not a SHA-pinned gist URL. A pinned URL freezes the desk.
- Automations cannot change the published site's HTML/CSS. Only the Build chat can.
- Duplicate-guard behavior can occasionally block concurrent writes. If a write fails, read the current SHA and retry.
- Verdicts and slot names must match the enums exactly or the site may not render the chip correctly.

---

## 10. Project memory snapshot

- Site: popularalien.grok.me / popularalien.com
- Data: mkenyon8/popular-alien/episodes.json (raw URL above)
- Morning, Midday, Evening, Late clocks all write the repo file
- Video remains empty
- ReFrame and thumbnail changes are pending Build pastes

---

*End of handoff. Keep this file updated when the architecture changes.*
