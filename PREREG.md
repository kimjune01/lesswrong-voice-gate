# Pre-Registration: LessWrong Voice Gate via Inductive Lineup

## Scope

This prereg covers **Stage 1: Discovery** — what drives detection in a LW paragraph lineup? The output is a ranked set of hypotheses about what makes LW prose distributionally distinct, not a confirmed answer.

**Stage 2: Confirmation** (separate prereg, written after Stage 1) takes the top candidate hypothesis from Stage 1 and tests it with fixed sample size and p-values.

**Stage 3: Operationalization** (separate prereg) builds a skill from the confirmed signal and validates it against held-out data.

## Research question

Can a frontier model distinguish candidate paragraphs from high-karma LessWrong paragraphs in a blind lineup, using only positive samples? If so, what drives the distinction?

## Hypotheses

**H0 (null):** GPT-5.5 identifies the candidate paragraph as the outlier at chance rate (1/6).

Four confound hypotheses explain *why* the model detects the outlier, ordered by expected effect size:

**H_topic:** Detection is driven by topic mismatch. Perturbation: reframe the candidate through a LW-native lens. If detection drops, topic was the signal.

**H_structure:** Detection is driven by format/structural mismatch. LW posts use specific patterns: footnotes, epistemic status headers, inline caveats, nested parenthetical asides, section-less flowing argument. Perturbation: reformat without changing the argument. If detection drops, structure was the signal.

**H_style:** Detection is driven by prose register mismatch. LW has a specific vocabulary, hedging pattern, and compression style distinct from HN/personal blogs. Perturbation: rewrite in LW register without changing topic or structure. If detection drops, style was the signal.

**H_residual:** Detection persists after controlling for topic, structure, and style. The residual may be conceptual framing, assumed background knowledge, implicit references, local argumentative norms, or domain substance. Defined operationally: residual reasons cite something other than surface prose.

The hypothesis graph is: H_topic → H_structure → H_style → H_residual. Each perturbation estimates whether removing one salient mismatch reduces detection, but the interventions are not guaranteed to isolate independent causal factors (topic reframing may change style; structure carries epistemic stance).

## Why induction, not rubric

Prior work ([slop-detection](/slop-detection)) showed that explicit rubrics become exploit surfaces under adversarial feedback. Two iterations collapsed the score gap. A rubric-free lineup forces the model to induce the distribution from examples, making the detection criteria implicit and harder to game.

## Design

### Positive sample collection

- **Source:** LessWrong frontpage posts with karma >= 100, published 2024-2026
- **Sample size:** 30 posts, stratified by topic cluster:
  - AI alignment / safety (10)
  - Rationality / epistemics (10)
  - Applied / technical / field reports (10)
- **Exclusion:** posts by Eliezer, Scott Alexander, Zvi
- **Format:** markdown body, stripped of metadata, author, karma
- **Status:** collected via LessWrong GraphQL API (`data/positive-samples/`)

### Paragraph-level lineup

The unit of measurement is a paragraph, not a whole post. The paragraph lineup measures per-paragraph and derives the post verdict from the red fraction.

For each candidate paragraph:
1. Classify its **section role**: opening, argument, evidence, transition, conclusion
2. Draw 5 paragraphs from LW positive samples matched by section role
3. Shuffle into random order with neutral labels (Paragraph A through F)
4. Prompt: "Here are 6 paragraphs from blog posts in the same genre. Which one is least like the others? Name it and explain why."
   - Forced choice — no abstention. This fixes the null at exactly 1/6.
   - If the model names multiple: take the first named.
   - If it picks a LW paragraph: scored as miss (e = 1).
   - Reason codes extracted from the explanation after scoring correctness.
5. Repeat 3 times with different LW draws per paragraph

**Post-level verdict** is derived: if >= 30% of a post's paragraphs are red (detected 2+ times out of 3), the post fails the gate. The threshold is pre-registered, not discovered from data.

**Per-post heatmap output:**
- **Green (0-1/3 detected):** paragraph passes
- **Yellow (2/3 detected):** borderline
- **Red (3/3 detected):** paragraph fails

With reason codes per detection: multi-label from {topic, structure, style, conceptual_framing, reference_density, argument_quality, length, specificity, other}. Not forced into the hypothesis graph — open coding reveals artifacts the graph would hide.

### Topic gate (pre-lineup)

