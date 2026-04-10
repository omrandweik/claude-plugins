---
name: lint
description: Use when the user says "lint", "health check", "check the wiki", or "find issues". Runs structural integrity checks, content quality audits, contradiction detection, and staleness tracking across the wiki.
---

# Lint

## When to Use

When the user requests a health check of the wiki, or as part of periodic maintenance. Lint checks structural integrity, content quality, staleness, and knowledge gaps.

## Protocol

### 1. Structural Checks

Scan all wiki pages for:

- **Broken backlinks**: `[[wikilinks]]` pointing to non-existent pages
- **Orphan pages**: Pages with no inbound links from other pages
- **Missing cross-references**: Concepts mentioned in body text that have their own page but aren't wikilinked
- **Index sync**: Pages that exist on disk but aren't in `wiki/index.md`, or index entries pointing to missing pages

### 2. Content Quality

- **Pages with `confidence: low`** that need sourcing or verification
- **Pages with `needs_review: true`** that haven't been user-verified
- **Missing required sections**: Check each page type has its required sections per CLAUDE.md (e.g., concept pages need Interview Angle and See Also)
- **Thin pages**: Pages under 200 words that may need expansion

### 3. Contradiction Detection

Flag a contradiction when:
- A quantitative claim differs between two wiki pages by more than a trivial amount
- Two pages use the same term with materially different definitions
- A claim is contradicted by a source in `raw/`

**Resolution Protocol:**

1. Add a `## Disputed Claims` section to the relevant wiki page:

   ```markdown
   ## Disputed Claims

   ### Market impact exponent value
   - **Claim A (current wiki)**: Exponent ≈ 0.5. Source: Almgren et al. (2005).
   - **Claim B (new source)**: Exponent ≈ 0.6 for large-tick stocks. Source: Toth et al. (2011).
   - **Reconciliation**: May reflect tick-size regime differences. Range 0.4–0.7.
   - **Status**: UNRESOLVED — flagged for user review
   - **Interview note**: Knowing this debate exists is itself valuable.
   ```

2. Never delete the original claim — preserve both sides with full attribution.
3. When the user resolves: update status to `RESOLVED — [decision]`.

### 4. Staleness Tracking

Every wiki page should include staleness metadata:

```yaml
last_verified: [date]
staleness_category: fast|medium|slow
```

**Staleness Categories:**

| Category | Topics | Review Interval |
|----------|--------|-----------------|
| **fast** | Market structure, exchange rules, regulatory, venue mechanics | 90 days |
| **medium** | Strategy performance, empirical estimates, industry landscape | 180 days |
| **slow** | Theory, mathematical foundations, econometric methods | 365 days |

Flag pages where `today - last_verified > review_interval`. For `fast` category overdue pages, offer to run a web search to check for updates.

### 5. Confidence Audit

The structured confidence block:

```yaml
confidence:
  level: medium
  reason: auto-researched, not yet user-verified
  source_quality: high
  corroboration: low
  recency: high
```

**Confidence Rules:**

| Scenario | Level | Reason |
|----------|-------|--------|
| User-provided content (raw notes, coursework) | high | User-sourced, first-party knowledge |
| Ingested from peer-reviewed paper | high | Peer-reviewed academic source |
| Ingested from practitioner blog/article | medium | Single practitioner source |
| Auto-researched from web search | medium | Auto-researched, not yet user-verified |
| Synthesized across sources with agreement | high | Multiple corroborating sources |
| Synthesized across sources with disagreement | medium | Sources disagree, see Disputed Claims |
| Extrapolated or inferred | low | Inferred, not directly sourced |

**Upgrade path:** Auto-researched pages start at `medium`, upgrade to `high` after user review. Update: `reason: user-verified [date]`.

### 6. Gap Detection

Feed results into the `skills/expand/` skill:

- Wikilinks pointing to non-existent pages
- Technical terms mentioned in 2+ pages without their own article
- "See Also" references to missing pages
- Interview questions referencing uncovered concepts

### Output Format

Produce a structured lint report:

```markdown
### Wiki Lint Report — [date]

**Summary:** X issues found (Y critical, Z warnings)

#### Broken Links
| Source Page | Broken Link | Suggested Fix |
|------------|-------------|---------------|
| pairs-trading | [[ornstein-uhlenbeck]] | Create new page or remove link |

#### Orphan Pages
| Page | Created | Suggestion |
|------|---------|------------|
| sofr-mechanics | 2026-04-07 | Add links from macro-relative-value, rates-carry |

#### Stale Pages
| Page | Category | Last Verified | Days Overdue | Risk |
|------|----------|--------------|--------------|------|
| iex-order-types | fast | 2026-01-15 | 82 days | IEX specs may have changed |

#### Unresolved Disputes
| Page | Claim | Status |
|------|-------|--------|
| market-impact | exponent value | UNRESOLVED |

#### Knowledge Gaps (→ skills/expand/)
| Missing Concept | Interview Relevance | Referenced By |
|----------------|--------------------|--------------| 
| ornstein-uhlenbeck | high | 4 pages |

#### Low Confidence Pages
| Page | Level | Reason |
|------|-------|--------|
| auto-researched-topic | medium | needs user review |
```

## Examples

**Full lint:** "lint" → Scan all 50 wiki pages, produce the full report, suggest fixes for critical issues.

**Targeted lint:** "check links in the concepts/ directory" → Scan only concept pages for broken links and missing cross-references.

## Edge Cases

- **Large wiki (>100 pages):** Focus on critical issues first (broken links, contradictions), then warnings (staleness, thin pages). Don't try to fix everything in one pass.
- **Many disputes:** If >5 disputes are flagged, prioritize by interview relevance and present the top 5.
- **Git integration:** After fixing lint issues, commit: `[lint] Fixed N broken backlinks, flagged M stale pages`
