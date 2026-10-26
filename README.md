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

TakeMeter sorts posts and comments from Reddit's Miffy collector community into four kinds: people passing on buying information (`info`), people sharing their own Miffy story or opinion (`experience`), people asking the group for help (`question`), and short comments that just like or agree with a post (`appreciation`). Most of the posts are about where to find Miffy merch, what people own, and how they feel about the Miffy boom. The goal is to help a moderator or shopper quickly pull out the useful buying tips and unanswered questions from a very busy feed.

---

## Label Taxonomy

### `info`

**Definition:** Gives practical, checkable details about Miffy products or buying them (where something is sold, prices, stock, release dates, whether a seller is legit); the post is mainly about passing that information on.

**Example 1:**
> Those look like the Blue Sky Halloween Miffy candle houses! I know they were released early October at TJMaxx/Homegoods.

**Example 2:**
> Hey LA people, stop chasing that polyester from Target! There is a new Miffy pop up in Downtown LA at the Bloc!

### `experience`

**Definition:** The writer talks about their own Miffy experience, collection, or opinion (what they own or bought, how they got into Miffy, how they feel about the trend); the focus is their personal story or view, not facts for others or a request.

**Example 1:**
> Miffy is a character, anyone can start liking miffy lol I grew up with Hello Kitty, and I started to like Miffy for her aesthetics and I like to collect things.

**Example 2:**
> I bought the purple one for a friend. NGL, I want to keep it. Then I spent an hour tonight trying to figure out which one I’d want most. I ended up getting Melanie. Now I’m regretting and want the others! lol

### `question`

**Definition:** The main purpose is to ask other readers for an answer, advice, or opinion (where to buy, is it good quality, has anyone used this seller); it has a question or explicit request, even if context comes first.

**Example 1:**
> Anyone purchase from Martube gift before?

**Example 2:**
> Can I have the link for this?

### `appreciation`

**Definition:** A short comment that just likes or agrees with the post (compliments, "love this", "same", "so cute"); it adds no new facts, no personal story, and asks no question.

**Example 1:**
> Super cute!

**Example 2:**
> They’re fabulous! You’re so lucky!

### The hardest boundary

**Which two labels:** `info` and `experience`

**The decision rule I used every time:**
If the post's main point is a fact another shopper could act on (where it's sold, price, stock, whether a seller is legit), it's `info`, even when it's told in the first person ("I went to Target and they said..."). If the main point is what the writer owns, bought, or feels, and any store or price is just part of their story, it's `experience`. If a post is mainly a request, it's `question` no matter how much story comes first.

---

## The Dataset

**Where the posts came from:** Reddit posts and comments from r/miffy, mostly about buying Miffy merch (Target, TJ Maxx, HEMA, pop-ups, resellers), showing collections, and the Miffy trend.

**How I labelled them:** I labelled the first 33 cold myself. The other 167 were pre-labelled by Claude using my four definitions, and I then read and checked every one of them, correcting the ones I disagreed with. Rows are marked `cold` or `pre-labeled, checked` in the note column of `labels.csv`.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `experience` | 83 | 41.5% |
| `info` | 47 | 23.5% |
| `appreciation` | 43 | 21.5% |
| `question` | 27 | 13.5% |
| **Total** | 200 | 100% |

**Three hard cases**

**1.**
> *The post:* I went to Target yesterday, and even though the app showed that the item was in stock, the employees told me they weren’t allowed to sell it. They said they couldn’t take it out of the back room, even though the item had already been officially released and was listed as in stock. …
>
> *Could have been:* `experience` or `info`
>
> *I chose,* `info`*, because:* it's told as a personal story, but its main point is a fact other shoppers can use: the app can say in stock while the store won't sell it.

**2.**
> *The post:* To try and control my impulses (and because my husband will totally think I’m officially crazy if I buy one more Miffy) I began taking screenshots of all the interesting Miffy’s out there that I haven’t seen around much. What’s a rare Miffy you found?
>
> *Could have been:* `experience` or `question`
>
> *I chose,* `question`*, because:* most of the post is the writer's story, but it ends with a direct request to the group, so the purpose is to get answers.

**3.**
> *The post:* I’ve always loved Miffy & HK, but does anyone else feel like licensed character merchandise as a whole has become an automatic target for scalpers & has turned people into overnight fans? …
>
> *Could have been:* `experience` or `question`
>
> *I chose,* `experience`*, because:* it does ask "does anyone else feel...", but the post is mostly the writer's own opinion about scalpers and the Miffy boom, and the question is just a way to share that view, not a request for an answer.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128 tokens, seed 42

**Anything I changed from the defaults, and why:** I didn't change any of the training settings. The one thing I changed was the environment: the notebook first crashed with a `dill` pickling error because the project's virtual environment used Python 3.14, which `requirements.txt` says is not supported. I rebuilt the `.venv` with Python 3.13 and training then ran. Training uses the Apple Silicon GPU (MPS).

**Split sizes:** 139 train / 31 val / 30 test (a stratified 70/15/15 split of my 200 posts).

| Label | Train | Val | Test |
|---|---|---|---|
| `info` | 33 | 7 | 7 |
| `experience` | 58 | 13 | 12 |
| `appreciation` | 29 | 7 | 7 |
| `question` | 19 | 4 | 4 |