Before voice testing, run a topic fitness check:

1. Extract the candidate's central question in one sentence
2. Present it alongside 10 real LessWrong post titles+abstracts (high karma, mixed topics)
3. Prompt: "A user is deciding where to post this. Does it belong in this collection, or somewhere else? If somewhere else, where?"

A candidate routed elsewhere doesn't enter the lineup. The topic gate also reports reframing suggestions.

### Candidate conditions

- **C1 (calibration — should pass):** held-out LessWrong posts
- **C2 (HN negative):** high-scoring Hacker News blog posts. **Assumption:** HN posts are distributionally distinct from LW posts. We do not have LW rejection data for these posts. Source: `data/negative-samples-hn/`
- **C3 (raw AI — should fail):** Claude-generated posts on LW-adjacent topics
- **C4 (humanized AI):** Claude posts after copyedit pipeline (humanize → tighten → sharpen)
- **C5a (target raw):** june.kim blog posts as originally written
- **C5b (target reframed):** same posts after topic gate reframing (if the gate suggests a reframe)
- **C5c (known rejection):** the union-find compaction post submitted to LessWrong and rejected (commit 3b269c0). Rejection feedback: "LessWrong has an unusually high bar for first-time posters." One data point, ambiguous reason — we cannot infer why it was rejected. The lineup result is compared against the real outcome, not interpreted as confirming a cause.
- **C6 (perturbation):** C5 posts rewritten to probe one confound at a time (topic-matched → structure-matched → style-matched). Each perturbation targets one hypothesis edge but may affect others (see failure modes: perturbation entanglement).

### Trial count

Per candidate post: P paragraphs × 3 repetitions. A typical post has ~15 paragraphs = ~45 trials.

Per condition: one representative post, ~45 trials.

Total: 6 conditions × ~45 trials = ~270 trials.

### Statistical plan

**E-values for discovery.** This is Stage 1 — hypothesis discovery, not confirmation. E-values (Ramdas 2023) are the right tool: anytime-valid, compose across conditions, allow early stopping on dead branches. The output is a ranked set of candidate hypotheses, not a confirmed result. Confirmation (Stage 2) uses a separate prereg with fixed sample size and p-values on the top candidate from this stage.

**Per-trial e-value:** Likelihood ratio bet with fixed alternative q = 0.5 (model picks candidate with probability 0.5 under alternative).
- Null: candidate selected with probability 1/6
- If correct: e = q / (1/6) = 3
- If incorrect: e = (1-q) / (5/6) = 0.6
- Null expectation: (1/6)(3) + (5/6)(0.6) = 1.0 ✓

Betting rule is fixed before trials and does not adapt to observed outcomes.

**Per-paragraph e-value:** Product across 3 repetitions. E_paragraph = e₁ × e₂ × e₃.
- 3/3 detected: E = 27
- 2/3 detected: E = 5.4
- 1/3 detected: E = 1.08
- 0/3 detected: E = 0.216

Threshold for "red": E_paragraph >= 5.4 (detected 2+/3). Heatmap thresholds are separate from evidence thresholds — a paragraph can be "red" for revision purposes without constituting strong statistical evidence on its own.

**Per-post e-value:** Product of per-paragraph e-values. Used as an evidence-ranking score. Because paragraph trials are clustered by post and repeated over the same candidate paragraph, post-level products are interpreted descriptively in Stage 1, not as calibrated Type-I-error guarantees. Report paragraph count alongside E_post so magnitude is interpretable.

**Decision threshold:** E >= 20 is "strong evidence" against the null. E >= 100 is "very strong." These thresholds are pre-registered.

**Also report:** red fraction (detected paragraphs / total paragraphs) and full detection-rate distributions per condition. E-values are the inferential tool; red fractions and heatmaps are the descriptive output.

**Secondary metrics:**
- Reason codes mapped to confound hypotheses (multi-label, not forced into the hypothesis graph — codex review caught this)
- C1 e-values and red fraction (defines the empirical false-positive baseline — not expected to equal chance exactly, since held-out LW paragraphs may differ from pool LW paragraphs in author, topic, or length)
- C2 vs C3: do HN posts and raw AI produce different reason-code distributions?
- C6 perturbation trajectory: does each perturbation reduce the per-post e-value?

### Interpretable outcomes

