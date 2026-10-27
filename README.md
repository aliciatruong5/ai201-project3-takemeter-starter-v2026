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
| 1 | Overall accuracy | ≥ 0.70 on every seed | MISSED | 0.400, 0.433, 0.400. All three are below 0.70, and the best seed is 0.267 short. |
| 2 | Every label's F1 | ≥ 0.60 for every label | MISSED | `info`, `appreciation` and `question` scored 0.000 on all three seeds. Only `experience` got a score (0.571, 0.605, 0.571), and even that is below 0.60 on two of three seeds. |
| 3 | Balance | every label 15–40% of my data | MISSED | `experience` is 83 of 200 (41.5%), over the 40% ceiling, and `question` is 27 of 200 (13.5%), under the 15% floor. This comes from my labelled data, so it is the same on every seed. |
| 4 | Consistency | ≥ 85% agreement when I relabel 30 of my own posts | NOT MEASURED | I haven't done the blind relabel of my own posts. I did agree with staff on 28 of 30 (93%), but that tests a different thing: applying staff's written rules to running-forum posts, not my own Miffy labels. I'm not counting it as MET. |
| 5 | Confidence | ≥ 0.80-confidence predictions right ≥ 85% of the time, and more often than less-sure ones | MISSED | The model's average confidence was only 0.36–0.40, so it almost certainly never reached 0.80. The most-sure third was right 50%, 50% and 40% of the time, and the least-sure third 10%, 40% and 40%, so it was no better on seed 2024. |

**Diagnoses**

**Criteria 1, 2 and 5: the model never learned to separate the labels (too few examples, and an unbalanced set).**
The confusion matrix on seed 42 shows the model called all 30 test posts `experience`, my biggest label. That alone explains the accuracy of 0.400, because 12 of the 30 test posts are `experience`, so always guessing it scores 12 / 30 = 0.40. The F1 of 0 on the other three labels and the weak confidence follow from the same thing. The model's validation loss also barely moved across training (about 1.32 to 1.26, against about 1.39 for pure guessing among four labels), so it hadn't learned much by the end of 3 epochs. With 139 training posts, only 19 `question` and 29 `appreciation` examples, and a skewed split, there isn't enough signal for a small model to pick up the three smaller labels.
How I know it is not a labelling problem: my agreement with staff was 93% (28 of 30), and the two disagreements were both cases where I applied a clear written rule loosely, not cases where the labels were ambiguous. The zero-shot baseline with bare label names also scored 0.667 accuracy and 0.676 macro F1 on the same 30 posts, so the four labels can be told apart. The problem is the amount of data and its balance, not that the task can't be done. The agreement set is a different taxonomy and domain, so this is indirect evidence about my own 200 labels, not proof.

**Criterion 3: balance missed because of how Miffy posts are distributed.**
The community naturally has many personal posts and few direct questions, so `experience` ends up biggest and `question` smallest. I set the ceiling and floor before collecting. `experience` is over by 1.5 points and `question` is under by 1.5 points, which is a small miss in my own data, not a labelling error. The agreement report doesn't point to a consistency problem, but my two pile (a) disagreements suggest I should re-check the `info`/`experience` boundary in my own labels, since a mislabelled `info` post that I called `experience` would add to the `experience` count.

**Criterion 4: not measured.**
I need to relabel 30 of my own posts without looking at my earlier labels and then compare them. Until I do, I can't say MET or MISSED.

---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 28 / 30 = 93%

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
> *The post:* Carbon plates aren't the whole story. My times dropped 2% in the shoes and another 2% the following year in the same pair, when the only thing that changed was going from 35 to 50 miles a week.
>
> *Staff said / I said:* `analysis` / `hot_take`
>
> *My call, and why:* Staff is right. Rule 1 asks whether the checkable fact carries the argument. Here it does: the claim "plates aren't the whole story" rests on the 2% drops and the 35 → 50 miles a week, and without those numbers there is no argument. I read the opening line as a confident claim and stopped there, instead of asking whether the numbers were load-bearing or decoration.
>
> *Which pile:* (a) the rule covered it and I applied it loosely.

**2.**
> *The post:* Every injury thread on this forum is the same injury and the same three replies.
>
> *Staff said / I said:* `hot_take` / `reaction`
>
> *My call, and why:* Staff is right. Rule 2 says `reaction` reports what just happened to the writer and how it felt, and `hot_take` asserts something general about the world. This post says nothing about something that happened to the writer or how they felt. It claims something general about every injury thread, with no support, and the rule says a heated post can still be a `hot_take`. I let the annoyed tone decide, when the rule says to ask whether the post is about a claim or about the writer.
>
> *Which pile:* (a) the rule covered it and I applied it loosely.

**What the pattern in my disagreements tells me:**
I disagreed with staff on 2 of 30 posts, and both were pile (a): the written rule covered the case and I applied it loosely. In both, a surface feature of the post (a confident opening line in the first, an annoyed tone in the second) outweighed the test the rule actually gives. I didn't win any, and neither is a gap in the definitions. That's evidence about my own 200 labels: I probably did the same on my Miffy posts, especially on the `info`/`experience` boundary, where a first-person story can hide a fact other shoppers could use, and my own decision rule says the fact decides. It also lines up with the confusion matrix, where `info` and `experience` are the pair the model couldn't separate. 28 of 30 is a high agreement rate, so I don't think my labelling was careless, but it suggests I should re-check my `info` and `experience` labels against the decision rule instead of the tone of the post.

---

## The Improvement

**What I changed:** One training setting: epochs, from 3 to 10. Learning rate (2e-5), batch size (16), max length (128), the data and the labels stayed the same. No rows in `labels.csv` changed for this run.

