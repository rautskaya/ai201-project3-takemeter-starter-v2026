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

## What This Does

This classifier sorts r/fitbit posts into three labels:

- `analysis`: a claim backed by a specific, checkable fact (a battery %, a step count, a spec)
- `request`: asking for help, troubleshooting or buying advice
- `hot_take`: a confident opinion with no evidence

It separates claims that have evidence behind them from questions and from opinions that don't.

---

## Label Taxonomy

### `analysis`

**Definition:** Makes a claim backed by a specific, checkable fact — a battery percentage, a step count, a named model/spec comparison, a screenshot of data.

**Example 1:**
> "Google health app and Charge 6 battery life" — battery went from 55% to 49% during a 24-minute drive, cutting battery life in half.

**Example 2:**
> "I run 3 miles everyday and it was always tracked appropriately before the switch and now my runs are getting under valued by a lot (between half mile and .75)." — a known distance measured against Fitbit's number.

### `request`

**Definition:** Asking for specific help, troubleshooting, or buying advice — seeking information rather than asserting a claim.

**Example 1:**
> "I accidentally removed my Fitbit from the app and now I can't get it paired again. The setup process finds the device but fails after entering the code. Any ideas?"

**Example 2:**
> "My Fitbit app is showing yesterday's resting heart rate but today's value never appears, even after several manual syncs. Has anyone fixed this without reinstalling the app?"

### `hot_take`

**Definition:** A confident claim about the product or brand with no supporting evidence offered.

**Example 1:**
> "After 13yrs of owning fitbit products I am moving on... Google pretty much ruined a great product by stripping away features while simultaneously making the app more buggy and worthless." — a strong verdict with no specific feature, number or bug named.

**Example 2:**
> "Google always makes it worse" — "Why does everything Google touches turn to $hit? I despise this new app... We just want the numbers... It's now useless." Strong, emotional judgment with no specific data or checkable fact behind it.

### The hardest boundary

**Which two labels:** `analysis` vs. `hot_take`

**The decision rule I used every time:** If a strong opinion is backed by a specific, checkable fact or measurement, label it `analysis` — even if the tone is heated. Only use `hot_take` when the judgment is asserted with no supporting evidence at all.

This came from a genuinely ambiguous post: "My charge 6 battery life has plummeted from 6-7 days pre-app migration to 3-4 days with the google health app... Am I insane or is this app somehow draining my battery twice as fast?" The frustrated tone made it *feel* like `hot_take`, but it gives a specific before-and-after battery measurement. The rule is about evidence, not tone, so this one is `analysis`.

---

## The Dataset

**Where the posts came from:** Public posts from r/fitbit.

**How I labelled them:** I labelled 20 posts cold, by hand. I labelled the remaining 180 using an AI tool, then checked them and fixed about 20.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `request` | 74 | 37% |
| `analysis` | 72 | 36% |
| `hot_take` | 54 | 27% |
| **Total** | 200 | 100% |

**Three hard cases**

**1. Charge 6 battery drain**
> *The post:* Battery loses about 15% overnight. Did the Google Health app cause it, and does anyone have a fix?
>
> *Could have been:* `request` or `analysis`
>
> *I chose `request`, because:* It has a measurable fact, but its main purpose is "how do I fix this?"

**2. Fitbit Air workout accuracy**
> *The post:* Compares Fitbit with another heart-rate monitor using specific numbers (75–93% HR, 5,500 steps, sleep score 92), then says the data is unreliable.
>
> *Could have been:* `analysis` or `hot_take`
>
> *I chose `analysis`, because:* The tone is harsh, but the complaint is backed by checkable evidence. Evidence beats tone.

**3. "I'm done with Fitbit" Charge 6 post**
> *The post:* Charge 6 stopped charging. The user has owned several Fitbits and ends with "Fitbit, I'm done with you."
>
> *Could have been:* `hot_take` or `request`
>
> *I chose `hot_take`, because:* There's a technical problem, but the post is venting and quitting the brand, not asking how to fix it.

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs · learning rate 2e-5 · batch size 16 · seed 42 · max length 128 tokens

**Anything I changed from the defaults, and why:** Nothing. I kept the defaults so this first run is a clean starting point to compare against.

**Split sizes:** train 139 · val 31 · test 30

| Label | Train | Val | Test |
|---|---|---|---|
| `analysis` | 50 | 11 | 11 |
| `hot_take` | 38 | 8 | 8 |
| `request` | 51 | 12 | 11 |

`hot_take` has only 8 posts in the test split, so its F1 will be the jumpiest across seeds.

---

## How I Used AI

**Moment 1**

- *What I asked for:* What my notebook section 2 output meant (`3 labels: analysis, hot_take, request`, plus the default settings).
- *What came back:* Claude checked `labels.csv` against `LABELS` and confirmed they matched exactly. It reported my counts (74 / 72 / 54) and warned that `hot_take` would get only about 8 posts in the test split, so its score would be jumpy.
- *What I changed:* Nothing in the data. I kept the default settings and treated `hot_take` as my least stable label.

**Moment 2**

- *What I asked for:* I gave an AI tool my three label definitions and the `analysis` vs `hot_take` rule, and asked it to label the remaining 180 posts.
- *What came back:* One label per post.
- *What I changed:* About 20 of the 180 labels were wrong. I read through the AI labels and fixed those.

**Pre-labelling disclosure:** I labelled 20 posts cold, by hand. I labelled the remaining 180 using an AI tool, then checked them and fixed about 20.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



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
