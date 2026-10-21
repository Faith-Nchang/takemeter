# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1.

Overall accuracy is at least 0.65 on the held-out set, on every one of the three seeds.

**Why this target:**

A guesser gets about 0.33 on three labels, and a model that only ever says `asserted` would get roughly 0.50, since about half the comments I read were that label. 0.65 sits 15 points above that lazy strategy, which is where I'd start calling it useful rather than lucky. I did not pick 0.75 because the training set is only about 140 comments and the `argued`/`asserted` line is subtle even for me.

---

## 2.

Every label has an F1 of at least 0.50 on the held-out set, on every seed.

**Why this target:**

This stops overall accuracy from hiding a dead label. I set 0.50 rather than 0.60 because `argued` is my thinnest label (about a quarter of what I read, so roughly 8 of about 30 test posts) and its F1 will swing between seeds. 0.50 still means it has to beat the mushy middle of `asserted`.

---

## 3.

No label is more than 60% of labels.csv, and none is less than 15%.

**Why this target:**

The project's cap is 70%, but `asserted` was about half of the 80 comments I read, so 70% would let the data go badly lopsided before I noticed. The 15% floor is 30 of 200 rows, the least I think a label needs so that a test split has more than a handful of it. I will oversample long comments to reach this for `argued`.

---

## 4.

On the 30 staff posts, my raw agreement with the staff labels is at least 0.70.

**Why this target:**

The staff taxonomy says a rate in the 70s is normal because staff picked ambiguous posts on purpose. 0.70 is the bottom of that range, so it asks me to apply a written rule I didn't write about as consistently as its authors did, which is the same skill that decides whether my own 200 labels are consistent.

---

## 5.

In results.json, the model's most-confident third of test predictions is at least 0.80 accurate, and at least 15 points more accurate than its least-confident third.

**Why this target:**

A confidence score is only useful if it separates the guesses worth trusting from the rest. 0.80 is above my overall accuracy target of 0.65 on purpose, since confident guesses should be right more often. With about 30 test posts a third is only 10, so one wrong answer moves this by 10 points, and that is why I only asked for 80% and not 90%.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
