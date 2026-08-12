---
name: branded-client-report
description: "Create beautiful, branded performance reports from Perspective funnel data. Use this skill whenever the user asks for a client report, monthly report, performance report, funnel report, stakeholder update, or wants to show funnel results to a client, boss, or team. Trigger on: 'create the report for [client]', 'monthly report', 'client update', 'show [client] how their funnels performed', 'I need to report these numbers'. Requires the Perspective MCP connection. Outputs a slide-style HTML presentation, delivered as a web link plus a ready-made PDF, branded for the recipient."
---

You create performance reports the user can send as-is. The bar: the recipient reads it in 3 minutes, understands what happened, and feels the sender has things under control.

The typical user is an agency reporting to a client, but the same flow works for anyone reporting to a boss, a team, or themselves. When there's no client, the "client brand" is simply the user's own brand, so don't assume a client exists; just ask who the report is for if it isn't obvious.

## When the skill starts

If the user activated this skill without naming a client, funnel, or timeframe, open with a short intro so they know what they just unlocked: two or three sentences on what this skill does (a branded, send-ready performance report built from their real funnel data, as a slide-deck presentation delivered as a web link plus a ready-made PDF) and why it matters (the report reads like an hour of work and takes one prompt). Then immediately offer the first step: "Who's the report for, and should I build last month? The branding comes straight from their funnel, so no logo hunting needed."

Keep the intro under 100 words. If the user already named a client or timeframe, skip the intro and scope.

## Before you start

Check that the Perspective MCP is connected (look for Perspective tools among your available tools). The report stands on real numbers, so without the connection, don't mock anything up. Point the user to the setup (about 2 minutes): [How to connect the Perspective MCP with Claude](https://intercom.help/perspective-funnels/en/articles/15374243-how-to-connect-perspective-mcp-with-claude), and stop there.

Know the MCP's limits before you plan the run, so you don't burn time probing for things it cannot give you:

- It does NOT expose the funnel's on-page design, copy, or its public URL. Branding and screenshots always come from the live page or the client's website (see the URL playbook below).
- `get_kpi_metrics` (sessions, new_contacts, conversion_rate), `get_chart_metrics` (page_to_page_conversion_rate, contacts_over_time, top_utm_sources), `list_contacts`, and `list_domains` cover everything this skill needs.
- `chart_top_utm_sources` sometimes returns "not found" even when contacts clearly carry UTMs; verify against `list_contacts` before claiming "no tracking".

## Step 1: Scope, with as few questions as possible

Infer what you can, then ask for the rest in ONE message:

- **Which funnels:** If the user names a client or pastes an app.perspective.co editor URL (the 24-hex funnel id is in the path), resolve it via `list_workspaces` (returns every workspace with funnel ids and names) and confirm the list in passing rather than asking open-endedly.
- **Timeframe:** Default to the last full calendar month vs. the month before. Only ask if the user hinted at something else.
- **Branding:** Best source is the live funnel (already branded for exactly this client), second best the client's website. In the same scope message, also ask whether the user wants to add an extra design reference (a website, a brand guide, a deck they like); phrase it as optional ("I'll pull the branding from the funnel; want me to use another site as an additional design reference?"). If they name one, blend it in: the funnel stays the source of truth for logo and colors, the reference informs layout, typography feel, and imagery style. Extract with `getComputedStyle` in the browser: heading + body font families, button colors and radius, logo `<img>` URL, and absorb the copy tone while you're there. For monochrome brands, pick ONE accent color from their photography (e.g. the ocean teal in a beach shot) and use it consistently; don't fall back to generic green/red for deltas. Never invent or approximate a logo. After pulling the branding, confirm it in one line ("Using the logo, the blue, and the font style from the [funnel name] design") so the user can correct it before you build. That sentence is also a small wow: they never told you any of it.
- **Format:** There is exactly one format, the presentation, so never ask about format. It is always delivered twice: as a web link (slides in the browser) AND as a ready-made PDF you generate yourself, which covers the "I want a document to attach" case.

### Finding the live funnel URL (do this in order, don't guess)

1. `list_domains` gives the client's custom domain (e.g. funnel.client.com). Published funnels live at `https://<domain>/<funnel-slug>/`.
2. The funnel slug is NOT available via the API. `list_contacts` meta only has `ps_start_slug`, which is the first PAGE's slug, not the funnel slug. Slug guessing wastes time: curl gets Cloudflare-403'd and the browser 404s on wrong slugs.
3. Check the client's website for links to the funnel domain (read all `<a href>`s).
4. If that fails, ask the user for the publish link. One question beats fifteen minutes of probing, and they always have it.

Inside the funnel, page slugs are discoverable by clicking through in the browser (each step changes `location.pathname`); useful when you need specific pages for screenshots.

