# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

TakeMeter sorts **Hacker News comments** (public threads on news.ycombinator.com, no login) by how much a comment *supports* what it says. Regulars know the difference between a comment that shows its work and one that just announces a conclusion, and a third kind that isn't making a claim at all. The classifier has three labels: `argued`, `asserted` and `chatter`.

---

## Label Taxonomy

Three labels. A comment is judged on what it does, not on its topic or tone.

### `argued`

**Definition:** makes a point and backs it with a specific, checkable fact (a number, date, quote, link, named example or reproducible detail) that carries the point.

**Example 1:**
> Scale is a big part of it.
> Every minute, 500 hours of video are uploaded to YouTube.
> Even Google can't afford to run the biggest baddest model on every single one of those hours.

**Example 2:**
> https://chaos.social/@fluepke/117474156947559280
> Private keys can be trivially recovered from the public keys because they're tiny. I found D=156, flüpke found D=4.
> Yes, that's the entire private key.

### `asserted`

**Definition:** states a claim, opinion or prediction with no support, or with support a stranger couldn't check ("in my experience", "I recall reading"). It might be true; it isn't argued.

**Example 1:**
> Cummings is an absolute crank. He had the opportunity to try and he ended up failing spectacularly, like Liz Truss or Yannis Varoufakis.

**Example 2:**
> No non-technical user in their right mind will be trying to vibecode an OS and flashing it to some random off-brand tablet anytime soon.

### `chatter`

**Definition:** asserts nothing: a genuine question, a joke, agreement or thanks, a plug, a bare pointer to a link.

**Example 1:**
> Maybe OP is on naughty list facing serious scrutiny during the postmortem review.

**Example 2:**
> Why do you posit that this is a lot more important than the Internet?

### The hardest boundary

**Which two labels:** `argued` vs `asserted`.

**The post that sits on it:**
> People should prepare their kids to be happy adults, not just employed adults.
> I recall reading of studies that found that a significant majority of adults who did not learn to play an instrument when they were young wish that they had.
> Only a small minority of adults who did learn to play an instrument when young wish that they hadn't.

It names a finding ("studies found...") so it *sounds* evidenced, but no study is named, no number given, and nobody could look it up from this text.

**The decision rule I used every time:**
Cross out the specific fact and read what's left. If the conclusion stands just as well without it, the fact was decoration, so the comment is `asserted`. If the comment falls apart without it, the fact is doing the work, so it's `argued`. A fact only counts if a stranger could check it from the text alone: a number, a named source, a link, a date, a reproducible detail. "I recall reading", "studies show" and "in my experience" with nothing specific count as no fact at all.

**Two secondary rules**
- `asserted` vs `chatter`: if you can state the claim the comment is making in one sentence, it is `asserted`, even if it is phrased as a question or a joke ("Why would anyone use X?" implies a claim). A real question that asks for information is `chatter`.
- `argued` vs `chatter`: a bare quote or link with no point attached is `chatter`. It becomes `argued` only when the comment says what the quote or link proves.

---

## The Dataset

**Where the posts came from:** 200 public Hacker News comments from 12 front-page threads on 2026-10-20 (AI inference, a certificate-authority compromise, iPad software, UK speech law, a cloud outage, parenting, housing and others), pulled through the public HN search API (`hn.algolia.com/api/v1`). Whole comments, no trimming, no usernames kept. I only drew from comments of 3 to 150 words, so very long comments are missing. I sampled so that long and short comments were both included, and I took every `argued` comment I found, because it is the rarest label. That means the label mix below is *not* what a random HN thread looks like: `asserted` is closer to two thirds of the 330 comments I labeled before choosing these 200.

**How I labelled them:** I did not label these cold. Claude read 330 sampled comments and applied my definitions and decision rule, and I took the 200 rows below from them. Every row's `note` says it was pre-labelled by Claude and not yet reviewed by me. <!-- TODO before submitting: read every row yourself, fix the labels you disagree with, relabel 20 rows with no help and set their note to `cold`, and update this paragraph to say what you changed. -->

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `asserted` | 92 | 46% |
| `argued` | 60 | 30% |
| `chatter` | 48 | 24% |
| **Total** | 200 | 100% |

