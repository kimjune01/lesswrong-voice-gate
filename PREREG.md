# Pre-Registration: LessWrong Voice Gate via Inductive Lineup

## Research question

Can a frontier model distinguish candidate paragraphs from high-karma LessWrong paragraphs in a blind lineup, using only positive samples — no rubric, no "AI slop" definition?

## Hypotheses

**H0 (null):** GPT-5.5 identifies the candidate paragraph as the outlier at chance rate (1/6).

Four confound hypotheses explain *why* the model detects the outlier, ordered by expected effect size:

**H_topic:** Detection is driven by topic mismatch. The candidate writes about GitHub contribution pipelines; the lineup writes about AI alignment. Perturbation: reframe the candidate through a LW-native lens. If detection drops, topic was the signal.

**H_structure:** Detection is driven by format/structural mismatch. LW posts use specific patterns: footnotes, epistemic status headers, inline caveats, nested parenthetical asides, section-less flowing argument. Perturbation: reformat without changing the argument. If detection drops, structure was the signal.

**H_style:** Detection is driven by prose register mismatch. LW has a specific vocabulary, hedging pattern, and compression style distinct from HN/personal blogs. Perturbation: rewrite in LW register without changing topic or structure. If detection drops, style was the signal.

**H_voice:** Detection persists after controlling for topic, structure, and style — the candidate lacks something irreducible about the author's relationship to the LW intellectual community. This is the hard signal.

The hypothesis graph is: H_topic → H_structure → H_style → H_voice. Each perturbation kills or confirms one edge.

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
- **Format:** markdown body, stripped of metadata, author, karma
- **Status:** collected via LessWrong GraphQL API (`data/positive-samples/`)

### Paragraph-level lineup

The unit of measurement is a paragraph, not a whole post. Post-level lineups are strictly weaker: they detect but don't localize. The paragraph lineup measures per-paragraph, derives the post verdict from the red fraction.

For each candidate paragraph:
1. Classify its **section role**: opening, argument, evidence, transition, conclusion
2. Draw 5 paragraphs from LW positive samples matched by section role
3. Shuffle into random order with neutral labels (Paragraph A through F)
4. Prompt: "Here are 6 paragraphs from blog posts in the same genre. One may not match the others. Which one, and why? If they all match, say so."
5. Repeat 3 times with different LW draws per paragraph

**Post-level verdict** is derived: if >= 30% of a post's paragraphs are red (detected 2+ times out of 3), the post fails the gate. The threshold is pre-registered, not discovered from data.

**Per-post heatmap output:**
- **Green (0-1/3 detected):** paragraph passes
- **Yellow (2/3 detected):** borderline
- **Red (3/3 detected):** paragraph fails

With reason codes per detection: {topic, structure, style, voice}.

### Topic gate (pre-lineup)

Before voice testing, run a topic fitness check:

1. Extract the candidate's central question in one sentence
2. Present it alongside 10 real LessWrong post titles+abstracts (high karma, mixed topics)
3. Prompt: "A user is deciding where to post this. Does it belong in this collection, or somewhere else? If somewhere else, where?"

A candidate routed elsewhere doesn't enter the lineup. The topic gate also reports reframing suggestions: "Bot-magnet issues on GitHub" is an HN post; "Adversarial dynamics in open-source contribution markets" is a LW post. Same content, different lens.

### Candidate conditions

- **C1 (calibration — should pass):** held-out LessWrong posts
- **C2 (HN negative — should fail):** high-scoring Hacker News blog posts. Same internet, different community. Tests community detection vs quality detection. Source: `data/negative-samples-hn/`
- **C3 (raw AI — should fail):** Claude-generated posts on LW-adjacent topics
- **C4 (humanized AI):** Claude posts after copyedit pipeline (humanize → tighten → sharpen)
- **C5 (target):** june.kim blog posts, reframed through topic gate if needed
- **C6 (perturbation):** C5 posts rewritten to control for one confound at a time (topic-matched → structure-matched → style-matched). Each perturbation isolates one hypothesis edge.

