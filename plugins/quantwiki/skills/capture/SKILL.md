---
name: capture
description: Use when the user says "capture this", "save this to wiki", "file this", or "this should be a page". Files conversation insights back into the wiki as new or updated pages.
---

# Capture

## When to Use

When the user wants to file an insight from a conversation back into the wiki. Also used proactively to suggest capturing substantive Q&A exchanges.

## Protocol

### The Capture Command

When the user says "capture this", "save this to wiki", "file this", or "this should be a page":

1. **Identify the core insight** from the preceding conversation:
   - An answer that synthesized multiple wiki pages
   - A new connection between concepts not previously documented
   - A worked-through problem or derivation
   - A mock interview exchange with useful follow-up chains
   - A clarification or correction to existing wiki content

2. **Determine the right action:**
   - New standalone topic → create a new wiki page
   - Enriches an existing page → append with a `## Captured Insight [date]` header
   - Comparison or connection → create a comparison page or add cross-references
   - Interview Q&A → add to the relevant interview prep page

3. **Tag captured content** in frontmatter:
   ```yaml
   source_type: captured-from-session
   capture_date: [date]
   session_context: [brief note, e.g., "mock interview on factor models"]
   ```

4. Update `wiki/index.md` and append to `log.md`.

5. Git commit: `[capture] Filed [topic] discussion into wiki`

### Auto-Capture Suggestions

After any substantive Q&A exchange (more than a one-line answer), proactively offer:

> "This answer touches on [X, Y, Z]. Would you like me to capture this into the wiki? I'd update [specific pages] and/or create [new page]."

The user can say yes, no, or modify what gets captured.

**Critical rule: Never auto-capture without asking.** The user decides what's worth keeping.

## Examples

**New page capture:** After a deep discussion about Hawkes processes for order flow modeling → "This could be its own concept page. Want me to create `hawkes-process-order-flow.md`?"

**Existing page enrichment:** After answering a question about GARCH vs realized vol → "I can add this comparison to `time-series-arma-garch.md` under a new section. Want me to?"

**Interview Q&A capture:** After a mock interview exchange on market impact → "Those follow-up questions were good — want me to add them to `market-impact-square-root-law.md` under Interview Angle?"

## Edge Cases

- **Trivial exchanges:** Don't offer to capture one-liner answers or factual lookups. Only suggest capture for substantive, multi-paragraph content.
- **Duplicate content:** Before creating a new page, check if the topic is already covered. If so, offer to enrich the existing page instead.
- **Corrections:** If the conversation corrected an error in the wiki, capture should update the existing page (not create a new one) and note the correction in the page's edit history.
