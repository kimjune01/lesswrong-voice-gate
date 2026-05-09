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
