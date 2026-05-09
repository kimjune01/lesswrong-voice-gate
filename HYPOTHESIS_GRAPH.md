# Hypothesis Graph: LessWrong Voice Gate

## Research question

Can a paragraph-level inductive lineup detect community membership in LessWrong posts, and which confound drives detection?

## Nodes

### H0 — Null
**Status:** PENDING
**Claim:** GPT-5.5 identifies the candidate paragraph at chance rate (1/6).
**Kill condition:** Any condition shows red fraction significantly above chance (Bonferroni alpha=0.0083).
**Edge if killed:** → fan out to H_topic, H_structure, H_style

### H_topic — Topic confound
**Status:** PENDING
**Claim:** Detection is driven by topic mismatch between candidate and LW lineup.
**Perturbation:** Reframe C5 post through LW-native lens (C6a). Keep argument and structure, change framing.
**Kill condition:** C6a red fraction does NOT drop by >= 15pp vs C5.
**Evidence trajectory:** If C6a drops sharply → CONFIRMED (topic was the signal). If no drop → KILLED (topic isn't the driver).
**Edge if killed:** → H_structure

### H_structure — Format/structural confound
**Status:** PENDING
**Claim:** Detection is driven by format mismatch (paragraph density, heading patterns, footnotes, epistemic status markers).
**Perturbation:** Reformat C6a into LW structural conventions (C6b). Same argument, same framing, different format.
**Kill condition:** C6b red fraction does NOT drop by >= 10pp vs C6a.
**Evidence trajectory:** If C6b drops → CONFIRMED (structure was the residual signal). If no drop → KILLED.
**Edge if killed:** → H_style

### H_style — Prose register confound
**Status:** PENDING
**Claim:** Detection is driven by vocabulary, hedging patterns, compression style distinct from LW register.
**Perturbation:** Rewrite C6b in LW prose register (C6c). Same argument, same framing, same structure, different voice.
**Kill condition:** C6c red fraction does NOT drop by >= 10pp vs C6b.
**Evidence trajectory:** If C6c drops → CONFIRMED (style was the residual signal). If no drop → KILLED.
**Edge if killed:** → H_voice

### H_residual — Residual distributional mismatch
**Status:** PENDING
**Claim:** Detection persists after controlling for topic, structure, and style. The residual may be conceptual framing, assumed background knowledge, implicit references, local argumentative norms, or domain substance.
**Perturbation:** None available — this is the terminal node.
**Kill condition:** C6c e-value ≈ 1 (all confounds were sufficient, no residual).
**Evidence trajectory:** If C6c e-value still strong → CONFIRMED (residual exists). If near 1 → KILLED (confounds explained everything).

## Graph structure

```
H0 (null)
 │
 │ [any condition above chance]
 ▼
H_topic ──[C6a drops >=15pp]──► CONFIRMED (fix: reframe topic)
 │
 │ [C6a doesn't drop]
 ▼
H_structure ──[C6b drops >=10pp]──► CONFIRMED (fix: reformat)
 │
 │ [C6b doesn't drop]
 ▼
H_style ──[C6c drops >=10pp]──► CONFIRMED (fix: rewrite register)
 │
 │ [C6c doesn't drop]
 ▼
H_residual ──[C6c e-value strong]──► CONFIRMED (residual mismatch exists)
           ──[C6c e-value ≈ 1]──► KILLED (confounds explained everything)
```

## Cross-references

- **slop-detection:** established that rubric-based detection is a wasting asset under feedback. This experiment uses lineup (rubric-free) specifically because of that finding. NOT retesting AI detection.
- **drip pipeline codex crosscheck:** the lineup technique originates from PR description testing. This experiment tests whether it transfers to long-form prose paragraphs.
- **Peirce (prereg Q9):** the hypothesis (lineup detection works) comes from the PR crosscheck, not from this data. The domain transfer is the novel claim.

## Predictions (timestamped 2026-05-09)

| Node | Prediction | Scored when |
|------|-----------|-------------|
| H0 | Killed (C3 raw AI will be well above chance) | After all trials |
| H_topic | Killed or weakly confirmed (topic matters but isn't dominant) | After C6a trials |
| H_structure | Weakly confirmed (LW has distinctive structure) | After C6b trials |
| H_style | Confirmed (LW register is the strongest confound) | After C6c trials |
| H_residual | Borderline — expect C6c e-value weak but >1 | After C6c trials |
