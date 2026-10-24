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

Overall accuracy is at least 0.70 on the held-out set, on every one of the three seeds (42, 7, 2024).

**Why this target:**

With three labels a coin-flip gets about 33%, and always guessing the biggest label gets maybe 40%. 0.70 is roughly double chance and means about 7 in 10 posts are routed correctly, which is where I'd call a forum-post sorter useful. I'm not going higher because my labels (`info`, `experience`, `question`) overlap at the edges, e.g. a post that tells a story and ends with a question, so some disagreement is built into the taxonomy.

---

## 2.

Every label has an F1 of at least 0.60 on the held-out set, on all three seeds.

**Why this target:**

Accuracy can hide one label failing. Each label has only about 8-10 test examples, so one wrong prediction swings its F1 a lot. That is why this floor is lower than the overall target. I expect `question` to be easiest (it often has a question mark) and `experience` vs `info` to be the hard pair.

---

## 3.

No label has fewer than 25% or more than 45% of my labelled examples (about 50-90 of 200).

**Why this target:**

A skewed split lets the model score well by favouring the big label, and a label with few examples gets a noisy F1. Three labels would be 33% each if even. This range allows natural imbalance (people post more questions and stories than fact dumps) without letting any label become a rounding error.

---

## 4.

When I relabel a random 30 of my own posts without looking at my earlier labels, I agree with myself on at least 85% (26 of 30).

**Why this target:**

If I can't match my own labels, the model has no consistent signal to learn, and its accuracy ceiling is my agreement rate. 85% is below 100% because of the `experience`/`info` and `question`/`experience` borderline posts, but low enough to expose a definition that is too vague. This also checks my definitions against the staff-set exercise.

---

## 5.

Predictions with confidence of at least 0.80 are correct at least 85% of the time, and that rate is higher than for predictions below 0.80, on all three seeds.

**Why this target:**

A confidence score is only useful if it means something. If high-confidence predictions are no more accurate than low-confidence ones, I can't use confidence to send uncertain posts to a human. 85% is deliberately above the overall 0.70 target, because the confident subset should be the more reliable one.

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
