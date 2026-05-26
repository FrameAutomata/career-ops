# Mode: text — Tailored Markdown Resume

Generate a JD-tailored resume as a markdown (`.md`) file. Same keyword-injection, summary rewrite, and bullet reordering as `modes/pdf.md` — only the final render differs. Output is plain markdown that mirrors the structure of `cv.md`, so the user can paste it into their preferred editor / template without further conversion.

## Full pipeline

1. Read `cv.md` as the source of truth
2. Read `config/profile.yml` for candidate identity and contact info
3. Ask the user for the JD if it is not in context (text or URL)
4. Extract 15-20 keywords from the JD
5. Detect JD language → CV language (EN default)
6. Detect role archetype → adapt framing (use `modes/_profile.md` archetype mapping)
7. Rewrite Professional Summary by injecting JD keywords + the user's narrative bridge (same rules as `pdf` mode — NEVER invent skills)
8. Select top 3-4 most relevant projects for the JD
9. Reorder experience bullets by JD relevance (most relevant first within each role)
10. Inject keywords naturally into existing achievements (NEVER invent)
11. Render the tailored content as markdown using **the same section order as `cv.md`** (see "Output structure" below)
12. Read `name` from `config/profile.yml` → normalize to kebab-case lowercase (e.g. "John Doe" → "john-doe") → `{candidate}`
13. Write to `output/cv-{candidate}-{company}-{YYYY-MM-DD}.md`
14. Report: file path, section count, keyword coverage %, top 3 unmatched JD keywords (so the user can decide whether to manually address them)

## Output structure (mirrors cv.md)

The output must use the same headings and order as the candidate's existing `cv.md`. Do not invent new sections or reorder them. Read `cv.md` first and replicate its structure.

A typical layout looks like this — but always defer to what `cv.md` actually contains:

```markdown
# {{NAME}}

{{CONTACT_LINE}}

## Professional Summary

{{TAILORED_SUMMARY}}

## Skills

{{SKILLS — same categories as cv.md, with JD-relevant items surfaced first within each category}}

## Professional Experience

{{EXPERIENCE — each role from cv.md, bullets reordered by JD relevance, keywords injected}}

## Projects

{{TOP_3_4_PROJECTS — selected by JD relevance}}

## Education

{{EDUCATION — verbatim from cv.md unless certifications are JD-relevant and should be surfaced}}
```

Replace `{{...}}` placeholders with the tailored content. Use the candidate's exact heading text from `cv.md` (e.g. if their CV says "Professional Experience" use that, not "Work Experience").

## ATS Rules

Same intent as `modes/pdf.md`, adapted to markdown:

- Standard section headers (Professional Summary, Skills, Professional Experience, Projects, Education) — keep whatever wording `cv.md` already uses
- UTF-8, plain text (no smart quotes, no em dashes pasted from Word)
- Bullets with `-` or `•` (match `cv.md`'s existing convention)
- Distribute JD keywords: summary (top 5), first bullet of each role, skills section
- No tables, no images, no HTML, no code fences inside resume content — pure markdown prose + headings + bullets

## Keyword injection strategy (ethical, truth-based)

Identical to `modes/pdf.md`. Examples of legitimate reformulation:

- JD says "REST microservices" and CV says "Express.js APIs" → "REST microservices using Express.js"
- JD says "CI/CD pipelines" and CV says "GitHub Actions workflows" → "CI/CD pipelines with GitHub Actions"
- JD says "PostgreSQL on AWS RDS" and CV says "PostgreSQL with Supabase" → keep as-is (don't fabricate RDS)

**NEVER add skills the candidate does not have. Only reword real experience using the exact JD vocabulary.**

## Post-generation

Update tracker if the job is already registered: change the artifact column (the existing "PDF" column) from `❌` to `✅`. In this fork, that column means "resume artifact generated" — text and PDF outputs both qualify.

Report to the user:

```
output/cv-{candidate}-{company}-{YYYY-MM-DD}.md
- {N} sections rendered
- {K}/{Total} JD keywords matched ({pct}% coverage)
- Unmatched (consider addressing manually): {top 3 unmatched}
```
