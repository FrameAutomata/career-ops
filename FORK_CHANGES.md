# Fork Changes — FrameAutomata/career-ops

Customizations in this fork that diverge from upstream santifer/career-ops.

## Tailored markdown resume mode (`cv.output_format: "text"`)

Adds a third option alongside `html` and `latex` for generating a tailored resume.

**Why:** the fork owner already has a preferred resume format and only needs the JD-tailoring step, not the HTML→PDF render.

**What it does:** runs the same keyword extraction, summary rewrite, bullet reordering, and keyword-injection pipeline as `modes/pdf.md`, then writes the result as markdown that mirrors the structure of `cv.md`. No HTML, no Playwright, no PDF.

**How to use:**

- One-shot: `/career-ops text` (with a JD in context)
- Always-on: set `cv.output_format: "text"` in `config/profile.yml` — `auto-pipeline` (paste a JD with no sub-command) will route to text instead of PDF

**Output:** `output/cv-{candidate}-{company}-{YYYY-MM-DD}.md`

**Tracker:** the existing `PDF` column in `data/applications.md` is reused — in this fork it means "resume artifact generated" (text or PDF both qualify).

### Files involved

| File | Type | Status under upstream `update-system.mjs apply` |
|------|------|-------------------------------------------------|
| `modes/text.md` | new file | survives (not in SYSTEM_PATHS) |
| `modes/auto-pipeline.md` Step 3 | edit | **overwritten** — re-apply this fork's branch after upstream updates |
| `.agents/skills/career-ops/SKILL.md` | edit (argument-hint, routing table, discovery menu, context-loading list) | **overwritten** — re-apply after upstream updates |
| `config/profile.example.yml` line 70 | edit (comment) | survives (not in SYSTEM_PATHS) |

If you ever run `node update-system.mjs apply` and lose the `text` branch in auto-pipeline / SKILL.md, just re-apply this branch's commits on top — the rest of the wiring is intact.
