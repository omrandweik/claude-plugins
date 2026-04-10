---
name: interview
description: Use when the user says "drill me", "quiz", "quiz mode", "mock interview", "study session", or "what should I review". Generates interview questions, runs mock interviews, tracks performance, and suggests study plans.
---

# Interview

## When to Use

When the user wants interview preparation: quizzing, mock interviews, study sessions, or performance tracking.

## Protocol

### Difficulty Tiers

| Tier | Format | Example |
|------|--------|---------|
| 1 — Recall | "What is X?" / "Define Y" | "What is the square root law of market impact?" |
| 2 — Explain | "Walk me through X" / "How does Y work?" | "Walk me through how you'd construct a pairs trading strategy" |
| 3 — Apply | "What happens when X?" / "How would you handle Y?" | "Your spread just blew through 3 sigma. What do you do?" |
| 4 — Critique | "What's wrong with X?" / "Why might Y fail?" | "Someone shows you a backtest with Sharpe 4.0. What are your concerns?" |
| 5 — Design | "Build X from scratch" / "How would you approach Y?" | "Design a mid-frequency equity stat arb strategy end-to-end" |

### Quiz Generation Protocol

1. **Select topics** based on:
   - Pages with `interview_relevance: high` not recently quizzed
   - Pages reviewed least recently (spaced repetition)
   - User's weakest areas (from quiz tracker)
   - Random sampling across categories for breadth

2. **Generate a question** at the appropriate tier. Start at Tier 2-3. Escalate to 4-5 if the user answers well. Drop to 1 if they struggle.

3. **After the user answers**, evaluate:
   - **Accuracy:** Core concept correct?
   - **Depth:** Beyond surface level?
   - **Nuance:** Edge cases, practical considerations, known debates?
   - **Communication:** Clearly structured? (Matters in real interviews)

4. **Provide feedback** referencing specific wiki pages:

   ```
   Good answer on the core concept. You missed two things:
   1. Permanent vs. temporary impact decomposition — see [[market-impact-square-root-law]]
   2. Interaction with alpha decay — see [[alpha-decay-and-signal-half-life]]
   
   Follow-up (Tier 4): "If your alpha has a 2-hour half-life and you're trading 5% of ADV, 
   how do you think about optimal execution horizon?"
   ```

### Performance Tracking

Maintain `wiki/meta/quiz-tracker.md`:

```markdown
# Quiz Performance Tracker

## Topic Scores (rolling last 5 quizzes per topic)
| Topic | Last Quizzed | Avg Score (1-5) | Trend | Priority |
|-------|-------------|-----------------|-------|----------|
| market-impact | 2026-04-10 | 4.2 | up | low (strong) |
| kalman-filter | 2026-04-08 | 2.5 | — | HIGH (weak) |
| cointegration | 2026-04-09 | 3.8 | up | medium |

## Recommended Focus Areas
1. kalman-filter — scored low, hasn't been re-quizzed
2. portfolio-construction — never quizzed
3. garch-volatility — quizzed once, scored 2.0
```

### Study Session Generator

When the user says "study session" or "what should I review?":

1. Pull the quiz tracker
2. Identify 5-10 highest-priority topics (low scores + high interview relevance + longest since last review)
3. For each topic, provide:
   - The wiki page to read
   - 2-3 key points to internalize
   - A sample question they might get asked
4. After review, offer to quiz on the material

### Mock Interview Mode

When the user says "mock interview":

1. Simulate a full interview round (30-45 minutes):
   - **5 min:** Warm-up / resume walkthrough
   - **10 min:** Technical rapid-fire (Tier 1-2, breadth)
   - **10 min:** Deep-dive on one topic (Tier 3-5, depth)
   - **5 min:** Market scenario / "what would you do" (Tier 3-4)
   - **5 min:** Strategy design or case question (Tier 5)

2. Present one question at a time. Wait for the user's answer before evaluating and moving on.

3. At the end, provide a scorecard:

   ```
   ### Mock Interview Scorecard
   - Technical breadth: 4/5 — strong across micro, stats, execution. Gap on vol modeling.
   - Technical depth: 3/5 — good on market impact, surface-level on factor models.
   - Communication: 4/5 — clear structure, could be more concise on strategy walkthrough.
   - Market awareness: 3/5 — mentioned recent events but didn't connect to portfolio implications.
   - Overall: LIKELY ADVANCE — main risk is factor model depth. 
     Review: [[pca-factor-models]], [[barra-risk-model]], [[factor-crowding]]
   ```

## Examples

**Quick quiz:** "drill me on 5 questions" → Select 5 topics from high-priority areas, generate Tier 2-3 questions, evaluate answers, update quiz tracker.

**Study session:** "what should I review?" → Check tracker, identify kalman-filter (weak), portfolio-construction (never tested), garch (weak). Provide reading plan with key points and sample questions.

**Mock interview:** "mock interview" → Run full 30-45 minute simulation with resume walkthrough, rapid-fire, deep-dive, scenario, and design question. Score at end.

## Edge Cases

- **No quiz tracker exists yet:** Create `wiki/meta/quiz-tracker.md` with empty tables on first quiz session. Create the `wiki/meta/` directory if needed.
- **User wants specific topic:** "quiz me on market impact" → Override the selection algorithm, generate questions from that topic across multiple tiers.
- **User disagrees with evaluation:** Adjust the score in the tracker. The user's self-assessment is valid data.
- **User wants harder/easier:** Explicitly shift tiers up or down as requested.
