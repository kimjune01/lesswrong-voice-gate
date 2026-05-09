# Research Program: Adversarial Evaluation at Scale

## Premise

Gatekeepers (maintainers, moderators, reviewers) face a scaling problem: evaluate submissions without reading every one, without rejecting good work, without being gamed. Agents make this harder — they produce volume that pattern-matches to quality without necessarily being quality.

Nobody is red-teaming against agents yet. Heuristic filters exist (account age, co-author tags). Adversarial traps that exploit agent decision-making do not.

## Completed experiments

### 1. slop-detection (2026-03)

Can a model detect AI-generated prose after humanization? Result: 30/31. Rubric-based scoring is a wasting asset — two adversarial iterations collapsed the score gap. The qualitative "is this slop?" question outperformed the 6-dimension rubric. The rubric was the exploit surface.

### 2. lesswrong-voice-gate (2026-05, Stage 1 in progress)

Can a model detect community membership via inductive lineup? Paragraph-level, positive-samples-only, no rubric. Decomposes detection into topic, structure, style, and residual confounds. E-values for discovery, p-values for confirmation in Stage 2.

## Open questions

### Agent-side (offense)

- **Honeypot detection.** A pristine issue with no history on a repo that has history is a tell. Real bugs have discussion, failed attempts, edge cases. What features distinguish honeypots from real issues?
- **Canary detection.** Dead code with a trivial test that exists only to catch automated submissions. How does an agent distinguish canary code from real code?
- **Contributor fingerprinting.** Consistent commit timing, PR description style, issue selection patterns. What fingerprint does the pipeline leave, and can it be varied?
- **Bot-magnet avoidance.** body-count catches the obvious case (N closed PRs). What about issues that are about to become bot magnets — newly filed, mechanical, well-labeled?

### Defender-side (defense)

- **Honeypot design.** What makes an effective honeypot? Trivial change, dead code, passing test suite, maintainer-filed. What bait maximizes agent submissions while minimizing false positives on human contributors?
- **Lineup as moderation tool.** The same lineup that tests whether a post belongs on LessWrong could be used by moderators to screen submissions. Positive samples from the community, candidate is the submission. Does this scale?
- **Body-count as intake filter.** Maintainers could run body-count on their own issues to identify which ones are attracting bot PRs. Close or restructure those issues.
- **Behavioral analysis.** Agent submissions have patterns: zero typos, consistent formatting, perfect test coverage, no questions asked. Can these be detected without rejecting careful human contributors?

### Meta

- **Attack-defense co-publication.** Every attack published becomes a defense. Every defense published becomes a target. Does this cycle converge (both sides improve until equilibrium) or diverge (arms race with no stable point)?
- **Trust vs merit equilibrium.** Communities that gate on trust shrink. Communities that gate on merit get flooded. What filtering regime survives in the long run? Does it depend on the cost of evaluation?
- **Curl precedent.** Curl shut down bug bounty intake after AI slop flood. But AI-assisted research found 50 real bugs in the same period. The median collapsed while the ceiling rose. Is this the general pattern?

## Tools built so far

| Tool | Location | Attack or defense? |
|------|----------|--------------------|
| body-count | `~/.sweep/bin/body-count` | Both — agents avoid honeypots, maintainers identify bot-magnet issues |
| org-gate | `~/.sweep/bin/org-gate` | Attack — avoid triggering org-level rejection |
| pr-status | `~/.sweep/bin/pr-status` | Both — agents check PR state, maintainers audit patterns |
| lineup (PR crosscheck) | `/drip` skill | Both — agents test PR voice, maintainers could screen submissions |
| lineup (paragraph) | this repo | Both — writers test community fit, moderators could screen posts |

## Shape of the program

Each experiment follows the same arc:
1. Build an attack (how does an agent pass a gate?)
2. Run it until it fails (what catches the agent?)
3. Build the detector (how would a defender catch this?)
4. Publish both (attack + defense become public tools)
5. Check if the detector survives adversarial feedback (slop-detection says: two iterations)

