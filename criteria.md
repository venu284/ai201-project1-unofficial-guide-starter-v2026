# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**

My 14 city-guide documents share a lot of vocabulary across different places.
Kestrelford and Marchwood appear in several guides, while Corry Lane appears in
both the Brightwater and eating guides and Corry Vale appears in its own guide
and several cross-region guides. That overlap makes it plausible that a
retriever pulls back a lexically close chunk about the wrong topic for the same
place, rather than the one that actually answers the question. I have not run
retrieval yet, so I cannot say which question that will be, only that the corpus
gives me a specific reason to expect one miss rather than zero.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**

Naming a source is not a judgment call for the system. `app.py` prints a
`Sources retrieved:` line built from the retrieved chunks' `source` fields.
That output either appears or the pipeline is broken, regardless of whether the
answer itself is right. I am setting this at 5 of 5 rather than 4 of 5 because a
miss here would point to a wiring problem, not to a hard question, and I want a
target that treats those two failure modes differently.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:**

I have not set the threshold yet, so I cannot point to a measured gap between
in-scope and out-of-scope distances. What I can say is that my corpus covers
everyday practical logistics — transport times, opening hours, cash versus
cards, hospital access — and at least one of my five out-of-scope questions
(the ibuprofen dosage one) sits in a loosely related practical/health space
rather than somewhere obviously unrelated like a Rust for-loop. That gives me
a concrete reason to expect one borderline case rather than assuming the gate
will be clean across all five.

---

## 4. Something about your chunks

<!-- YOU WRITE THIS ONE.

     How would you know if your chunks were the right size? Name something
     countable or observable.

     Examples of the right shape — don't copy these, they should come from
     what you actually saw in Milestone 3:
       - "At least 4 of 5 sampled chunks read as a complete thought, with no
          sentence cut in half at either end."
       - "No chunk is shorter than 200 characters, since anything below that
          in my corpus turned out to be a heading with no content under it." -->



Of the five chunks printed by `python app.py chunks -n 5`, at least 4 contain
one complete `##` section, with no sentence cut off at the start or end of the
chunk.

**Why this target:**

My documents are 1.4K to 2.5K characters long and organized into `##` sections
that each carry one fact, such as opening hours, transport frequency, or a price
comparison. At the 800-character default, a chunk boundary can land inside a
section and split the fact from its number. Four of five leaves room for one edge
case, a section that is genuinely longer than 800 characters, without excusing a
chunker that regularly cuts sentences in half.



---

## 5. Your choice

<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. It could be about
     speed, about refusals, about a particular kind of question your corpus
     handles badly, about source attribution being correct rather than merely
     present — anything, as long as it names a number or an observable
     outcome. -->



For questions 1, 3, 4, and 5 in `QUESTIONS` in `questions.py` — "How much
cheaper is comparable food on Corry Lane than on Brightwater's riverside
strip?", "How often do Marchwood trams run on weekdays?", "During which
months is Kestrelford's Saturday market much reduced?", and "Does the
Kestrelford bus service run on Sundays?" — the system names a source document
that contains the `expects` text in all three evaluation runs, 4 of 4
questions.

**Why this target:**

Duplication across documents is a real feature of this corpus, not a
hypothetical. It is an easy way for a system to look right while citing the
wrong file, which criterion 2, "names a source," would not catch on its own. I
set this at 4 of 4 because a cited source that does not contain the fact is a
source-attribution failure, not a hard question.



---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
