# Work Log: LessWrong Voice Gate

## 2026-05-09T16:00 — Project inception

**Origin:** pallets/click, pallets/jinja, pallets/quart — three PRs batch-closed by davidism within 21 seconds. We were investigating the org-level pacing failure when the conversation shifted: if the drip pipeline's codex crosscheck can detect AI-likeness in PR descriptions via lineup, can the same technique detect community mismatch in long-form prose?

The question was prompted by thinking about what "passing" means at different levels. Passing a PR review is mostly mechanical (correct code, right tone). Passing LessWrong moderation is epistemic — the reader asks whether this post adds something they didn't already have. That's a harder gate, and if we can clear it programmatically, we can clear almost anything.

**The codex detour:** we asked codex (GPT-5.5) about the LessWrong strategy. It produced an elaborate rubric — six claim classifications, falsification checks, risk flags. Solid work, but exactly the wrong approach. /slop-detection already proved that rubrics become exploit surfaces under adversarial feedback. Two iterations to convergence. Any rubric we build teaches the adversary what shape to optimize toward.

**The reframe:** June pointed out that the novelty isn't the problem — the arguments are genuinely novel. It's purely style. But "style" is actually three things conflated: topic framing, structural conventions, and prose register. We need to separate them.

**Why lineup over rubric:** a lineup forces the model to induce the distribution from examples. The detection criteria stay implicit — you can't optimize against what isn't named. The PR crosscheck already works this way. The question is whether it transfers from PR descriptions (short, formulaic) to blog posts (long, varied).

## 2026-05-09T16:15 — Pre-registration drafted

Initial design: 5 LW posts + 1 candidate in a 6-entry post-level lineup. 4 conditions, 30 trials each. The key design choice was using only positive samples — no "bad" examples, no definition of what LW voice *isn't*. Induction from positive examples only. The model learns what "belongs" looks like and flags what doesn't.

## 2026-05-09T16:30 — Prereg checklist (22 questions)

Walked the prereg through all 22 questions from /the-prereg-checklist (Bacon → Gwern). Key findings:

- **Feynman (#16):** most likely artifact is the model recognizing its own output, not detecting LW voice mismatch. Condition with user's actual (human-written) posts is the discriminator.
- **Ioannidis (#18):** 20 trials × 4 conditions underpowered with multiple comparisons.

## 2026-05-09T16:45 — Topic gate added

June pointed out that topics need to be filtered through LW's lens too. A perfectly voiced post on a topic they don't care about still gets buried. This is obvious in retrospect but the initial design assumed topic was controlled by the experimenter — it isn't. The audience controls what's topical.

Added a pre-lineup topic fitness check using the same inductive method: present the candidate alongside 10 real LW post titles, ask where it belongs. The insight is that **topic translation** is often possible — same substance, different framing. "Bot-magnet issues on GitHub" routes to HN. "Adversarial dynamics in open-source contribution markets" routes to LW. The topic gate should report the reframing, not just the verdict.

**Peirce and abduction as LW candidate:** LW has built an entire epistemology around Bayesian updating — P(H|E) — without asking where H comes from. Peirce's abduction is the missing piece: hypothesis *generation* is a different operation from hypothesis *testing*. The sweep pipeline is a concrete implementation of abduction (surprise → hypothesis → perturbation → classify trajectory). This is the kind of "new handle for a real pattern" that LW rewards — a named gap in their existing framework, filled with a working system.

## 2026-05-09T17:00 — Confound hypothesis graph

The single H0/H1 pair was too coarse. If the model detects the outlier, *why* matters more than *that*. We decomposed "detection" into four confound hypotheses: H_topic → H_structure → H_style → H_voice. This is the same abduction structure as /investigate — a hypothesis graph where each perturbation kills or confirms one edge, and you follow the surviving path.

The ordering is a prediction: topic is the strongest confound (easiest to fix), voice is the weakest (hardest, maybe impossible). If the ordering is wrong, that's informative — it means we misunderstand what makes LW prose distinctive.

**HN negatives as a discriminator:** June said "we have easily rejectable samples we can scrape from HN." This created Condition 2. The critical comparison is C2 (HN) vs C3 (raw AI). If the model detects both at the same rate, it's detecting "not LW." If it detects raw AI higher than HN, it's partially detecting AI-ness rather than community mismatch. The reason codes separate these: HN posts should fail on topic/structure, AI posts should fail on style/voice.

## 2026-05-09T17:15 — Data collection

- **30 LW positive samples** collected via GraphQL API (`documentId` selector, not `slug`). Karma >= 100, 2024-2026. Excluded Eliezer, habryka, So8res, Scott Alexander, Zvi.
- **15 HN negative samples** fetched from best stories. Blog posts only (no GitHub, YouTube, Wikipedia, PDFs). Stripped HTML, trimmed to 2000 words.

## 2026-05-09T17:30 — Bonferroni correction

June caught the Ioannidis problem: "statistical power on that many confounds?" With 6 simultaneous comparisons, a positive somewhere is almost guaranteed by chance. Ran power analysis with scipy — 20 trials at Bonferroni alpha=0.0083 had only 75% power at 50% true detection. Not acceptable. Bumped to 30 trials per condition: power 0.95 at 50%. The alternative was fewer conditions, but the confound chain is the whole point — it's better to spend more trials than to lose the ability to distinguish topic from structure from style.

Effects below 40% are undetectable at this sample size. Acceptable tradeoff: a gate that only catches 30% of the time isn't reliable enough to use anyway.

## 2026-05-09T17:45 — Paragraph-level lineup replaces post-level

The sweep pipeline already does this for PR descriptions — the codex crosscheck shuffles one candidate into 5 real PR descriptions. But PRs are atomic. Blog posts have internal structure: an opening that sets the frame, argument paragraphs that build on each other, evidence sections, conclusions. A post-level lineup says "this post doesn't belong" but doesn't say which part.

Paragraph-level lineup decomposes the signal. Section-role matching (opening among openings, argument among arguments) prevents the trivial confound of detecting positional role rather than voice. The output is a heatmap — green/yellow/red per paragraph with reason codes. You revise the red ones, rerun, iterate.

**Disposing of the weaker technique:** June said "we dispose of strictly weaker techniques." Post-level lineup is strictly weaker — same information, lower resolution. The post verdict is derived from paragraph results (>= 30% red = fail). No need to run both. One technique, one measurement, one statistical test.

## 2026-05-09T18:00 — Clean prereg

Five rounds of edits had left the prereg internally inconsistent — Phase 2 was redundant with the main design, checklist answers referenced stale trial counts, condition numbering was off. Consolidated into one coherent document. The final shape: one technique (paragraph-level lineup), 6 conditions, 4 confound hypotheses ordered a priori, ~270 trials, Bonferroni-corrected.

**The meta-observation:** this prereg is itself an abduction. We hypothesized that lineup detection works for community membership, decomposed "detection" into competing explanations, and designed perturbations to kill each one. The experiment structure mirrors the sweep pipeline: hypothesis graph → perturbation → classify trajectory → follow surviving edge. If we submit this to LessWrong, the experiment design *is* the Peirce/abduction argument — a concrete instance of the framework we're proposing.

**Status:** prereg ready for codex volley. Positive samples collected (30 LW). Negative samples collected (15 HN). Candidates for conditions 3-6 not yet prepared.