One thing to watch: `argued` comments are longer (median about 57 words, against about 39 for `asserted` and about 20 for `chatter`), so a model could lean on length instead of evidence.

**Three hard cases**

**1.**
> *The post:*
> > "A computer can never be held accountable, therefore a computer must never make a management decision."
> Legendary quote from an internal 1979 IBM training document [1]
> https://simonwillison.net/2025/Feb/3/a-computer-can-never-be...
>
> *Could have been:* `argued` or `chatter`
>
> *I chose:* `chatter`, because the quote and link are the whole comment. It never says what the quote proves. Under my secondary rule a bare quote or link is `chatter`.

**2.**
> *The post:*
> I very much doubt it.  For consumers it is all about cost.  Heating in winter with Natural Gas is 1/3 the price of electricity and much cheaper than Oil.
> To get people off of fossil fuels, remove oil subsidizes and tax fossil fuels a lot.  Otherwise moving to renewals will take a very long time.
> And yes, we can put up solar panels on your house, but that cost $ which people do not have.  Solar Panels should be subsidized in an easy manner for people to get funds for installation.
>
> *Could have been:* `argued` or `asserted`
>
> *I chose:* `argued`, because "1/3 the price of electricity" is a checkable number and the argument (consumers decide on cost) rests on it. It is unsourced, so a stricter reading would say `asserted`.

**3.**
> *The post:*
> Apple Music has a dedicated button to add a song to favourites, which I use. When I plug my phone in my car I can then say “play my favourites” and it plays it.
> Until I’ve upgraded to iOS 27 and after saying “play my favourites” the phone responds now with “playing FAMILY favourites” (no idea what that means) and plays some random songs.
> Incredibly annoying, great example of trying to be too clever for its own good.
>
> *Could have been:* `argued` or `asserted`
>
> *I chose:* `argued`, because the specific behavior ("play my favourites" becoming "playing FAMILY favourites" after iOS 27) is a detail a stranger could reproduce, and the complaint depends on it. A vaguer "Siri is worse now" would have been `asserted`.

---

## The Training Run

**Base model:** `distilbert-base-uncased`, fine-tuned for 3-way classification.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, 128 tokens per post, seed 42. Run on CPU (no GPU), PyTorch 2.14.1+cpu. Training took about 2.7 minutes.

**Anything I changed from the defaults, and why:** Nothing. Every hyperparameter is the notebook's default. I did not do the optional second run. I ran the notebook's section 1–5 code as a script (same code, same settings) because Jupyter isn't installed in my `.venv`.

**Split sizes:** 139 train / 31 validation / 30 test (stratified 70/15/15 by the notebook).

| Label | Train | Val | Test |
|---|---|---|---|
| `argued` | 42 | 9 | 9 |
| `asserted` | 64 | 14 | 14 |
| `chatter` | 33 | 8 | 7 |

`chatter` has only 7 examples in the test split, under the notebook's threshold of 8, so its score will move a lot between seeds.

I haven't studied the numbers yet; that's next unit's work. They are in `results.json`. On a first glance, the model never got a `chatter` post right on this split.

---

## How I Used AI
**1. Reading the community and drafting the labels.** Claude pulled real comments from the Hacker News API, read 80 of them, and proposed the three labels (`argued`, `asserted`, `chatter`) and the "cross out the fact" decision rule for the `argued` vs `asserted` boundary. I asked it to do the reading and first draft; the label names, definitions and rule are in the README as it drafted them.

**2. Pre-labelling the dataset (disclosure).** Claude pre-labelled 330 sampled comments using my definitions and rule, and the 200 rows in `labels.csv` were chosen from those 330. Every row's `note` says "pre-labelled by Claude, not yet reviewed by me". When I asked whether the file made sense, Claude flagged about a dozen `argued` rows it thought were too generous (public-service notices, bare links, vague numbers). 


