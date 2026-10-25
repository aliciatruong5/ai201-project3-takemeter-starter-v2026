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