### Trial count

Per candidate post: P paragraphs × 3 repetitions. A typical post has ~15 paragraphs = ~45 trials.

Per condition: one representative post, ~45 trials.

Total: 6 conditions × ~45 trials = ~270 trials.

### Statistical plan

**Primary metric:** red paragraph fraction per condition.

- Chance baseline: 1/6 ≈ 16.7% of paragraphs detected by chance
- Per-paragraph: red if identified as outlier in >= 2/3 repetitions
- Per-post: red fraction = red paragraphs / total paragraphs
- Per-condition: Bonferroni-corrected alpha = 0.05/6 = 0.0083. Test whether the red fraction is significantly above the null rate (one-sided binomial on paragraph counts)

**Power:** For a 15-paragraph post, detecting a true red rate of 40% vs null 16.7% has >90% power. Bonferroni across 6 conditions: still well-powered.

**Secondary metrics:**
- Reason codes mapped to confound hypotheses
- C1 false positive rate (should be ≈ chance)
- C2 vs C3: community mismatch vs AI-ness — do HN posts and raw AI fail for different reasons?
- C6 perturbation trajectory: which confound removal has the largest effect?

### Success criteria

1. C1 red fraction ≈ chance (false positive check)
2. C2 red fraction significantly above chance (detects community, not just quality)
3. C3 red fraction significantly above chance (baseline sensitivity)
4. C6 perturbations produce a monotonic trajectory: each confound removed reduces detection
5. C5 provides actionable signal: either passes (ready to submit) or the surviving confound names the fix

### Failure modes

- **Topic confound:** model detects topic, not voice. Mitigated by section-role matching and topic gate.
- **Length confound:** paragraphs vary in length. Mitigated by min 2 sentences per paragraph, length-aware draw from LW pool.
- **Memorization:** model recognizes LW posts from training. Mitigated by author exclusion + C1 false positive check.
- **Low power:** at 15 paragraphs with Bonferroni correction, power is >90% at 40% true red rate, ~50% at 30%. Effects below 30% are undetectable but also too weak to be a useful gate.

## Models

- **Primary:** GPT-5.5 via codex (continuity with slop-detection)
- **Secondary:** Gemini 3.1 Pro (independent model family, cross-validation)
- Disagreement between models suggests the detection signal is model-specific, not structural.

## Outputs

- `data/positive-samples/` — 30 LW posts (collected)
- `data/negative-samples-hn/` — 15 HN posts (collected)
- `data/candidates/` — posts per condition
- `trials/` — one file per trial with lineup, prompt, response, scored result
- `analysis.py` — binomial tests, reason coding, heatmap generation
- `RESULTS.md` — written after all trials complete, not before

## What we will NOT do

- No rubric in the prompt. The whole point is induction from examples.
- No iterative refinement during the experiment. Run all ~270 trials, analyze, then decide.
- No cherry-picking which trials to report. All trials, all results.
- No post-hoc metric invention. The metrics above are the metrics.

## Prereg checklist (22 questions)