---

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════
     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

## Baseline vs. Trained

**Evaluation inputs:** The baseline uses all 30 rows of the existing `test_split.csv`, with no sample cap or fallback, and the exact definitions copied from this README into `label_definitions.txt`. Reconstructing the notebook's seed-42 split from `labels.csv` reproduced the saved test posts and labels in order. Their per-label counts match `results.json`, and none of these test texts occurs in the reconstructed training or validation sets. The original `criteria.md`, `labels.csv`, `results.json`, and `test_split.csv` remain unchanged. The environment check passed all nine checks, with only the initial baseline-download warning.

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.400 | 0.567 | +0.167 |
| Macro F1 | 0.300 | 0.382 | +0.082 |
| F1 — `argued` | 0.400 | 0.429 | +0.029 |
| F1 — `asserted` | 0.500 | 0.718 | +0.218 |
| F1 — `chatter` | 0.000 | 0.000 | +0.000 |

Difference is trained minus baseline, rounded to three decimal places. Run with `.venv\Scripts\python.exe baseline.py --trained results.json` on October 27, 2026, using `facebook/bart-large-mnli`. The baseline predictions and full-precision metrics are saved in `baseline_results.json`; the trained results remain in `results.json`.

**What I predicted before I looked:**
 the trained model may beat the zero-shot baseline on `asserted`, because it learned this community's unsupported claims. The existing trained results had already been inspected; this is a prediction before baseline results, not a claim that it preceded the unit 5 results.

**What the gap actually means:**
On this held-out split, the trained model gets 17 of 30 posts right, compared with 12 of 30 for the zero-shot baseline: a gain of 16.7 percentage points in accuracy and 0.082 macro F1. The prediction about `asserted` was supported: its F1 increases from 0.500 to 0.718, the largest per-label gain. `argued` improves only slightly, and both models have zero F1 on `chatter`, so training has not solved that label. This is evidence of a benefit on this split, not yet evidence that the gain is stable across different splits; the three-seed evaluation is still pending.



---

## Run Log — Before

### Spread across three seeds

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.567 | 0.467 | 0.467 | 0.100 |
| Macro F1 | 0.382 | 0.212 | 0.212 | 0.170 |
| F1 — `argued` | 0.429 | 0.000 | 0.000 | 0.429 |
| F1 — `asserted` | 0.718 | 0.636 | 0.636 | 0.082 |
| F1 — `chatter` | 0.000 | 0.000 | 0.000 | 0.000 |

Accuracy varies by 10 percentage points across these splits. Seeds 7 and 2024 predict only `asserted`, while `chatter` has zero F1 in all three trials; the spread therefore hides substantial differences in whether `argued` is recognized at all.

### Criteria run log

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 on every seed | 0.567 | 0.467 | 0.467 | MISSED |
| 2. Every label F1 (minimum shown) | ≥ 0.50 on every seed | 0.000 | 0.000 | 0.000 | MISSED |
| 3. Dataset label proportions | Each between 15% and 60% | 30% / 46% / 24% | Same dataset | Same dataset | MET |
| 4. Staff agreement | ≥ 0.70 on 30 posts | Pending | Shared check | Shared check | PENDING |
| 5. Confidence: top-third accuracy / gap over bottom third | ≥ 0.80 / ≥ 0.15 | 0.70 / 0.20 | 0.60 / 0.40 | 0.90 / 0.80 | MISSED |

Criterion 3 is measured once on all 200 rows of `labels.csv`: `argued` 60 (30%), `asserted` 92 (46%), and `chatter` 48 (24%). These are dataset proportions, not three independent trial measurements. Criterion 4 is also a single shared check and cannot be evaluated without the staff answer key.