## Step 2: Pull the data

Per funnel, for the report window and the comparison window (fire these in parallel, both windows at once):

- `get_kpi_metrics`: kpi_total_sessions, kpi_new_contacts, kpi_conversion_rate
- `get_chart_metrics` page_to_page_conversion_rate: per-step visitor counts for the funnel-concept view and the biggest drop-off
- `get_chart_metrics` contacts_over_time across BOTH windows in one call: reveals traffic stops, spikes, and when leads actually arrived; this is often the real story
- `get_chart_metrics` top_utm_sources: channel mix (verify odd results against `list_contacts`)

If a funnel has no data for the window, show it as "no data" with a one-line reason. Every number in the report comes from the pulled funnel metrics, always. Never invent, estimate, or "improve" numbers, and never offer to; one invented number poisons trust in the whole report.

## Step 3: Write the narrative

Numbers alone are not a report. Structure the story:

1. **Headline result:** the one number the recipient cares about (usually leads; CPL if ad data is connected)
2. **What drove it:** 2-3 sentences, plain language
3. **Per-funnel breakdown:** table with sessions, leads, conversion rate, change vs. previous period
4. **What we're doing next:** 2-3 concrete actions. This section is what makes the sender look proactive, so it needs real actions, not filler.

Writing rules, and why:

- Plain language a non-marketer understands: "more visitors became leads" beats "CVR improved 80bps". The recipient is often not a marketer.
- Never hide a bad month. State it, explain it, and lead with the action plan. Recipients forgive bad numbers; they don't forgive surprises.
- Every percentage gets an absolute number next to it ("+23% (312 vs. 254 leads)"), because percentages without absolutes read as spin.
- Remember who is sending this: the user (usually an agency) BUILT the funnel they are reporting on, and the recipient is their client. The report is implicitly a review of the sender's own work, so never frame funnel weaknesses as flaws or mistakes in how it was built. Frame them as levers the sender is actively working: "biggest opportunity", "this step is our next test", "the June rework already lifted this from X to Y". Stay honest about the numbers themselves, but the explanation always leads to the action plan, never to blame, and next steps read as a confident plan, not a repair list.
- No em-dashes anywhere in the output.

## Step 4: Build the output

The output is a single-file HTML presentation, truly self-contained: embed the logo, all images, AND the brand fonts as data URIs. Fonts: fetch the Google Fonts css2 URL for the needed families/weights, download the latin woff2 files, base64 them into `@font-face` rules (variable fonts cover several weights with one file; declare `font-weight: 400 700` ranges instead of duplicating). Self-contained means the file works offline, in side panels, AND as a hosted artifact (whose CSP blocks all external hosts).

**Read `reference/example-deck.html` before you write any markup.** It is a real, approved deck from a past run with the images and fonts stripped out, so it is small enough to read in full. Use it for the structural and CSS patterns that took several rounds of feedback to get right: slide scaffolding and scroll-snap, the entrance-animation observer with per-element `transitionDelay`, the count-up, the self-drawing SVG chart, the funnel step timeline with drop-off chips, the screenshot frames, the print rules. Copy the mechanics, never the content or the styling: colors, fonts, imagery, wording and slide count come from this client's brand and this client's data. A deck that looks like the example instead of like the client has failed.

A scroll-snap, slide-style HTML deck (6-9 slides), presentable live in a call and exportable to PDF:

- Title slide: logo, "Performance Report [Month]"; if the client has a strong brand photo, use it full-bleed with a dark overlay
- Headline stat slide: huge number with a count-up animation, short label, change vs. last period
- One simple animated chart where it earns its place, built strictly from the pulled data: aggregate `chart_contacts_over_time` daily values into weeks for a leads-per-week line (self-drawing SVG via stroke-dashoffset), and use the `page_to_page_conversion_rate` counts for the funnel-step view. Chart values must re-sum to the reported totals. Highlight the record data point in the accent color; skip the chart entirely if the window has too little data to show a shape
- Related stats share ONE slide as two invisible columns (e.g. traffic + conversion rate) instead of one thin slide each
- A "funnel concept" slide: vertical step timeline with per-step visitor counts and drop-off chips ("−59% exit here"), the biggest exit highlighted, and 1-2 funnel screenshots beside it. Do NOT make a separate "what the funnel looks like" slide; the client knows their own funnel, the screenshots are supporting texture, not the message.
- "What's next" and a closing slide (mirror the title treatment)
- Smooth fade/slide-up entrance animations via IntersectionObserver: use a threshold around 0.35 (0.5 never fires when a slide is taller than the viewport) and reset the animation classes when a slide leaves the viewport so entrances replay on every visit (essential when presenting). Stagger sequential content like funnel steps at ~450ms per item (130ms staggers blur into one fade), and implement the stagger via `el.style.transitionDelay` set per element, NOT via setTimeout chains (throttled tabs batch timeouts and break the order). Cascade trap: never write an ancestor rule like `.in .fade { opacity: 1 }`; the moment the slide gets its class, every child appears at once and silently defeats all per-element staggering. Trigger visibility only through the element's own class (`.fade.in`). Verify the sequence by sampling computed opacity over time, not by checking class names. Arrow-key navigation, slide indicator bottom right
- Big bold typography (clamp-based sizes), one idea per slide, dark and light slides alternating for rhythm
- Small brand motifs beat big effects: a glyph from the client's logo as kicker prefix (some logos hide one, like an "=" or an accent mark), a quiet editorial running footer on each slide ("CLIENT · PERFORMANCE REPORT · MONTH"). Skip ghost/outline background typography; tested and rejected as noise.
- **Print-ready CSS (for the PDF you generate):** no export button in the deck; user-triggered printing is unreliable (hosted artifact viewers sandbox away `window.print()`, and browser print dialogs default to portrait, which wrecks the slide layout). Instead YOU generate the PDF (see Step 5) and the deck just has to print correctly: scroll-snap off, one slide per page (`page-break-after: always`, landscape A4 via `@page { size: A4 landscape }`), every animation forced to its final state (`opacity: 1 !important`, no transforms, charts fully drawn), dark slides keep their backgrounds (`print-color-adjust: exact` on the slide containers), indicator hidden. Re-assert multi-column grid layouts with `!important` inside `@media print`; mobile `max-width` media queries can falsely match during print layout and silently collapse columns or hide content. Also wire `window.addEventListener('beforeprint', ...)` to force every slide to its final animated state and set count-up numbers to their targets, so a manual Cmd+P also produces complete pages.

**Funnel screenshots (mobile mockups):** Show real pages, never rebuilt approximations and never live iframes (iframes render blank when the file is opened locally or in side panels). Cloudflare blocks curl AND headless Chrome on funnel domains, so capture via a real Chrome window:

```bash
open -na "Google Chrome" --args --app="<funnel-page-url>" --window-position=0,25 --window-size=390,880 --user-data-dir=/tmp/shot-profile
sleep 9
screencapture -x -R0,25,390,880 raw.png
sips --cropOffset 72 0 -c 1460 748 raw.png   # trims title bar (top 72px @2x) and scrollbar (right)
sips -s format jpeg -s formatOptions 72 raw.png --out shot.jpg
```

Frame them as clean rounded screenshots: aspect ratio ~9/17.5, border-radius ~22px, hairline border, soft shadow, `object-fit: cover; object-position: top`. NO CSS phone bezels or notches; they produce edge artifacts and cover the funnel's own header.

If the brand color is unreadable as text (yellow, neon), use it for backgrounds and accents and keep text dark. Readability outranks brand fidelity.

## Step 5: Quality check, then deliver

Verify visually before delivering: serve the file over localhost (a 10-line node http server; python http.server may be sandbox-blocked, and the preview pane suppresses some content in file:// mode), scroll each slide, and screenshot after the entrance animations have settled.

Checklist:

- Logo renders (embedded, not a placeholder)
- Every number matches the pulled data exactly, and derived numbers are consistent with each other
- Every change indicator points the right way
- No Perspective branding anywhere (this is the user's report, not ours)
- The generated PDF is correct: landscape, one slide per page, dark slides keep their backgrounds, all numbers and charts in final state, nothing clipped, no indicator visible. Read it page by page before delivering
- "Next steps" contains real actions and matches what the data slides highlight (if one slide says step X is the biggest leak, the next steps must talk about step X)

Deliver four things, always all of them:

1. The HTML file itself.
2. A stable web link: publish as an Artifact (strip the doctype/html/head/body wrapper first; keep title, style, content, script) and re-publish the same file path on every revision so the URL never changes. Always state the link in your reply.
3. A ready-made PDF, generated headlessly from the served deck: `chrome --headless --print-to-pdf="<Client>-Performance-Report-<Month>.pdf" --no-pdf-header-footer <localhost-url>` (the @page landscape rule makes it come out right; never rely on the user's print dialog, which defaults to portrait). Regenerate it after every deck change.
4. A 2-sentence summary the user can paste into the email they send with it.

Assume the user may not be deeply familiar with AI tools or HTML files, so close with a short, matter-of-fact note on how to get the report to their client; helpful, not lecturing. Two routes, one line each: "Send your client the link, they can open it in any browser. Or attach the PDF to your email if you'd rather send a file." 

Then offer the natural next step: "Want me to set this up as a recurring monthly report?" That one sentence turns a one-off into a habit.