| # | Source | Answer |
|---|--------|--------|
| 1 | Bacon | Positive samples stratified by topic cluster. Lineup draws randomized. Section-role matching prevents positional bias. |
| 2 | Bacon | Data collection fixed: 30 LW posts + 15 HN posts collected before trials. Candidate selection per condition fixed before trials. No adjustments after seeing results. |
| 3 | Descartes | **Critical assumption:** outlier detection is driven by voice/register, not topic or memorization. If false, the experiment measures topic fit or author recognition. C1 (calibration) tests this. |
| 4 | Hume | The mechanism: LW paragraphs share distributional properties that a model induces from examples. An outlier deviates from this induced distribution. The claim is correlational — "detected as outlier" correlates with "wouldn't survive on LW" — not causal. |
| 5 | Hume | Results apply to GPT-5.5's detection ability on English-language rationalist blog posts. No claim about other models, communities, languages, or human readers. |
| 6 | Mill | Each condition varies one thing: C1 = real LW (null), C2 = HN (community confound), C3 = raw AI (generation confound), C4 = humanized AI (pipeline test), C5 = user writing (target), C6 = perturbations (confound isolation). Lineup composition is randomized within conditions. |
| 7 | Mill | Control is C1: held-out LW paragraphs that should NOT be detected. Isolates "outlier detection" from "detecting anything not in the pool." |
| 8 | Chamberlin | Competing explanations for detection: (a) topic mismatch — mitigated by section-role matching; (b) memorization — mitigated by author exclusion + C1; (c) AI self-recognition — tested by C2 vs C3 comparison; (d) random luck — addressed by binomial test with Bonferroni. |
| 9 | Peirce | The hypothesis (lineup detection works) comes from the PR-description crosscheck in the drip pipeline. The LW application is a new domain. Not inferred from this data. |
| 10 | Fisher | Lineup position randomized (shuffled labels A-F). Positive sample draws randomized. Section-role matching is systematic but pre-registered. |
| 11 | Popper | **Falsification:** C1 red fraction significantly above chance → method broken (detecting "not in pool"). C3 red fraction NOT above chance → method has no power. Either kills the approach. |
| 12 | Popper | The bar is informative: Bonferroni-corrected alpha=0.0083 on paragraph-count binomial. Not testing "better than coin flip" — testing "better than 1/6 random guess on a 6-way forced choice." |
| 13 | Kuhn | **Invisible assumption:** "high karma = representative LW post." Karma correlates with agreement and timing, not just quality. May exclude contrarian posts. "LW voice" may not be a single distribution — stratification partially addresses. |
| 14 | Platt | Excludes memorization (via C1), topic matching (via section-role + reason coding), AI self-recognition (via C2 vs C3). If all are excluded and detection persists, voice/register mismatch is the remaining explanation. |
| 15 | Meehl | Predictions are specific and ordered: C1 < C5 < C4 < C2 < C3 in red fraction. More data sharpens the ordering. The prediction can fail if the ordering is wrong. |
| 16 | Feynman | **Most likely artifact:** the model recognizes its own output in C3-C4 rather than detecting LW voice mismatch. C5 (human-written, not AI) is the test: if detected at the same rate as C3, the method detects "not AI," not "not LW." If lower, voice is a real signal. |
| 17 | Pearl | No causal claim. "Lineup detection rate predicts LW reception" is correlational. No intervention on LW moderation. |
| 18 | Ioannidis | ~270 trials across 6 conditions. Bonferroni correction (alpha=0.0083) controls family-wise error. Power >90% at 40% true red rate for 15-paragraph posts. Researcher degrees of freedom limited by pre-registered metrics, fixed conditions, no iterative refinement. Confound hierarchy ordered a priori, not post-hoc. |
| 19 | Mayo | Passing requires: C1 at chance AND C3 above chance AND C2 above chance AND reason codes map to voice/register. A method that only detects topic, memorization, or AI-ness would fail at least one gate. |
| 20 | Gwern | **Full trail published.** All trial files with prompts, responses, scores. analysis.py with no manual overrides. PREREG committed before any trials. No prompt iteration during experiment. Repo is public. |
| 21 | Gwern | **Predictions, timestamped now** (as red paragraph fractions): C1 < 20%. C2 >= 60%. C3 >= 75%. C4 >= 40%. C5 20-50%. C6: topic-match drops by >= 15pp vs C5, structure-match by >= 10pp more, style-match by >= 10pp more. Ordering: topic > structure > style in effect size. |
| 22 | Ramdas | No sequential testing. All ~270 trials run before analysis. No peeking, no sample expansion. If underpowered, note in RESULTS.md and pre-register a follow-up. |

## Timeline

1. Collect positive samples — done (30 LW posts)
2. Collect negative samples — done (15 HN posts)
3. Prepare candidates (6 conditions)
4. Segment all posts into paragraphs, classify section roles
5. Run ~270 paragraph-level lineup trials
6. Score, generate heatmaps, run binomial tests
7. Write RESULTS.md
