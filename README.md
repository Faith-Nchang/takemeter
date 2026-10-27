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

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

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

**Agreement rate:** ___ / 30 = ___%

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

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
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