Three labels have fewer than about 8 examples in the test split: `info` (7), `appreciation` (7) and `question` (4). Scores for those labels will move a lot between seeds, because one wrong prediction on `question` changes its recall by 25 points.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* Help redefining my label definitions, which were one-line and overlapped (my `reaction` and `question` labels).
- *What came back:* Longer definitions based on what a post is mainly for, so there is a clear way to break ties between labels.
- *What I changed:* I renamed `reaction` to `experience`, then asked for the definition to be broadened so it covers anyone telling their own Miffy story or opinion, not just a reaction to something that just happened.

**Moment 2**

- *What I asked for:* Whether I needed another label, because many of my Reddit comments just said they liked a post.
- *What came back:* A suggestion to add `appreciation`, plus rewritten definitions for the other three that fit Miffy buying and collecting posts. It also adjusted my balance target for four labels.
- *What I changed:* I accepted `appreciation` and the new definitions, then read through the labels Claude gave my posts and corrected the ones I disagreed with.

**Pre-labelling disclosure:** I labelled the first 33 posts myself without a model. Claude pre-labelled the other 167 using my four definitions, and I read and checked every one before marking it `pre-labeled, checked` in `labels.csv`.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.300 | 0.400 | +0.100 |
| Macro F1 | 0.204 | 0.143 | -0.061 |
| F1 — `appreciation` | 0.000 | 0.000 | +0.000 |
| F1 — `experience` | 0.387 | 0.571 | +0.184 |
| F1 — `info` | 0.000 | 0.000 | +0.000 |
| F1 — `question` | 0.429 | 0.000 | -0.429 |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->
I expect the fine-tuned model to gain the most over the baseline on `experience` and `info`. Those two labels overlap the most in my data, since a lot of posts tell a personal story and also mention where something was bought or how much it cost. A model working only from my one-line definitions has no way to learn where I draw that line. A model trained on 139 posts I labelled myself can pick up the pattern: `experience` is about what the writer owns or feels, and `info` is about facts another shopper could use. For `question` and `appreciation` I expect the baseline to hold its own, or even win. Both are easy to spot straight from the definition (a request for an answer, or a short "so cute"), and the fine-tuned model has few examples of them to learn from (19 and 29 in training).

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->
Fine-tuning added very little. The trained model is ahead on accuracy (0.40 vs 0.30), but that is 3 more posts right out of 30, and it is behind on macro F1 (0.14 vs 0.20). Both models are weak. The trained model gets all of its score from `experience`, my biggest label (83 of 200 posts), where it beats the baseline (F1 0.57 vs 0.39). It scored 0 on `info`, `appreciation` and `question`, which means it mostly learned to guess the biggest label. The baseline is the only one that gets any `question` posts right (F1 0.43), but it scored 0 on `info` and `appreciation`.

This partly matches my prediction. The trained model did win on `experience` and the baseline did win on `question`. I was wrong about `info`, where neither model scored, and about `appreciation`, where I expected the baseline to do well and it scored 0. The test set is only 30 posts (4 `question`, 7 each for `info` and `appreciation`), so a single run on one seed can't tell me much. The three-seed run next unit will show whether the `experience` gain and the `question` loss are real or just a result of this one split.

---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 on every seed | 0.400 | 0.433 | 0.400 | MISSED |
| 2. Every label's F1 | ≥ 0.60 for every label | lowest 0.000 (`info`, `appreciation`, `question`) | lowest 0.000 (same three) | lowest 0.000 (same three) | MISSED |
| 3. Balance | every label 15–40% of my data | 13.5% – 41.5% | 13.5% – 41.5% | 13.5% – 41.5% | MISSED |
| 4. Consistency | ≥ 85% agreement when I relabel 30 of my own posts | not measured yet | not measured yet | not measured yet | NOT MEASURED YET |
| 5. Confidence | ≥ 0.80-confidence predictions right ≥ 85% of the time, and more often than less-sure ones | most-sure third 0.50 vs least-sure third 0.10 | 0.50 vs 0.40 | 0.40 vs 0.40 | MISSED |

Criterion 3 does not depend on the seed, because it is about my labelled data (`experience` is 41.5% and `question` is 13.5%), so the same numbers appear in every column. For criterion 5, the model's average confidence was only about 0.36–0.40 on every seed, so it almost certainly never made a 0.80-confidence prediction at all, and the "most-sure third" is the closest thing I can measure.

### Confusion matrix

Seed 42:

| true \ predicted | info | experience | appreciation | question |
|---|---|---|---|---|
| **info** | 0 | 7 | 0 | 0 |
| **experience** | 0 | 12 | 0 | 0 |
| **appreciation** | 0 | 7 | 0 | 0 |
| **question** | 0 | 4 | 0 | 0 |

**My biggest off-diagonal number, and what it means:**
The biggest number is 7: 7 real `info` posts were called `experience`, and 0 went the other way. The same 7 happened for `appreciation`, and all 4 `question` posts were called `experience` too. The model called every one of the 30 test posts `experience`, so it didn't learn the `info`/`experience` boundary or any other. It learned to guess my biggest label (83 of 200). The one-direction pattern is the model collapsing onto the biggest label, not evidence of a particular labelling problem. The agreement report can still help show whether the `info`/`experience` line is blurry in my own labels.

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
