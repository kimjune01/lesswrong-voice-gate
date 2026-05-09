# Hypothesis Graph: LessWrong Voice Gate

## Research question

Can a paragraph-level inductive lineup detect community membership in LessWrong posts, and which confound drives detection?

## Nodes

### H0 — Null
**Status:** PENDING
**Claim:** GPT-5.5 identifies the candidate paragraph at chance rate (1/6).
**Kill condition:** Per-condition E_post evaluated individually. H0 is not killed globally — each condition's evidence is interpreted separately.
**Edge if killed per condition:** → evaluate H_topic, H_structure, H_style perturbation deltas

### H_topic — Topic confound
**Status:** PENDING
**Claim:** Detection is driven by topic mismatch between candidate and LW lineup.
**Perturbation:** Reframe C5 post through LW-native lens (C6a). Keep argument and structure, change framing.
**Kill condition:** C6a red fraction does NOT drop by >= 15pp vs C5.
**Evidence trajectory:** If C6a drops sharply → SUPPORTED. If no drop → KILLED.
**Edge if killed:** → H_structure

### H_structure — Format/structural confound
**Status:** PENDING
**Claim:** Detection is driven by format mismatch (paragraph density, heading patterns, footnotes, epistemic status markers).
**Perturbation:** Reformat C6a into LW structural conventions (C6b). Same argument, same framing, different format.
**Kill condition:** C6b red fraction does NOT drop by >= 10pp vs C6a.
**Evidence trajectory:** If C6b drops → SUPPORTED. If no drop → KILLED.
**Edge if killed:** → H_style

### H_style — Prose register confound
**Status:** PENDING
**Claim:** Detection is driven by vocabulary, hedging patterns, compression style distinct from LW register.
**Perturbation:** Rewrite C6b in LW prose register (C6c). Same argument, same framing, same structure, different voice.
**Kill condition:** C6c red fraction does NOT drop by >= 10pp vs C6b.
**Evidence trajectory:** If C6c drops → SUPPORTED. If no drop → KILLED.
**Edge if killed:** → H_residual

### H_residual — Residual distributional mismatch
**Status:** PENDING
**Claim:** Detection persists after controlling for topic, structure, and style. The residual may be conceptual framing, assumed background knowledge, implicit references, local argumentative norms, or domain substance.
**Perturbation:** None available — this is the terminal node.
**Kill condition:** C6c e-value ≈ 1 (all confounds were sufficient, no residual).
**Evidence trajectory:** If C6c e-value still strong → SUPPORTED. If near 1 → KILLED.

## Graph structure

H_topic, H_structure, H_style are contribution hypotheses evaluated by marginal perturbation delta along a fixed rewrite path. The path order is pragmatic, not a causal graph. All three may contribute simultaneously.

```
Perturbation path (fixed order):

C5a (raw) → C6a (topic reframed) → C6b (+ structure matched) → C6c (+ style matched)

Deltas measured:
  C5a → C6a : topic/framing delta
  C6a → C6b : structure delta
  C6b → C6c : style delta
  C6c residual : remaining detection

Per delta:
  reduced detection by >= preregistered threshold → that factor contributes
  did not reduce detection → that factor does not contribute on this path
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
