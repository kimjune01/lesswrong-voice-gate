# Pre-Registration: LessWrong Voice Gate via Inductive Lineup

## Research question

Can a frontier model distinguish a candidate blog post from high-karma LessWrong posts in a blind lineup, using only positive samples (no "AI slop" definition, no rubric)?

## Hypotheses

**H0 (null):** GPT-5.5 identifies the candidate as the outlier at chance rate (1/6).

Three confound hypotheses explain *why* the model detects the outlier. Each predicts different reason codes and responds to different perturbations:

**H_topic:** Detection is driven by topic mismatch. The candidate writes about GitHub contribution pipelines; the lineup writes about AI alignment. Perturbation: reframe the candidate through a LW-native lens (e.g., "adversarial dynamics in open-source contribution markets"). If detection drops, topic was the signal.

**H_structure:** Detection is driven by format/structural mismatch. LW posts use specific patterns: footnotes, epistemic status headers, inline caveats, nested parenthetical asides, section-less flowing argument. Perturbation: reformat the candidate into LW structural conventions without changing the argument. If detection drops, structure was the signal.

**H_style:** Detection is driven by prose register mismatch. LW has a specific vocabulary, hedging pattern, and compression style distinct from HN/personal blogs. Perturbation: rewrite the candidate in LW register without changing topic or structure. If detection drops, style was the signal.

**H_voice (target):** Detection persists after controlling for topic, structure, and style — the candidate lacks something irreducible about the author's relationship to the LW intellectual community. This is the hard signal and the one worth investigating if the others are eliminated.

The hypothesis graph is: H_topic → H_structure → H_style → H_voice. Each perturbation kills or confirms one edge. The investigation follows the surviving path.

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

### Topic gate (pre-lineup)

Before voice testing, run a topic fitness check. Not every june.kim post belongs on LessWrong — the site has a specific appetite and a candidate must fit it.

1. Extract the candidate's central question in one sentence
2. Present it alongside 10 real LessWrong post titles+abstracts (high karma, mixed topics)
3. Prompt: "A user is deciding where to post this. Does it belong in this collection, or somewhere else? If somewhere else, where?"

A candidate that gets routed elsewhere (Hacker News, personal blog, ML subreddit) doesn't enter the voice lineup. The topic gate filters before the voice gate — no point polishing voice for an audience that won't click.

**Topic translation:** some june.kim posts have LW-relevant substance but wrong framing. "Bot-magnet issues on GitHub" is an HN post. "Adversarial dynamics in open-source contribution markets" is a LW post. Same content, different lens. The topic gate should also report: "this could fit if reframed as X."

### Candidate posts

- **Condition 1 (calibration — should pass):** known LessWrong posts held out from the pool
- **Condition 2 (HN negative — should fail):** high-scoring Hacker News blog posts. Same internet, different community. Tests whether the lineup detects community membership, not just "good writing." Source: `data/negative-samples-hn/`
- **Condition 3 (raw AI — should fail):** Claude-generated posts on LW-adjacent topics
- **Condition 4 (humanized AI):** Claude posts after copyedit pipeline (humanize → tighten → sharpen)
- **Condition 5 (target):** the user's actual blog posts from june.kim, reframed through the topic gate's suggested lens if needed
- **Condition 6 (perturbation):** Condition 5 posts rewritten to control for one confound at a time (topic-matched, then structure-matched, then style-matched). Each perturbation isolates one hypothesis edge.

### Trial count

- 20 trials per condition × 6 conditions = 120 total
- Each trial uses a fresh draw of 5 positive samples
- Candidate post is fixed within a condition but lineup composition varies

### Statistical plan

**Primary metric:** outlier identification accuracy per condition.

- Chance baseline: 1/6 ≈ 16.7%
- Detection threshold: significantly above chance at p < 0.05 (one-sided binomial test)
- With 20 trials: need >= 7/20 correct identifications to reject H0 (binomial, n=20, p=1/6, alpha=0.05)

