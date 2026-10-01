# The Unofficial Guide

Venu Vemuru — `city_guides` corpus

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

I chose the `city_guides` corpus: 14 structured markdown guides to a fictional
region, covering individual towns plus region-wide guides on eating, transport,
walking, seasons, and accessibility. The system splits each guide by `##`
section, embeds the chunks, retrieves the closest ones for a question, and has a
model write an answer, with the retrieved source documents printed alongside it.
It answers specific factual questions, such as how often a tram runs, when
kitchens stop serving, which months a market is reduced, or whether a bus runs
on Sundays. If the best retrieved chunk is further than the relevance cutoff
(0.6), it replies "I don't have enough information about that" instead of
guessing.

## Chunking Strategy

**Chunk size:** no fixed size — split on markdown `##` section headers.
**Overlap:** none.

`split_documents` in `chunker.py` splits each document's text on `\n(?=## )`
instead of cutting fixed-size character windows. My corpus is 14 city guides,
each organized into `##` sections that carry one self-contained fact —
opening hours, transport frequency, a price comparison — and those sections
run 174 to 711 characters, comfortably under the starter's 800-character
default. That meant the fixed-size chunker was never going to need to split a
section further; it was only ever going to risk cutting one in half at an
arbitrary character boundary, splitting a fact from the number that goes with
it. Four of my docs (`guide_eating.md`, `guide_seasons.md`, `guide_walking.md`,
`guide_regional_transport.md`) have no intro paragraph before the first `##` —
just the title line — so I fold that fragment into the first real section
instead of emitting a 23–27 character chunk that's just a heading.

I didn't change strategy partway through — the corpus read in Milestone 1
already made the section boundaries obvious as the right cut points, so this
was the first thing I tried.

## Sample Chunks

**Chunk 1** — source: `guide_accessibility.md#0` — produced by: `chunker.py::split_documents`

```
# Getting around the region with limited mobility

An honest assessment rather than a promotional one. Some of these places are
difficult and it is better to know in advance.
```

**Chunk 2** — source: `guide_corry_vale.md#5` — produced by: `chunker.py::split_documents`

```
## Where to stay

Perhaps thirty beds in the entire valley, spread across two pubs and a handful of farmhouse rooms. In summer these are booked months ahead. Camping is permitted on two marked fields and nowhere else.
```

**Chunk 3** — source: `guide_givens_mill.md#2` — produced by: `chunker.py::split_documents`

```
## Getting around

Everything is on one street along the river. The mill is at one end and the church at the other, eight minutes apart. The riverside path continues in both directions for as far as you want to walk.
```

**Chunk 4** — source: `guide_kestrelford.md#4` — produced by: `chunker.py::split_documents`

```
## What to see

The market square on a Saturday morning is the main event and has run continuously since the 1400s. The parish church has a 13th-century tower you can climb for £2. The old trackbed walk runs six miles to the next village along an easy gradient and is the best half-day here.
```

**Chunk 5** — source: `guide_pellew_sands.md#6` — produced by: `chunker.py::split_documents`

```
## When to go

June and September for the beach without the crowds. July and August are busy and the town is at its most itself, for better and worse. Winter is bleak, largely closed, and has a following among people who like that sort of thing.
```

## Sample Answer

**Question:** How often do Marchwood trams run on weekdays?

**Answer:**

```
  (best distance 0.384, cutoff 0.6)

Marchwood trams run every 8 minutes on weekdays (from guide_marchwood.md).

Sources retrieved: guide_eating.md, guide_kestrelford.md, guide_marchwood.md, guide_regional_transport.md
```

**My relevance cutoff:** `THRESHOLD = 0.6` in `config.py`.

I ran `python app.py retrieve` on my five real questions and the five
`OUT_OF_SCOPE` questions and wrote down the best (lowest) distance for each.
The groups did not overlap: the real questions ranged from 0.222 to 0.384, and
the out-of-scope ones from 0.803 to 0.975. The gap runs from 0.384 to 0.803, and
0.6 sits close to its midpoint (about 0.59), leaving roughly 0.2 of room on each
side. That margin matters more than the exact number: a real question phrased
less cleanly than mine could drift toward 0.5 and still get through, and an
off-topic question that happens to share a word with the corpus would need to
improve by 0.2 before it slipped past. I kept the starter's 0.6 rather than
moving it, because the measurement put it where I would have put it anyway.

| Question | In corpus? | Best distance |
|---|---|---|
| How much cheaper is comparable food on Corry Lane than on Brightwater's riverside strip? | Yes | 0.3549 |
| What time do kitchens outside Marchwood usually stop serving food? | Yes | 0.2907 |
| How often do Marchwood trams run on weekdays? | Yes | 0.3842 |
| During which months is Kestrelford's Saturday market much reduced? | Yes | 0.2596 |
| Does the Kestrelford bus service run on Sundays? | Yes | 0.2217 |
| What is the capital of Mongolia? | No | 0.8026 |
| How do I change the oil in a diesel engine? | No | 0.8917 |
| Who won the 1994 World Cup? | No | 0.9747 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8459 |
| How do I write a for loop in Rust? | No | 0.8130 |

## How I Used AI

**1. Pressure-testing the acceptance criteria.** I asked Claude to run my five
criteria through the self-check, saying how it would test each one using only
the sentence. Four passed. It flagged criterion 5: I had named the questions by
topic ("Corry Lane price", "Marchwood tram"), so two graders could map those
labels to different entries in `questions.py`. It had also suggested a Corry
Lane / Corry Vale mix-up criterion as an option; I did not use it and kept the
duplicated-fact idea. I rewrote criterion 5 to cite the exact question text and
the `expects` field, then had the check re-run on the new wording.

**2. Planning the chunker.** I asked Claude to plan a chunking strategy for my
corpus. It measured the `##` sections (174 to 711 characters), found four guides
whose opening fragment is only a title (23 to 27 characters), and proposed
splitting on `##` and folding any opening fragment under 60 characters into the
next section. I limited the plan to `split_documents`, left `fallback_split` and
`config.py` unchanged, and added `app.py index` and `--from-doc` to the checks.
After implementing it I ran `chunker.py` (94 chunks) and indexing, and confirmed
`guide_eating` produced 5 chunks rather than 6, which shows the title fragment
was merged. I then corrected a stale docstring that still described the old
fallback.

I did this work with Claude and used Codex to check it, so I could compare how
each tool handled the same task. Codex's review of the finished README caught
wording that did not match the code, for example that the retrieved sources are
printed by the system rather than named by the model. I made those fixes
myself.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
