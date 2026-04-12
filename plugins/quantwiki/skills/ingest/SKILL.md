---
name: ingest
description: Use when the user says "ingest", "process this", drops a new file in raw/, or the auto-detect hook reports new unprocessed files. Processes source documents (PDF, docx, markdown, RTF) into wiki pages.
---

# Ingest

## When to Use

When the user provides a new source document (PDF, docx, markdown, RTF, etc.) in `raw/` and wants it processed into wiki pages. Also triggered by the auto-detect hook reporting new unprocessed files.

## Protocol

### Short Documents (<20 pages / <5000 words)

1. Read the full document
2. Identify key topics and how they map to existing wiki structure
3. Present a brief summary: "This covers X, Y, Z. I'll create N new pages and update M existing pages. Proceed?"
4. On confirmation, execute the full ingest:
   - Write new wiki articles following the article template in CLAUDE.md
   - Update existing articles with new information (cite the new source)
   - Add cross-references and backlinks
   - Update `wiki/index.md`
   - Append to `log.md`
5. After ingest, report what was done and flag any contradictions with existing content
6. Git commit: `[ingest] Processed [filename] — N pages created/updated`

### Long Documents (>20 pages / >5000 words)

**Step 1: Triage**

1. Extract the table of contents, section headers, and abstract/introduction only
2. Cross-reference against the existing wiki index
3. Produce a selective ingest plan:

   ```
   Source: raw/coursework/financial-engineering-notes.pdf (280 pages)

   Recommended sections to ingest:
   - Ch 3: Stochastic Calculus → enriches options-pricing.md, creates itos-lemma.md
   - Ch 7: Time Series Models → enriches alpha-decay.md, creates garch-volatility.md

   Sections to skip (low interview relevance):
   - Ch 1-2: Review of probability (already covered)
   - Ch 9: Fixed income analytics (out of scope)
   ```

4. Wait for user approval on the ingest plan before proceeding.

**Step 2: Incremental Processing**

Process approved sections one at a time:

1. Read the section
2. Discuss key points with the user
3. Write/update wiki pages
4. Update index and log
5. Confirm before moving to next section

**Step 3: Source Tracking**

For long documents, track ingest progress in `log.md`:

```markdown
## [YYYY-MM-DD] ingest | Financial Engineering Course Notes (partial)
- Source: raw/coursework/financial-engineering-notes.pdf
- Sections processed: Ch 3 (Stochastic Calculus), Ch 7 (Time Series)
- Sections remaining: Ch 12 (Portfolio Optimization) — approved, pending
- Pages created: 4
- Pages updated: 6
```

### Batch Ingest (multiple files at once)

1. List all files with a one-line description of each
2. Suggest an ingest order (most foundational/highest-priority first)
3. Process one at a time with brief summaries between each

### Source Classification

Automatically classify ingested sources and tag them:

- `type: academic-paper` → highest rigor expected, cite specific claims
- `type: practitioner-article` → good for implementation details, note potential bias
- `type: course-notes` → foundational material, link to more authoritative sources where possible
- `type: conversation-export` → treat as working notes, verify key claims
- `type: web-article` → check publication date, note if potentially outdated
- `type: exchange-documentation` → authoritative for market structure, check for updates
- `type: interview-prep` → directly populate `interview/` wiki section

### Rules

- **Never modify** the source file in `raw/`
- Always set `sources:` frontmatter to reference the raw file
- If the source contradicts an existing wiki page, flag the contradiction (see `skills/lint/` for dispute protocol)
- Prefer updating existing pages over creating near-duplicates
- Target: 5-15 pages touched per source document
- Always update `wiki/index.md` and append to `log.md`
- **Obsidian compatibility:** Always use `[[page-name]]` wikilink syntax (never markdown links). Add `aliases:` in frontmatter for common alternate names. Use block-style YAML tag lists. Follow the tag taxonomy in CLAUDE.md (category + relevance + confidence + staleness + 1-3 domain tags).

## Examples

**Short ingest:** "ingest Trading Notes.docx" → Read the docx, identify options/Greeks/IV content, propose creating `options-greeks-and-iv.md` and updating `vol-selling-overlays.md`, execute on confirmation.

**Long ingest:** "ingest financial-engineering-notes.pdf (280 pages)" → Read TOC only, cross-reference with wiki index, propose selective plan covering Ch 3, 7, 12 while skipping already-covered material, process incrementally with user approval between sections.

## Edge Cases

- **Binary files (docx, pdf):** Use python zipfile extraction for docx, Read tool with pages parameter for PDFs. For large PDFs (>10 pages), must specify page ranges.
- **Massive overlap:** If a source is 90%+ already covered (like a probability cheat sheet when probability content exists), skim for gaps only and report "low marginal value, N items extracted."
- **Contradictions:** Never silently overwrite. Flag and follow the dispute protocol in `skills/lint/SKILL.md`.
- **Multiple sources referencing same topic:** Merge into the existing wiki page rather than creating duplicates. Add the new source to the `sources:` frontmatter list.