**Secondary metrics:**
- Reason classification: for each correct identification, code the stated reason as {topic, structure, style, voice, substance, other} — maps directly to confound hypotheses
- False positive rate on Condition 1 (should be ≈ chance)
- Condition 2 (HN) vs Condition 3 (raw AI) comparison: does the model detect community mismatch or AI-ness?
- Condition 6 (perturbation) trajectories: which perturbation kills detection? That identifies the dominant confound.

### Success criteria

The experiment succeeds (voice gate is viable) if:
1. Condition 1 (held-out LW) detection rate ≈ chance (false positive check)
2. Condition 2 (HN) detection rate significantly above chance (the lineup detects community, not just quality)
3. Condition 3 (raw AI) detection rate significantly above chance (baseline sensitivity)
4. Condition 6 (perturbations) produces a monotonic trajectory: each confound removed reduces detection rate. The confound hierarchy H_topic → H_structure → H_style → H_voice is ordered by which perturbation has the largest effect.
5. Condition 5 (user's posts) provides actionable signal: either passes (ready to submit) or the surviving confound names the fix

### Failure modes

- **Topic confound:** model identifies the outlier by topic mismatch, not voice. Mitigated by topic-matching the candidate to the lineup draw.
- **Length confound:** LW posts vary wildly in length. Mitigated by trimming all entries to first 2000 words.
- **Memorization:** model recognizes specific LW posts from training. Mitigated by excluding the most famous authors and checking if Condition 1 accuracy is above chance (it shouldn't be).
- **Low power:** 20 trials per condition may be insufficient. Power analysis: at 20 trials, we can detect a true accuracy of >= 45% with 80% power (binomial test, H0 p=1/6). If the effect is smaller, we need more trials.

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

## Prereg checklist (22 questions)

| # | Source | Answer |
|---|--------|--------|
| 1 | Bacon | Positive samples are stratified by topic cluster to prevent cherry-picking posts that favor or disfavor detection. Lineup draws are randomized. |
| 2 | Bacon | Data collection is fixed: 30 LW posts collected before any trials run. Candidate selection per condition is fixed before trials. No adjustments after seeing results. |
| 3 | Descartes | **Critical assumption:** the model's outlier detection is driven by voice/register, not topic or memorization. If false, the experiment measures topic fit or author recognition, not voice. Condition 1 (calibration) tests this — if held-out LW posts are detected as outliers, the assumption fails. |
| 4 | Hume | The mechanism: LW posts share distributional properties (vocabulary, argument structure, epistemic register) that a model can induce from examples. An outlier deviates from this induced distribution. The claim is correlational — "detected as outlier" correlates with "wouldn't survive on LW" — not causal. We don't claim detection *causes* rejection. |
| 5 | Hume | Results apply to GPT-5.5's detection ability on English-language rationalist blog posts. No claim about other models, other communities, other languages, or human readers. |
| 6 | Mill | Each condition varies one thing: Condition 1 = real LW post (null), Condition 2 = raw AI, Condition 3 = humanized AI, Condition 4 = user's actual writing. Lineup composition varies across trials but is randomized, not systematically different between conditions. |
| 7 | Mill | Control is Condition 1: held-out LW posts that should NOT be detected. This isolates "outlier detection" from "detecting anything that isn't in the training pool." |
| 8 | Chamberlin | Competing explanations for high detection on Conditions 2-4: (a) topic mismatch, not voice — mitigated by topic matching; (b) length mismatch — mitigated by trimming to 2000 words; (c) memorization of specific LW posts — mitigated by author exclusion + Condition 1 check; (d) the model is random but we got lucky — addressed by binomial test. We test between (a) and "true voice detection" via reason coding. |
| 9 | Peirce | The hypothesis (lineup detection works) comes from the PR-description crosscheck in the drip pipeline, which uses the same lineup method. The LW application is a new domain. We are not inferring the hypothesis from this data. |
| 10 | Fisher | Lineup position is randomized (shuffled labels A-F). Positive sample draws are randomized. No systematic assignment bias. |
| 11 | Popper | **Falsification:** if Condition 1 identification rate is significantly above chance (>= 9/30), the method is broken — it's detecting "not in the pool" rather than "doesn't belong." If Condition 2 identification rate is NOT above chance, the method has no power. Either outcome kills the approach. |
| 12 | Popper | The bar (>= 9/30, p < 0.05 one-sided binomial) is informative: chance is 5/30. The gap between chance (5) and threshold (9) is meaningful. We are not testing "model does better than a coin flip" — we're testing "model does better than random guessing on a 6-way forced choice." |
| 13 | Kuhn | **Invisible assumption:** "high karma = good LW post." Karma correlates with agreement and timing, not just quality. A >= 100 threshold filters for broad approval but may exclude contrarian or niche posts that represent the real LW distribution. Also: "LW voice" may not be a single distribution — alignment posts vs rationality posts vs field reports may have different registers. Stratification partially addresses this. |
| 14 | Platt | The experiment excludes the alternative "lineup detection is just memorization" (via Condition 1). It excludes "lineup detection is just topic matching" (via topic-matched lineups + reason coding). If all alternatives are excluded and detection still works, voice/register mismatch is the remaining explanation. |
| 15 | Meehl | With a 6-way forced choice, more data makes the test harder only if the model's true accuracy is close to chance. If the model is genuinely detecting outliers, more trials increases confidence. The prediction is specific: Condition 1 at chance, Condition 2 well above chance, Conditions 3-4 somewhere between. More data sharpens these estimates. The prediction is directional but the ordering (C1 < C4 < C3 < C2) is specific enough to fail. |
| 16 | Feynman | **Most likely artifact:** the model recognizes its own output in Conditions 2-3 ("this sounds like me") rather than detecting voice mismatch with LW. This would still produce high detection rates but wouldn't generalize to human-written non-LW posts. Condition 4 (user's actual writing) is the test: if the model detects it at the same rate as raw AI, it's detecting "not AI" rather than "not LW." If it detects it at a lower rate, voice is a real signal. |
| 17 | Pearl | No causal claim. The claim is: "lineup detection rate predicts LW reception" — a correlational hypothesis. We don't intervene on LW moderation. |
| 18 | Ioannidis | 30 trials per condition, 4 conditions. Power: at n=30, binomial test detects true accuracy >= 40% with 80% power (H0: p=1/6). If the true effect is smaller (e.g., 25% accuracy), we're underpowered. Researcher degrees of freedom: limited by pre-registered metrics, no iterative refinement, all trials reported. Prior probability that lineup detection works: moderate — it works for PR descriptions, unknown for long-form prose. |
| 19 | Mayo | The test is moderately severe. Passing requires: (a) Condition 1 at chance AND (b) Condition 2 above chance AND (c) reasons coded as voice/register, not topic. A method that only detects topic mismatch would fail (a) or (c). A method that only detects "not in pool" would fail (a). A method that only detects AI output would pass (a-b) but provide no signal on Condition 4. |
| 20 | Gwern | **Full trail published.** All 120 trial files with prompts, responses, and scores. analysis.py with no manual overrides. PREREG committed before any trials run. No prompt iteration during the experiment. The repo is public. |
| 21 | Gwern | Predictions, timestamped now: (1) C1 held-out LW < 7/20. (2) C2 HN negative >= 14/20. (3) C3 raw AI >= 16/20. (4) C4 humanized AI >= 10/20. (5) C5 user posts 5-12/20. (6) C6 perturbations: topic-match drops detection by >= 4 trials vs C5, structure-match by >= 2 more, style-match by >= 2 more. Ordering: topic > structure > style in effect size. Scored after all trials complete. |
| 22 | Ramdas | No sequential testing. All 120 trials run before any analysis. No peeking, no sample expansion. If 20 trials per condition proves underpowered, we note it in RESULTS.md and pre-register a follow-up with more trials rather than expanding mid-experiment. |

## Timeline

1. Collect positive samples (scrape 30 LW posts)
2. Prepare candidates (4 conditions)
3. Run 120 trials
4. Score and analyze
5. Write RESULTS.md
