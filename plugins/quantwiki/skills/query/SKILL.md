---
name: query
description: Use when the user asks any question about quantitative finance, interview prep, strategies, concepts, or wiki content. Also triggered by "ask" or any question that can be answered by searching the wiki.
---

# Query

## When to Use

When the user asks a question that can be answered by searching and synthesizing wiki content. This is the default mode for any question about quantitative finance, interview prep, strategies, or concepts.

## Protocol

1. **Search**: Read the top-level `wiki/index.md` to identify relevant pages. If the wiki has scaled to hierarchical indexes, read the appropriate sub-index.
2. **Read**: Open the relevant wiki pages and extract the answer.
3. **Synthesize**: Combine information from multiple pages if needed. Always cite sources using `[[page-name]]` wikilink syntax.
4. **Gap detection**: If the question reveals a gap in the wiki (no page covers this topic, or coverage is shallow), note it: "This isn't well-covered in the wiki yet. Want me to research and file it?"
5. **File-back offer**: If the answer is substantial enough to be a standalone article or comparison, offer to capture it: "This answer touches on [X, Y, Z]. Would you like me to capture this into the wiki?"

### Answer Format

- Use markdown formatting (headers, tables, LaTeX math, code blocks) as appropriate
- Always cite the wiki pages used: "See [[page-name]] for more detail"
- For interview-oriented questions, include "table stakes vs. impressive" calibration
- For quantitative questions, include formulas in LaTeX
- Keep answers practitioner-depth — no introductory explanations

### Index Navigation

**Current (flat index):** Read `wiki/index.md` directly to find pages.

**After scaling (>120 pages, hierarchical):** Read `wiki/index.md` to identify which sub-index is relevant (e.g., `index-concepts.md`), then read that sub-index to find specific pages. This two-hop lookup keeps context window usage manageable.

**After Phase 2 (>300 pages, search available):** Search first using the search tool, then fall back to index browsing for exploration.

## Examples

**Direct question:** "What is the square root law?" → Read `wiki/index.md`, find `market-impact-square-root-law`, read that page, synthesize answer with citation.

**Cross-cutting question:** "How does alpha decay affect execution?" → Read both `alpha-decay-and-signal-half-life` and `transaction-cost-analysis`, synthesize the connection, cite both.

**Gap-revealing question:** "What's the Avellaneda-Lee framework?" → Search wiki, find no dedicated page but mentions in `stat-arb-overview`. Answer from claude-knowledge, note: "No dedicated page exists. Want me to research and create one?"

## Edge Cases

- **Question outside wiki scope:** If the question is about a topic far from quant finance / interview prep, answer from general knowledge but don't offer to file it unless the user asks.
- **Contradictory wiki content:** If two pages disagree, present both views and reference the dispute. Don't pick a side without flagging it.
- **Stale content:** If you suspect a wiki page's information may be outdated (e.g., regulatory rules, exchange mechanics), note the staleness risk in your answer.
