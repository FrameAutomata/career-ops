# Fork Changes — FrameAutomata/career-ops

Customizations in this fork that diverge from upstream santifer/career-ops.

## Branch layout — `main` and `dev/batch-local-llm` are the same commit

`setup.ps1` / `setup.sh` in `job-search-pipeline` clone this fork with
`--branch dev/batch-local-llm`, so that branch is what every install runs.
`main` is the default branch, so it is what a plain `git clone` of this fork
gets. **Both must carry everything**: `main` without `--cli` is a trap — the
pipeline's `run.sh`/`run.ps1` always pass `--cli "$BATCH_CLI"`, and upstream's
arg parser exits 1 on an unknown option, so `--batch` dies immediately.

Keep the two refs at the same commit. Do not let `main` drift behind again, and
do not repoint setup at `main` while existing checkouts track
`dev/batch-local-llm` — they would silently stop receiving updates.

## Fork-local files

`config/local-paths.txt` (gitignored) declares `FORK_CHANGES.md` to upstream's
`validate-system-paths-coverage.mjs`, which requires every tracked file to sit in
`SYSTEM_PATHS` or `USER_PATHS`. See DATA_CONTRACT.md → "Fork-local paths"
(upstream #2421). Declaring it there rather than in `update-system.mjs` is the
point: `apply` overwrites that file and git re-merges it on every sync, so a
declaration inside it would be erased by the process it exists to constrain.

## Tailored markdown resume mode (`cv.output_format: "text"`)

**Pending upstream as [PR #3343](https://github.com/santifer/career-ops/pull/3343)
(issue [#3342](https://github.com/santifer/career-ops/issues/3342))** — delete this
section when it merges, do not re-apply it.

Adds a third option alongside `html` and `latex` for generating a tailored resume.

**Why:** both existing output paths assume career-ops should generate the document.
The fork owner already has a preferred resume format and needs only the JD-tailoring
step, not the HTML→PDF render. The markdown path also requires no toolchain
(no Playwright, no tectonic/pdflatex) and has no CJK limitation.

**What it does:** runs the same keyword extraction, summary rewrite, bullet
reordering and keyword-injection pipeline as `modes/pdf.md`, then writes the result
as markdown mirroring the structure of `cv.md`.

**How to use:**

- One-shot: `/career-ops text` (with a JD in context)
- Always-on: set `cv.output_format: "text"` in `config/profile.yml` — `auto-pipeline`
  (paste a JD with no sub-command) will route to text instead of PDF

**Output:** `output/cv-{candidate}-{company}-{YYYY-MM-DD}.md`

**Tracker:** the `PDF` column is deliberately **left alone**. It tracks a PDF indexed
in `data/pdf-index.tsv`, which `find.mjs`, the dashboard and the `email` mode read to
locate an attachment; marking it for a run that produced no PDF points them at a file
that does not exist. (The pre-#3343 version of this mode marked it `✅` — that was
wrong, and is why this note exists.)

**Keep this branch's copy byte-identical to the PR branch** (`feat/cv-output-format-text`).
Two divergent copies of one feature means the merge conflicts when #3343 lands
instead of being a no-op — the same trap `--cli` fell into below.

### Files involved

| File | Type | Status under upstream `update-system.mjs apply` |
|------|------|-------------------------------------------------|
| `modes/text.md` | new file | in SYSTEM_PATHS as of #3343 — upstream will own it once merged |
| `modes/auto-pipeline.md` Step 3 | edit | **overwritten** — re-apply after upstream updates |
| `.agents/skills/career-ops/SKILL.md` | edit (argument-hint, routing table, discovery menu, context-loading list) | the 7 other CLI surfaces are symlinks to it, so one edit covers all |
| `config/profile.example.yml` | edit (the option + a `cv.sections` note) | survives |
| `modes/README.md`, `DATA_CONTRACT.md`, `update-system.mjs` | edit (mode registry, system-layer table, SYSTEM_PATHS) | **overwritten** — re-apply after upstream updates |

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
