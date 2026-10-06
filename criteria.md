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

## 1. Overall accuracy

Overall accuracy on the held-out test set is at least 0.65, in all three seed runs.

**Why this target:** For a three-label task, chance alone yields a correct label about one-third of the time, so 0.65 is nearly double that baseline. This is an ambitious target: the `analysis` and `hot_take` labels overlap in practice, and applying the taxonomy consistently required an explicit tie-breaking rule even when labeling by hand. The dataset is also small, at roughly 200 examples total, which limits how much the model can learn.

---

## 2. Per-label performance

Every label has an F1 of at least 0.50 on the held-out test set, in all three seed runs.

**Why this target:** `analysis` is expected to be the smallest label, since it requires a poster to cite a number, a model name, or a specific observation — more effort than asking a question or venting. If `analysis` ends up with roughly 40-50 examples total, only about 6-8 land in a 15% test split, and F1 computed on that few examples varies considerably between seeds. The target is set lower than criterion 1's accuracy target on purpose, to account for that instability, while still requiring every label — not only the easier ones — to clear a real bar above chance.

---

## 3. Balance

No single label makes up more than 55% of final labeled set.

**Why this target:** r/fitbit is a support subreddit, so `request` (troubleshooting, buying advice) is expected to be the most common label — possibly a majority if posts are collected passively. If one label takes over the dataset, a model can reach high overall accuracy (criterion 1) just by defaulting to that label, while quietly failing criterion 2 on the rest. A perfectly even split would be about 33% per label; 55% still allows real skew while setting a real limit that has to be actively collected against, not left to chance.

---

## 4. Balanced performance across labels

Macro F1 on the held-out test set is at least 0.55, in all three seed runs.

**Why this target:** Macro F1 is the average of each label's individual F1 score, with all three labels weighted equally regardless of how many examples each one has. This is different from criterion 2, which only checks that each label clears its own floor on its own — a model could pass that while still being uneven overall (for example, a strong score on `request` and weak scores on `analysis` and `hot_take`). Macro F1 would reflect that imbalance directly, since a weak label pulls the average down just as much as a strong label pulls it up. With 3 labels, a model with no real signal would score close to chance on each one individually, so 0.55 represents a clear, meaningful improvement across the full taxonomy, not just on the easiest label.

---

## 5. Confidence

The accuracy of the model's most-confident third of test predictions (`accuracy_most_confident_third` in `results.json`) is at least 15 percentage points higher than the accuracy of its least-confident third (`accuracy_least_confident_third`), in all three seed runs.

**Why this target:** A confidence score is only useful if it actually matches reality. If the model's most-confident and least-confident predictions are right about equally often, the confidence number isn't telling anyone anything real — it just looks precise. A 15-point gap is a clear, checkable sign that confidence and correctness move together, while still being realistic given the fuzzy boundary between `analysis` and `hot_take`, which will cause some wrong answers even among the model's confident guesses. 



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