Stage 1 does not "pass" or "fail." It produces a discovery pattern. Outcomes ranked by informativeness:

1. C1 establishes the empirical false-positive baseline. Large C1 values constrain interpretation of all other conditions but do not automatically kill the method.
2. C2 and C3 e-values characterize detection on these specific HN and AI posts, not HN or AI generally.
3. C5/C6 heatmaps show which paragraphs and which reason codes dominate.
4. Perturbation deltas (C5a → C6a → C6b → C6c) rank which rewrite had the largest effect on detection.
5. The top-ranked hypothesis from this ranking becomes the candidate for Stage 2 confirmation.

Monotonic trajectory (each perturbation reduces detection) is a **prediction**, not a success criterion.

### Hypothesis ranking (post-experiment)

After trials complete, surviving hypotheses are ranked by:
- **Cost to fix:** estimated work to remove this mismatch signal
- **Surprise:** how far the result deviates from predictions

High-surprise, low-cost findings are acted on first.

### Failure modes

- **Topic confound:** model detects topic, not voice. Mitigated by section-role matching and topic gate.
- **Length confound:** paragraphs vary in length. Mitigated by min 2 sentences per paragraph, length-aware draw from LW pool.
- **Memorization:** model recognizes LW posts from training. Mitigated by author exclusion + C1 false positive check. Note: C1 passing doesn't fully rule out memorization — recognized LW paragraphs would look in-distribution regardless.
- **HN assumption:** C2 assumes HN and LW are distributionally distinct. No LW rejection data for these specific posts.
- **Perturbation entanglement:** topic reframing may change style; structure carries epistemic stance. The confound chain is not a clean causal decomposition.
- **Reason code circularity:** the detector explains its own detection. Weak as measurement. Multi-label open coding partially mitigates.

## Models

- **Primary:** GPT-5.5 via codex (continuity with slop-detection)
- **Secondary:** Gemini 3.1 Pro (independent model family, cross-validation)
- Disagreement between models is reported; interpretation deferred to RESULTS.md.

## Outputs

- `data/positive-samples/` — 30 LW posts (collected)
- `data/negative-samples-hn/` — 15 HN posts (collected)
- `data/candidates/` — posts per condition
- `trials/` — one file per trial with lineup, prompt, response, scored result
- `analysis.py` — e-value computation, reason coding, heatmap generation, detection-rate distributions
- `RESULTS.md` — written after all trials complete, not before

## Operational specifications

### Paragraph segmentation
- Split on double newline (markdown paragraph break)
- Merge consecutive paragraphs under 2 sentences into one unit
- Exclude headings, bullet lists, blockquotes, code blocks, and footnotes as standalone paragraphs (they may be included as part of an adjacent paragraph)
- Maximum paragraph length: 300 words. Longer paragraphs split at the nearest sentence boundary after 200 words.

### Lineup sampling constraints
- No two LW paragraphs from the same source post in one lineup
- Length-match by token count within ±50%, falling back to ±100% if insufficient matches
- Section-role classification by GPT-5.5 with a fixed prompt (frozen in `prompts/section-role.txt` before trials)
- Candidate paragraph never repeated across the 3 repetitions with the same LW draw

### Model settings (frozen before trials)
- Primary: GPT-5.5 via codex. Record exact model identifier at trial time.
- Temperature: 0 (deterministic)
- System prompt: none
- Lineup prompt: frozen in `prompts/lineup.txt`
- Reason-code extraction: automated, blind to correctness label