**Which diagnosis pointed at it:** My diagnosis for criteria 1, 2 and 5 said the model never learned to separate the labels and was undertrained: it called every test post `experience`, and its validation loss barely moved over 3 epochs (about 1.32 to 1.26, against about 1.39 for pure guessing among four labels). The agreement report ruled out a labelling problem (28 of 30, 93%), so a training setting was the cheapest thing to test first. The other real cause was too few examples for the small labels (only 19 `question` in training), and 10 epochs does nothing about that.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 on every seed | 0.533 | 0.467 | 0.500 | MISSED |
| 2. Every label's F1 | ≥ 0.60 for every label | lowest 0.000 (`question`) | lowest 0.000 (`appreciation`, `question`) | lowest 0.000 (`question`) | MISSED |
| 3. Balance | every label 15–40% of my data | 13.5% – 41.5% | 13.5% – 41.5% | 13.5% – 41.5% | MISSED (unchanged, I didn't touch the data) |
| 4. Consistency | ≥ 85% agreement when I relabel 30 of my own posts | not measured yet | not measured yet | not measured yet | NOT MEASURED |
| 5. Confidence | ≥ 0.80-confidence predictions right ≥ 85%, and more often than less-sure ones | most-sure third 0.80 vs least-sure third 0.30 | 0.70 vs 0.50 | 0.70 vs 0.40 | MISSED |

Per-label F1 after the change (before in brackets):

| Label | Seed 42 | Seed 7 | Seed 2024 |
|---|---|---|---|
| `info` | 0.62 (0.00) | 0.50 (0.00) | 0.44 (0.00) |
| `experience` | 0.58 (0.57) | 0.59 (0.61) | 0.62 (0.57) |
| `appreciation` | 0.50 (0.00) | 0.00 (0.00) | 0.25 (0.00) |
| `question` | 0.00 (0.00) | 0.00 (0.00) | 0.00 (0.00) |

**Did it help, and how do I know:**
It helped, but not enough to meet any target. Accuracy went from 0.400 / 0.433 / 0.400 to 0.533 / 0.467 / 0.500 and macro F1 from about 0.14–0.15 to 0.27–0.42, on all three seeds. The model also stopped calling every post `experience`: on seed 42, `info` now gets 4 of 7 right (it got 0 of 7), and `appreciation` gets 3 of 7. The most-sure third of predictions is now right more often than the least-sure third on every seed (0.80 vs 0.30, 0.70 vs 0.50, 0.70 vs 0.40), where before seed 2024 was 0.40 vs 0.40. That part of criterion 5 now holds, but the 0.80-confidence part does not, because the model's average confidence when right is only about 0.50.
It didn't fix `question`, which scored 0.00 on all three seeds: on seed 42 its 4 test posts were called `experience` (3) and `info` (1). That fits my diagnosis, because training longer can't create examples that aren't there, and I have only 19 `question` posts in training. The gap between seeds is also wide (accuracy 0.467 to 0.533, `appreciation` F1 from 0.00 to 0.50), and the test set is only 30 posts, so the improvement is real on average but I wouldn't trust any single seed's number. I haven't tried any other epoch count or learning rate, so 10 isn't shown to be the best setting.

---

## What's Still Broken

**Criterion 1, overall accuracy (≥ 0.70): MISSED, best seed 0.533.** I would collect more posts, especially for `question` and `appreciation`, and then try a few epoch counts and learning rates instead of only 10 epochs. I stopped because the diagnosis points at too little data, and I can't fix that by changing a setting. I only got one improvement in this unit, and it moved accuracy about 10 points, which isn't close to the 17 I still need.

**Criterion 2, every label's F1 (≥ 0.60): MISSED, `question` is 0.00 on all three seeds.** I would add about 30 or more `question` posts from r/miffy, labelled with the same definition, because the model has only 19 `question` examples in training and its 4 test posts were all called other labels. I stopped because collecting and labelling that many posts would be a whole second round of data work, and training longer can't create examples that aren't in the data.

**Criterion 3, balance (every label 15–40%): MISSED.** `experience` is 41.5% and `question` is 13.5%. I would collect more `question` posts, which would fix the `question` share and pull `experience` below 40% at the same time. I didn't change this because it can't be fixed by relabelling: my labels follow my definitions, so moving rows between labels just to hit the percentage would mean changing labels I believe are right. I'm not going to lower the target either.

**Criterion 4, consistency (≥ 85% when I relabel my own posts): NOT MEASURED.** I would relabel 30 of my own posts without looking at my first labels and compare. I haven't done it. The 93% agreement with staff and the fact that I changed only 2 of 167 pre-labels are both encouraging, but neither is the test I wrote down.

**Criterion 5, confidence (≥ 0.80-confidence predictions right ≥ 85%): MISSED.** The part about confident guesses being more accurate than unsure ones now holds on all three seeds, but the model's average confidence when right is only about 0.50, so it never makes the 0.80 predictions the target is about. I would train on more data, since a model with so few examples isn't going to be sure of itself. I stopped for the same reason as criterion 1.

**The gap between what I meant my labels to capture and what the model learned:**
I meant `info` (a fact other shoppers can use) and `experience` (the writer's own story) to be different kinds of posts, but on seed 42 the model still called 3 of 7 `info` posts `experience`, and 4 of 7 `appreciation` posts and 3 of 4 `question` posts `experience` too. It learned "personal and first-person means `experience`" much better than the actual decision rules, so the smaller labels are mostly being absorbed into my biggest one.

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