The prereg checklist (23 questions) applies to each experiment. The hypothesis graph structures each investigation. E-values for discovery, p-values for confirmation. Publication forces merit-based evaluation by making tools available to both sides.

## Dynamics

### Non-stationary channel

The maintainer's attention is the channel. Contributions are the signal. Slop is the noise. Shannon gives capacity for a fixed noise floor. But the noise is adversarial — it adapts to the filter. Every improvement in filtering lowers the bar for submission, which increases volume, which demands better filtering. This is not classical information theory. It's a coupled dynamical system.

### No convergence

Both sides have a frequency knob. The contributor controls submission rate (drip). The maintainer controls review rate. The coupling between them is strong — feedback is fast, iteration is cheap. Strogatz would frame this as coupled oscillators. Depending on coupling strength: oscillation (arms race cycles), convergence (stable equilibrium), or chaos (unpredictable regime shifts like curl's shutdown). Strong coupling + fast iteration points to oscillation or chaos, not equilibrium. The research program doesn't end.

### Asymptotic bar

The bar rises until unaided contribution is impossible. Spell checkers → grammar checkers → style checkers → agent pipelines. Each generation of tooling becomes the floor. Nobody submits academic papers without LaTeX. Nobody will submit PRs without an agent pipeline. The tool becomes invisible, the unaided human becomes the outlier.

### Two-sided defense

Agents screening agents. Humans review only what survives machine pre-filtering. The maintainer never touches volume — they touch a feed at their chosen pace. The contributor's drip controls outbound rate. The maintainer's bot controls inbound rate. The human controls the frequency knob on both sides. The machines do the screening.

### Honeypots as active defense

An insta-ban suite that catches the bottom 90% by effort restores the hit rate above the maintainer's pain threshold. Open a bot-magnet issue with trivial change in dead code, passing test suite, maintainer-filed label — scores perfectly on every /actionable signal. Insta-ban on submission. The body-count check would walk right into it. Defense: a pristine issue with no history on a repo that has history is the tell. Real bugs have discussion, failed attempts, edge cases.

### Excalidraw pattern

18 dead PRs on one issue from 2 years ago. A human wouldn't touch it. The per-issue body-count check already catches this. No new tool needed — the existing threshold (3+ closed-unmerged) handles it. The lesson is that the tool works for extreme cases without modification.

### Complementation, not replacement

The machine handles volume and consistency. The human handles novelty and adaptation. The agent runs the pipeline; the human decides when to break it. Today the pipeline ran correctly — per-repo pacing, test gates, codex crosscheck, all green — and three batch-closes showed the rules were wrong. The human noticed a category the pipeline didn't have (org-level blast radius) and redrew the graph. That's metacognition: reasoning about the reasoning process, not just running it.

Machines adapt when someone writes a new rule. Humans adapt by noticing the rules don't apply anymore. Humans can outpace encoded adaptation because they engage in metacognition — they don't just follow the hypothesis graph, they notice when the graph itself is wrong. This is why complementation wins over full automation: the adversarial environment generates novel situations faster than any fixed pipeline can handle.

### Ratchet

Each human adaptation gets encoded into the pipeline and never regresses. The org-gate becomes baseline. The body-count check becomes baseline. Each tool only turns one way. The human adapts once, the machine remembers forever.

### Maintainer tooling demand

As the bar rises, maintainers' SNR problem gets worse. They will reach for tools that make the problem legible — tools that screen, score, and surface the signal before human attention is spent. If those tools are being built in public, by contributors red-teaming their own pipelines, they're the ones maintainers find first.

### Publication as forcing function

Publishing the attack forces merit-based evaluation. If the voice gate skill is public and anyone can use it, moderators can't reject selectively on suspicion of using it — that means rejecting anyone who writes well on their first post. But the countercase exists: communities may respond by restricting intake instead of improving evaluation (curl → GitHub-only reporting, no rewards). Trust vs merit is not resolved by publishing; it's surfaced.