### Candidate freeze
- Exact files/commits for each condition specified in `data/candidates/MANIFEST.md` before trials begin

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
| 6 | Mill | Each condition probes a different contrast: C1 = real LW (baseline), C2 = HN (community), C3 = raw AI (generation), C4 = humanized AI (pipeline), C5 = user writing (target), C6 = perturbations (confound). Lineup composition is randomized within conditions. |
| 7 | Mill | Control is C1: held-out LW paragraphs that should NOT be detected. Isolates "outlier detection" from "detecting anything not in the pool." |
| 8 | Chamberlin | Competing explanations for detection: (a) topic mismatch — mitigated by section-role matching; (b) memorization — mitigated by author exclusion + C1; (c) AI self-recognition — tested by C2 vs C3 comparison; (d) random luck — addressed by e-value accumulation. |
| 9 | Peirce | The hypothesis (lineup detection works) comes from the PR-description crosscheck in the drip pipeline. The LW application is a new domain. Not inferred from this data. |
| 10 | Fisher | Lineup position randomized (shuffled labels A-F). Positive sample draws randomized. Section-role matching is systematic but pre-registered. |
| 11 | Popper | **Falsification:** C1 e-value >> 1 → method broken (detecting "not in pool"). C3 e-value ≈ 1 → method has no power. Either kills the approach. |
| 12 | Popper | The bar is informative: e-value threshold 20 (strong) or 100 (very strong). Forced 6-way choice, null = 1/6. E-values compose across conditions without correction. |
| 13 | Kuhn | **Invisible assumption:** "high karma = representative LW post." Karma correlates with agreement and timing, not just quality. May exclude contrarian posts. "LW voice" may not be a single distribution — stratification partially addresses. |
| 14 | Platt | Excludes memorization (via C1), topic matching (via section-role + reason coding), AI self-recognition (via C2 vs C3). If all are excluded and detection persists, distributional mismatch is the remaining explanation — but "remaining" is not "proven." |
| 15 | Meehl | Predictions are specific and ordered: C1 < C5 < C4 < C2 < C3 in red fraction. More data sharpens the ordering. The prediction can fail if the ordering is wrong. |
| 16 | Feynman | **Most likely artifact:** the model recognizes its own output in C3-C4 rather than detecting LW voice mismatch. C5 (human-written, not AI) is the test: if detected at the same rate as C3, the method detects "not AI," not "not LW." If lower, voice is a real signal. |
| 17 | Pearl | No causal claim. "Lineup detection rate predicts LW reception" is correlational. No intervention on LW moderation. |
| 18 | Ioannidis | ~270 trials across conditions. E-values compose by multiplication — no Bonferroni needed, no family-wise error inflation. Researcher degrees of freedom limited by pre-registered metrics, fixed conditions, no iterative refinement. Confound hierarchy ordered a priori, not post-hoc. One post per condition is a pilot, not a general estimate — claims are about these specific posts, not the conditions as categories. |
| 19 | Mayo | Stage 1 does not have a single pass/fail gate. Severity is evaluated per-condition: C1 calibrates the baseline, C2/C3 test sensitivity on specific posts, C6 perturbation deltas rank confound contributions. If reason codes map to topic instead of register, that is a discovery, not a failure. |
| 20 | Gwern | **Full trail published.** All trial files with prompts, responses, scores. analysis.py with no manual overrides. PREREG committed before any trials. No prompt iteration during experiment. Repo is public. |
| 21 | Gwern | **Predictions, timestamped now** (as per-post e-values): C1 e ≈ 1. C2 e >= 100. C3 e >= 1000. C4 e >= 20. C5a e 5-100. C5b e < C5a. C6: each perturbation reduces e-value. Ordering of effect: topic > structure > style. |
| 22 | Ramdas | E-values avoid p-value multiple-testing corrections for optional stopping, but Stage 1 rankings remain exploratory. Post-level products are descriptive scores, not calibrated guarantees, because paragraph trials are clustered by post. |
| 23 | Storytelling | Every claim in the prereg must trace to a design choice, a prior result, or a stated assumption — not to an expected outcome. The hypothesis graph is the exception: it is a narrative with a spine (ordered predictions, kill conditions, edges). Narrative belongs there, not in the design or statistical plan. |

## Timeline

### Stage 1: Discovery (this prereg)
1. Collect positive samples — done (30 LW posts)
2. Collect negative samples — done (15 HN posts)
3. Prepare candidates (conditions C1–C6)
4. Segment all posts into paragraphs, classify section roles
5. Run paragraph-level lineup trials, accumulate e-values
6. Score, generate heatmaps, rank hypotheses by effect size
7. Write RESULTS.md with top candidate hypothesis

### Stage 2: Confirmation (separate prereg, after Stage 1)
8. Pre-register the top hypothesis from Stage 1
9. Fix sample size, define test statistic, commit to p-value threshold
10. Collect fresh samples (not reused from Stage 1)
11. Run confirmatory test
12. Report result

### Stage 3: Operationalization (separate prereg, after Stage 2)
13. Build a skill from the confirmed signal
14. Define heuristics and calibration samples
15. Validate against held-out data
16. Evaluate: does the skill improve LW acceptance?
