# Branded Client Report — a Claude skill

Turn **Perspective** funnel data into a branded, send-ready performance report with one prompt. Claude pulls the real numbers, writes the story a client reads in 3 minutes, and designs a slide deck in the client's own branding — delivered as a web link plus a finished PDF.

Built for [Perspective](https://www.perspective.co), the AI funnel platform.

## What it does

1. **Scopes in one question round.** Names a client or a funnel and it resolves the rest itself: the funnels via the Perspective MCP, the timeframe (defaults to last full month vs. the month before), and the branding. It only asks for what it genuinely can't infer, in a single message.
2. **Pulls the data live** through the Perspective MCP: sessions, leads, conversion rate, step-by-step drop-off, leads over time, and traffic sources — for the report window and the comparison window, across every funnel the client runs (also as one combined report for multiple funnels).
3. **Extracts the branding from the client's live funnel** (second source: their website): logo, fonts, colors, imagery, and the copy tone. No logo hunting, no brand-guide PDF required. You can optionally hand it an extra design reference (a website or deck you like) to inform the look.
4. **Writes the narrative, not just the numbers:** the headline result, what drove it, a per-funnel breakdown, and 2-3 concrete next steps. Plain language, every percentage next to its absolute number, bad months stated honestly and framed as next month's plan — the report always reads like the sender has things under control.
5. **Builds a slide-deck presentation** (6-9 slides, single self-contained HTML file): full-bleed brand photo on the title slide, count-up headline stat, an animated chart drawn from the real weekly numbers, a funnel-step view with per-step drop-offs, comparison table, and next steps. Entrance animations, arrow-key navigation, alternating dark and light slides.
6. **Delivers everything send-ready:** the HTML file, a stable web link the client can open in any browser, a finished landscape PDF for the classic email attachment, and a 2-sentence summary to paste into the email you send with it. Plus the offer to set it up as a recurring monthly report.

## Example

[See an example report](https://claude.ai/code/artifact/738aeead-82d9-4eb8-8585-cfa91335265e?open_in_browser=1) built with this skill: a July performance report for a retreat brand, with the branding pulled straight from their funnel. Demo data, illustrative numbers — your report will look like *your* client, not like this one.

## Requirements

- **Perspective MCP** connector, to read the real funnel data. No account yet? See [How to connect the Perspective MCP with Claude](https://intercom.help/perspective-funnels/en/articles/15374243-how-to-connect-perspective-mcp-with-claude) (about 2 minutes).
- **Browser** connector, optional, to pull branding and screenshots from the live funnel pages.

Without the Perspective MCP the skill stops and points to the setup instead of mocking anything up: the report stands on real numbers.

## Install

1. Download **`branded-client-report.skill`** from this repo.
2. In Claude (Cowork / desktop), open it and click **Save skill**, or add it under **Settings, Capabilities**.
3. Make sure the Perspective MCP is active, then ask something like *"Create a branded client report for [client]'s funnels, for July"*, *"Create the monthly report for [client]"*, or *"I need to report these numbers to my client."*

You can also install from source by copying `SKILL.md` into your skills directory.

## Repository structure

```
.
├── README.md
├── branded-client-report.skill     # packaged, installable skill
└── SKILL.md                        # skill instructions + frontmatter
```

## Guardrails

- Every number in the report comes from the pulled funnel metrics. The skill never invents, estimates, or "improves" numbers, and never offers to.
- Charts are built strictly from the pulled data and must re-sum to the reported totals; with too little data, the chart is skipped rather than faked.
- Never invents or approximates a logo, and confirms the extracted branding in one line before building.
- No Perspective branding anywhere in the output — the report belongs to the sender, not to us.
- Funnel weaknesses are framed as the sender's action plan, never as flaws in the work: the recipient is the sender's client.

## License

No license file is included yet. Add one (e.g. MIT) if you want to set explicit reuse terms.