Criterion 5 originally names `results.json`; that original measurement is 0.70 top-third accuracy versus 0.50 bottom-third accuracy, a 0.20 gap, so it misses the 0.80 requirement. The row also applies the same thresholds to all three new trials without changing the original criterion: seed 2024 alone passes both conditions, so the repeated-trial check also misses. The notebook sorts winning softmax confidence using `np.argsort` and uses the first and last 10 predictions (30 divided by 3); ties follow NumPy's existing sort behavior. No threshold or original criterion was revised.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

This is the **seed 42 before-run** matrix (`all_results[0]` in notebook section 7), covering 30 held-out posts. Rows are true labels; columns are predictions.

| true \ predicted | argued | asserted | chatter |
|---|---|---|---|
| **argued** | 3 | 6 | 0 |
| **asserted** | 0 | 14 | 0 |
| **chatter** | 2 | 5 | 0 |

**My biggest off-diagonal number, and what it means:**
Six real `argued` posts were predicted as `asserted`, while zero went the other way, showing that on this split the model tends to treat supported arguments as unsupported claims.

<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 on every seed | MISSED | All three accuracies are below 0.65; even the best is only 17/30 (0.567). |
| 2 | Every label F1 | ≥ 0.50 on every seed | MISSED | `chatter` has F1 0.000 on every seed; `argued` has 0.429, 0.000, 0.000. |
| 3 | Dataset balance | Every label between 15% and 60% | MET | The full-dataset proportions are 30%, 46%, and 24%, all within the original bounds. |
| 4 | Agreement with staff | ≥ 0.70 (at least 21/30) | PENDING | The TF's answer key is unavailable; no agreement rate has been calculated. |
| 5 | Confidence usefulness | Top third ≥ 0.80 accurate and gap ≥ 0.15 | MISSED | Original `results.json`: 0.70 top-third accuracy, despite a passing 0.20 gap. Across the new trials, seeds 42 and 7 also fail top-third accuracy. |

**Diagnoses**

**Criterion 1 — majority-label predictions dominate.** Seeds 7 and 2024 call all 30 test posts `asserted`, giving 14/30 accuracy and missing all 9 `argued` and 7 `chatter` posts in each split. Seed 42 recovers only 3 `argued` posts and no `chatter` posts. This identifies the immediate source of the accuracy miss: the model has not learned to separate the minority labels reliably. Dataset imbalance may contribute (`asserted` is 46%), but the counts alone do not establish why the model collapses to it.

**Criterion 2 — `chatter` is never predicted, and `argued` is unstable.** In seed 42, 5 of the 7 true `chatter` posts go to `asserted` and 2 to `argued`; in the other two seeds all 7 go to `asserted`. `argued` recall falls from 3/9 to 0/9. `chatter` is actually the smallest label (48 examples), despite the original criterion rationale anticipating that `argued` would be smallest; only 7 `chatter` examples occur in each test split. Small support makes the estimate coarse, but zero predictions across all three runs indicates a repeated failure, not merely one unlucky split. The remaining causes could include inconsistent training labels, difficult boundaries, or insufficient learning from 200 examples. The existing AI-prelabeling disclosure makes a label audit warranted, but no independent review currently proves labeling inconsistency.

**Criterion 5 — ranking confidence is better than trusting its top group.** Every seed's top third is more accurate than its bottom third by at least 0.15, but only seed 2024 reaches 0.80 top-third accuracy. Seed 42's top group gets 7/10 right and seed 7's gets 6/10 right. Seed 7 predicts only `asserted`, so even its most confident group cannot reliably separate the labels; confidence is ranking guesses within a model that already misses both minority classes. With 10 predictions per group, one error changes accuracy by 10 percentage points. This explains the measured miss without claiming that these three small groups establish calibrated probabilities.

**Limits of the diagnosis.** Accuracy varies by 10 percentage points, and `argued` F1 varies by 0.429. Those results support split sensitivity and majority-label collapse, but do not prove whether the underlying cause is labeling or training. The staff agreement evidence is unavailable, and the staff-post labels were produced by Codex, so a later comparison of that file would measure AI agreement rather than independent evidence of my own consistency. The staff taxonomy also differs from my taxonomy. No staff disagreements or agreement rate are invented here.

