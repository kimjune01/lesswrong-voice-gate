# Pre-Registration: LessWrong Voice Gate via Inductive Lineup

## Research question

Can a frontier model distinguish a candidate blog post from high-karma LessWrong posts in a blind lineup, using only positive samples (no "AI slop" definition, no rubric)?

## Hypothesis

**H0:** GPT-5.5 identifies the candidate as the outlier at chance rate (1/N where N = lineup size).

**H1:** GPT-5.5 identifies the candidate at a rate significantly above chance, and the stated reasons correspond to voice/register mismatch rather than topic mismatch.

## Why induction, not rubric

Prior work ([slop-detection](/slop-detection)) showed that explicit rubrics become exploit surfaces under adversarial feedback. Two iterations collapsed the score gap. A rubric-free lineup forces the model to induce the distribution from examples, making the detection criteria implicit and harder to game.

## Design

### Positive sample collection

- **Source:** LessWrong frontpage posts with karma >= 100, published 2024-2026
- **Sample size:** 30 posts, stratified by topic cluster:
  - AI alignment / safety (10)
  - Rationality / epistemics (10)
  - Applied / technical / field reports (10)
- **Exclusion:** posts by Eliezer, Scott Alexander, Zvi — too recognizable, would anchor the lineup on celebrity voice rather than site voice
- **Format:** title + body, stripped of metadata, author, karma

### Lineup construction

Each trial:
1. Draw 5 positive samples from the pool (without replacement within trial, with replacement across trials)
2. Insert 1 candidate post as the 6th entry
3. Shuffle into random order with neutral labels (Text A through F)
4. Prompt: "Here are 6 posts. One may not belong to the same distribution as the others. Which one, and why? If they all belong, say so."

### Candidate posts

- **Condition 1 (calibration):** known LessWrong posts held out from the pool (should NOT be identified as outliers)
- **Condition 2 (baseline):** raw Claude-generated posts on LW-adjacent topics (should be identified)
- **Condition 3 (humanized):** Claude posts after copyedit pipeline (humanize → tighten → sharpen)
- **Condition 4 (target):** the user's actual blog posts from june.kim

### Trial count

- 30 trials per condition = 120 total
- Each trial uses a fresh draw of 5 positive samples
- Candidate post is fixed within a condition but lineup composition varies

### Statistical plan

**Primary metric:** outlier identification accuracy per condition.

- Chance baseline: 1/6 ≈ 16.7%
- Detection threshold: significantly above chance at p < 0.05 (one-sided binomial test)
- With 30 trials: need >= 9/30 correct identifications to reject H0 (binomial, n=30, p=1/6, alpha=0.05)

**Secondary metrics:**
- Reason classification: for each correct identification, code the stated reason as {voice, structure, topic, substance, style, other}
- False positive rate on Condition 1 (should be ≈ chance)
- Condition 3 vs Condition 4 gap: does the copyedit pipeline close the gap toward the user's actual voice?

### Success criteria

The experiment succeeds (voice gate is viable) if:
1. Condition 1 (held-out LW posts) identification rate is NOT significantly above chance (false positive check)
2. Condition 2 (raw AI) identification rate IS significantly above chance
3. Condition 4 (user's posts) provides the actionable signal: either the user passes (identification ≈ chance, ready to submit) or the reasons name specific fixable tells

### Failure modes

- **Topic confound:** model identifies the outlier by topic mismatch, not voice. Mitigated by topic-matching the candidate to the lineup draw.
- **Length confound:** LW posts vary wildly in length. Mitigated by trimming all entries to first 2000 words.
- **Memorization:** model recognizes specific LW posts from training. Mitigated by excluding the most famous authors and checking if Condition 1 accuracy is above chance (it shouldn't be).
- **Low power:** 30 trials per condition may be insufficient. Power analysis: at 30 trials, we can detect a true accuracy of >= 40% with 80% power (binomial test, H0 p=1/6). If the effect is smaller than that, we need more trials.

## Models

- **Primary:** GPT-5.5 via codex (same model used in slop-detection, continuity with prior work)
- **Secondary:** Gemini 3.1 Pro (independent model family, cross-validation)
- If the two models disagree on outlier identification, that's informative — it suggests the detection signal is model-specific rather than structural.

## Outputs

- `data/positive-samples/` — 30 LW posts, stripped
- `data/candidates/` — posts per condition
- `trials/` — one file per trial with lineup, prompt, response, scored result
- `analysis.py` — binomial tests, reason coding, summary stats
- `RESULTS.md` — written after all trials complete, not before

## What we will NOT do

- No rubric in the prompt. The whole point is induction from examples.
- No iterative refinement during the experiment. Run all 120 trials, analyze, then decide.
- No cherry-picking which trials to report. All trials, all results.
- No post-hoc metric invention. The metrics above are the metrics.

## Timeline

1. Collect positive samples (scrape 30 LW posts)
2. Prepare candidates (4 conditions)
3. Run 120 trials
4. Score and analyze
5. Write RESULTS.md
