---
name: expand
description: Use when the user says "expand", "find gaps", "what's missing", "suggest new topics", or "research [topic]". Detects knowledge gaps in the wiki and optionally fills them through autonomous research.
---

# Expand

## When to Use

When the user wants to identify knowledge gaps in the wiki and optionally fill them through autonomous research. Also triggered by gap detection results from the lint skill.

## Protocol

### Gap Detection

1. **Scan all wiki pages** for concept references lacking their own page:
   - Wikilinks (`[[concept-name]]`) pointing to non-existent pages
   - Technical terms mentioned in 2+ pages without a dedicated article
   - "See Also" sections referencing pages that don't exist
   - Interview questions referencing concepts without wiki coverage

2. **Score each gap** on two dimensions:
   - `interview_relevance`: How likely to come up in a stat arb / mid-frequency equities pod interview? (high/medium/low)
   - `connectivity`: How many existing pages reference this concept? Higher = more central.

3. **Produce a ranked gap report:**

   ```
   | Rank | Missing Concept | Interview Relevance | Referenced By (count) | Suggested Sources |
   |------|----------------|--------------------|-----------------------|-------------------|
   | 1    | ornstein-uhlenbeck-process | high | 4 pages | Chan (2009), Avellaneda & Lee (2010) |
   | 2    | information-ratio | high | 3 pages | Grinold & Kahn (2000) |
   ```

4. **Wait for user approval** before creating new pages. Present the gap report and ask: "Which of these should I research and write?"

### Autonomous Research Protocol

When approved to fill a gap:

1. **Search for primary sources** using web search. Priority order:
   - Academic papers (SSRN, arXiv, NBER) — theoretical foundations
   - Practitioner publications (AQR, Man Group, Two Sigma) — practical implementation
   - Exchange/regulatory documentation (SEC, CME, NYSE) — market structure
   - High-quality technical blogs (QuantStart, Ernie Chan, Lopez de Prado) — implementation
   - Textbook references — cite specific chapters/sections

2. **Never use a single source.** Each auto-generated article must synthesize at least 2-3 sources. Cross-reference claims.

3. **Mark auto-researched content** in frontmatter:

   ```yaml
   source_type: auto-researched
   confidence: medium  # starts medium, upgrades to high after user review
   sources_consulted: [list URLs/papers]
   needs_review: true
   ```

4. **File raw sources**: Save useful web articles as markdown in `raw/web/` for future reference.

5. **Update backlinks**: After writing the article, update all pages that referenced the missing concept — add proper wikilinks and new cross-references.

6. **Git commit**: `[expand] Auto-researched [page-name].md`

### Proactive Topic Suggestions

Beyond filling detected gaps, suggest new topic areas not yet mentioned in the wiki:

> "Given the existing wiki covers [X, Y, Z], a stat arb interview would also likely cover [A, B, C] which are adjacent/foundational topics."

Present suggestions grouped by theme:

```
### Suggested Additions (not yet mentioned in wiki)

**Execution & Microstructure**
- Optimal execution with alpha decay (Almgren-Chriss with signal)
- Queue-reactive models for limit order placement
- Maker-taker vs. inverted fee models

**Statistical Methods**
- Ornstein-Uhlenbeck parameter estimation
- Hidden Markov models for regime detection

**Strategy & Portfolio**
- Avellaneda-Lee stat arb framework
- Barra risk model architecture
- Factor crowding detection
```

## Examples

**Gap scan:** "find gaps" → Scan all wiki pages, identify 15 missing concepts, rank by interview relevance x connectivity, present top 10 with suggested sources.

**Targeted research:** "research ornstein-uhlenbeck" → Web search for 2-3 sources, synthesize into a new concept page, tag as auto-researched, update all pages that reference OU.

## Edge Cases

- **User declines all suggestions:** Respect this. Don't repeatedly suggest the same topics across sessions.
- **No good sources found:** Mark `confidence: low` and note in the page that better sourcing is needed. Don't fabricate or hallucinate content.
- **Topic too broad:** If a suggested topic would be better as 2-3 atomic pages, propose the split rather than writing one overloaded page.