**One improvement selected for Phase 4 — increase epochs from 3 to 5.** In the saved before-run notebook output, validation loss still decreases from epoch 2 to epoch 3 in all three seeds (approximately 0.967→0.946, 1.009→0.997, and 1.018→1.010), while two runs still predict only the majority label. That gave a concrete reason to test whether additional training helps learn the minority boundaries. Only `EPOCHS` changed; the learning rate, batch size, token limit, base model, dataset, taxonomy, criteria, and seeds 42/7/2024 stayed fixed. The completed experiment and results are reported in **The Improvement** below; the benefit does not prove that undertraining was the only cause.

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** PENDING — `data/staff_labels.csv` is unavailable because I do not currently have access to a TF. The 30 labels in `my_staff_labels.csv` were produced by Codex under the staff taxonomy without consulting an answer key; this is not a completed human agreement check. Criterion 4 remains pending until the required independent labels and evidence are available.

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

No disagreements are listed yet because the staff answer key is unavailable. Once it is supplied, run `agreement.py` and record every differing post, both labels, the reasoned call under the staff rules, and its category (rule applied loosely, rule missing, or defensible ambiguity).

**What the pattern in my disagreements tells me:**

Pending. Without staff labels, no disagreements can be identified or adjudicated, and no labeling-consistency conclusion can be drawn from this check.


---

## The Improvement

**What I changed:**

In the existing notebook, increased `EPOCHS` from 3 to 5 and set `RUN_LABEL = "after"` so the new trials write `results_three_seeds_after.json`. The run label controls the output filename; epochs are the only experimental change. The base model remains `distilbert-base-uncased`, learning rate `2e-5`, batch size 16, token limit 128, and seeds 42, 7, and 2024. The dataset, label order, taxonomy, criteria, and split procedure are unchanged, so the same seed selects the same train/validation/test posts before and after. Each trial starts from the base model rather than continuing the previous seed's trained model.

**Which diagnosis pointed at it:**

Seeds 7 and 2024 predicted only `asserted` before the change, and validation loss was still falling between epochs 2 and 3 in every seed. Two additional epochs test whether more training helps distinguish `argued` and `chatter`. This is a measured training hypothesis; the unavailable staff agreement check cannot establish whether training labels are also inconsistent.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 on every seed | 0.567 | 0.567 | 0.633 | MISSED |
| 2. Every label F1 (minimum shown) | ≥ 0.50 on every seed | 0.000 | 0.000 | 0.444 | MISSED |
| 3. Dataset label proportions | Each between 15% and 60% | 30% / 46% / 24% | Same dataset | Same dataset | MET |
| 4. Staff agreement | ≥ 0.70 on 30 posts | Pending | Shared check | Shared check | PENDING |
| 5. Confidence: top-third accuracy / gap over bottom third | ≥ 0.80 / ≥ 0.15 | 0.70 / 0.30 | 0.70 / 0.30 | 1.00 / 0.70 | MISSED |

These are the completed five-epoch trials in `results_three_seeds_after.json`, run on CPU with PyTorch `2.14.1+cpu`. Criterion 3 remains a single dataset-level measurement, and criterion 4 remains unavailable. Criterion 5 applies the original thresholds to the after-trials for comparison; the original `results.json` measurement is preserved and still misses its original target. Both confidence conditions must hold in every trial, so seed 2024's success does not make the overall verdict MET. Values below are rounded for display; verdicts use unrounded values.

### Before and after, paired by seed

| Measure | Seed 42 before | Seed 42 after | Seed 7 before | Seed 7 after | Seed 2024 before | Seed 2024 after |
|---|---|---|---|---|---|---|
| Overall accuracy | 0.567 | 0.567 | 0.467 | 0.567 | 0.467 | 0.633 |
| Macro F1 | 0.382 | 0.422 | 0.212 | 0.382 | 0.212 | 0.554 |
| F1 — `argued` | 0.429 | 0.600 | 0.000 | 0.429 | 0.000 | 0.500 |
| F1 — `asserted` | 0.718 | 0.667 | 0.636 | 0.718 | 0.636 | 0.718 |
| F1 — `chatter` | 0.000 | 0.000 | 0.000 | 0.000 | 0.000 | 0.444 |
| Most-confident-third accuracy | 0.700 | 0.700 | 0.600 | 0.700 | 0.900 | 1.000 |
| Confidence accuracy gap | 0.200 | 0.300 | 0.400 | 0.300 | 0.800 | 0.700 |

