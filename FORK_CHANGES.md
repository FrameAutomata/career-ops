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

---

## Multi-CLI batch runner (`batch/batch-runner.sh --cli`)

Branch: `dev/batch-local-llm`. **Pending upstream as [PR #738](https://github.com/santifer/career-ops/pull/738)** —
this section should be deleted, not re-applied, once that merges.

**Why:** upstream's runner is Claude Code-only — it dispatches `claude -p` with
`--dangerously-skip-permissions` and `--append-system-prompt-file`, flags no other
CLI has. `job-search-pipeline`'s `--batch` mode always passes `--cli "$BATCH_CLI"`
(`run.sh` / `run.ps1`), so without this flag the pipeline's interactive batch path
dies on `Unknown option: --cli`.

**What it does:** adds `--cli claude|opencode|gemini|qwen`. The `claude` arm is
upstream's code path unchanged. The other three have no `--append-system-prompt-file`,
so the resolved prompt file is inlined ahead of the user prompt as a single string.
`--parallel > 1`, `--strict-mcp-config` and the rate-limit/session retry loop are
claude-only; the retry loop's detection only matches claude logs, so other CLIs
fall through to a single attempt. `spend_tier` resolves to Claude model names, so
non-claude CLIs take `--model` verbatim or their own default. `opencode` falls back
to `ollama launch opencode -y -- run` when the `opencode` binary is not in PATH.

**Keep this branch's `batch-runner.sh` byte-identical to the PR branch**
(`feat/batch-multi-cli`). Two divergent copies of the same feature in one fork means
the merge conflicts when #738 lands, instead of being a no-op.

**Superseded by upstream:** `--skip-pdf` and `--min-score` started here; `--min-score`'s
cross-platform float fix went up as PR #735 (merged) and upstream has since grown its
own `--skip-pdf`. This fork now carries upstream's.

### Files involved

| File | Type | Status under upstream `update-system.mjs apply` |
|------|------|-------------------------------------------------|
| `batch/batch-runner.sh` | edit (header, usage, arg parse, preflight, model resolution, worker dispatch, run banner) | not in SYSTEM_PATHS, but upstream rewrites this file often — expect conflicts on merge, and resolve by taking `feat/batch-multi-cli`'s version rather than hand-merging |
| `AGENTS.md` OpenCode row | edit (documents the ollama fallback) | **overwritten** — re-apply after upstream updates |
