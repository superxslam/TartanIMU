# TartanIMU Challenge — Team Report Template

Copy this file, rename it `report_<team_name>.md`, and submit it together with
your code and weights by **2026-09-20 23:55 UTC** — the moment the competition
closes. **Earlier is much better**; see the note at the end about section 8.

A placement becomes final only after we receive your artifacts and audit them,
so please do not leave this to the last evening.

**Please keep the headings below and their order unchanged, and keep every
section — write "not used" rather than deleting one.** Comparability across
teams matters far more than length: a filled-in page is worth more to us than a
polished essay. Every team that submits is credited in the challenge analysis
paper.

Delete the *italic prompts* as you fill each section in.

---

## Submission identification

| Field | Your answer |
| --- | --- |
| Team name (exactly as on the leaderboard) | |
| Members (name — affiliation) | |
| Contact email | |
| Submission you want ranked (Kaggle submission ID, or the submission filename + its UTC timestamp) | |
| Public score of that submission | |
| Score you expect us to reproduce | |
| Code repository or archive (a private link is fine) | |
| Commit SHA that produced the checkpoint | |
| Checkpoint file name(s) | |
| Checkpoint SHA-256 | |
| Config file path inside the repository | |
| Total training cost (GPU type × hours) | |

> `sha256sum <checkpoint>.pt` on Linux, `shasum -a 256 <checkpoint>.pt` on macOS.
> The hash lets us prove the weights we audit are the weights you ranked with.

## Artifact checklist

Tick each item once it is included in the package you send.

- [ ] **Final checkpoint(s)** for the submission named above.
- [ ] **Training code** at the exact commit named above.
- [ ] **The exact config / hyper-parameter file** used — the file itself, not a
      description of it.
- [ ] **Inference script** that turns the checkpoint into a submission CSV,
      including every test-time processing step.
- [ ] **Environment**: a lockfile, `requirements.txt`, or a container image.
- [ ] **This report.**

If any part of your work is under a constraint that prevents sharing, tell us
in the last section — we would much rather know the constraint than receive an
incomplete package.

## Compliance statement

The rule, restated: **predictions must come from one model with one shared set
of weights.** Platform-specific routing *inside* one network (learned
conditioning, mixture-of-experts, adaptive heads) is allowed and encouraged;
four separately selected expert models are not.

- [ ] All ranked predictions come from a single model with one shared set of
      weights.

Then declare, plainly, what your pipeline does. These are **disclosures, not
accusations** — several of them are perfectly legal, and we ask only so that
the audit reproduces your number instead of a different one.

| Question | Yes / No | If yes, describe |
| --- | --- | --- |
| Weight averaging across checkpoints (soup, EMA, SWA)? | | |
| Test-time augmentation, output scaling, or calibration? | | |
| Any per-platform behavior — and is it internal routing or separate models? | | |
| Pretrained weights not included in the release? | | |
| External data (public or private) beyond the challenge dataset? | | |
| Anything else that changes the numbers and is not in the training code? | | |

Signed (name, date):

---

## Technical report

*The ten sections below are the report itself. Short is fine — one to three
pages is typical.*

## 1. Backbone

*One or two sentences on the architecture, plus parameter count. Name the
family (e.g. CNN + recurrent, temporal convolution, transformer, state-space)
and say what is new relative to the released baseline.*

## 2. Loss

*Every term and its weight. If a weight was tuned or scheduled, say so.*

## 3. Data handling

*Augmentations, resampling, normalization, and any re-splitting of train/val.
If you trained on a split different from the released one, this is the section
that tells us so.*

## 4. Training schedule

*Epochs, learning-rate schedule, batch size, optimizer, hardware, wall-clock
time. Enough for us to re-run it.*

## 5. Model selection — how did you choose which checkpoint to submit?

*Be specific and honest. Validation score? Public leaderboard? An average of
several checkpoints? A hunch on the last epoch? How many candidate submissions
did you evaluate before choosing this one?*

> **Good:** "Best of 14 checkpoints by public leaderboard score; local
> validation was not used because it disagreed with the leaderboard."
> **Not enough:** "We selected the best model."
>
> This section is a genuine research result, not paperwork. How teams decide
> what to submit is one of the questions the analysis paper asks.

## 6. Inference-time processing

*Everything that happens between the checkpoint and the CSV: TTA, weight
averaging, output scaling, filtering, smoothing, clipping, per-platform
constants. Post-processing is easy to forget and often worth more than it
looks.*

## 7. External resources

*Pretrained weights, external datasets, other people's code you built on,
anything not in the release. Please credit it here.*

## 8. ★ What did NOT work

*Ideas you tried and abandoned, and roughly what each cost you — an afternoon,
a week, 200 GPU-hours. Bullet points are ideal.*

> **This is the section we value most, and the one almost nobody publishes.**
> Fifty-one teams' negative results are the single most valuable thing this
> challenge can leave behind: they tell the next generation of researchers which
> doors are already known to be closed. Rough numbers are fine — "tried X, gave
> up after two days, no gain" is a complete entry.

- Tried … → outcome … → cost …
- Tried … → outcome … → cost …
- Tried … → outcome … → cost …

## 9. ★ If you had to name one component that mattered most, what would it be?

*One paragraph. If you could keep exactly one thing from your pipeline and
throw the rest away, what would you keep, and why do you believe it?*

## 10. Anything else we should know

*Constraints on sharing, known failure cases, things you would do differently,
questions for the organizers, or feedback on the benchmark itself. Optional but
read carefully.*

---

## One practical note: write section 8 *now*, not in October

Section 8 is the part every team finds hardest to reconstruct afterwards, and
the part we value most. Right now you still remember which ideas you tried and
dropped, and roughly what they cost. A month after you stop running experiments
most of that is gone, and what remains has usually been tidied up into a story.

Keep a running list from today until the close and section 8 will cost you
almost nothing — while being the most valuable page in your report.

## How your artifacts are used

They are used for exactly two things: **verifying the final ranking**, and the
**challenge analysis paper**, in which every submitting team is credited. We do
not redistribute your code or your weights.