### Confusion matrix after the change — seed 42

| true \ predicted | argued | asserted | chatter |
|---|---|---|---|
| **argued** | 6 | 3 | 0 |
| **asserted** | 3 | 11 | 0 |
| **chatter** | 2 | 5 | 0 |

The largest off-diagonal count is now 5 true `chatter` posts called `asserted`, with none going the other way. Compared with the seed-42 before-matrix, 3 additional `argued` posts are correct, but 3 additional `asserted` posts become `argued`; the two changes cancel in overall accuracy.

**Did it help, and how do I know:**

Partly. Accuracy was unchanged on seed 42, improved by 10 percentage points on seed 7, and improved by 16.7 points on seed 2024. Mean accuracy across the three trials rose from 0.500 to 0.589; macro F1 improved on every seed. Neither seed 7 nor seed 2024 predicts only `asserted` anymore, and seed 2024 correctly identifies 2 of 7 `chatter` posts. However, `chatter` still has zero F1 on two seeds and only 0.444 on the third, while `asserted` F1 regressed from 0.718 to 0.667 on seed 42. No previously missed across-seed criterion became MET. More training helped some boundaries, but did not make the classifier meet the original targets.

**Verification:** The protected inputs, baseline results, original trained results, saved original test split, and three-seed before-results were checked byte-for-byte after execution and remained unchanged. With the same dataset and split code, the same three seeds retain the same held-out posts. The notebook's section 6/7 outputs now show the after-run; the before-run remains in `results_three_seeds_before.json` and the before tables above.

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

**Accuracy (criterion 1, MISSED):** Even the best after-run is 19/30 (0.633), below 0.65. Next I would review the `argued`/`asserted` errors against the written definitions and obtain independently reviewed examples before another experiment. I stopped after the one planned improvement so its effect could be measured separately.

**Per-label performance (criterion 2, MISSED):** `chatter` is still the weakest label: F1 is 0.000, 0.000, and 0.444. `argued` also misses 0.50 on seed 7. Next I would audit the `chatter`/`asserted` boundary and, if needed, collect more carefully reviewed examples using the unchanged taxonomy. Those are additional interventions, so neither was mixed into this epoch experiment.

**Confidence (criterion 5, MISSED):** Seeds 42 and 7 reach only 7/10 correct in their most-confident thirds. Next I would evaluate confidence on more held-out examples and assess whether it remains useful after fixing the weak labels. I did not tune a confidence threshold on these test results or lower the target; the current groups are too small to justify a broad reliability claim.

**Staff agreement (criterion 4, PENDING):** The TF's answer key is unavailable, so `agreement_results.json` and disagreement adjudications are still missing. Obtain the key and run `agreement.py` to complete that comparison. Because Codex supplied the current 30 labels, an independent human labeling pass is also needed to support the original claim about my own consistency; AI-to-staff agreement alone cannot do that.

**Submission status:** The baseline, both three-seed result files, both run logs, verdicts, diagnoses, and typed matrices are present. The staff agreement work remains pending, so this is not yet a complete assignment submission. No commit or submission was made as part of this phase; the current unit has three commits before this phase, so at least one additional commit is needed to reach four. Use the same repository URL as unit 5.

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
The taxonomy is meant to distinguish supported arguments, unsupported claims, and comments that assert nothing, but the seed-42 after-matrix still sends every `chatter` post into a claim label. Longer training recovered more `argued` posts while introducing reverse errors from `asserted`, so the model has learned part of the support boundary without reliably learning all three intended categories.

<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
