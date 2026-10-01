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
| 1. Retrieved chunk contains the answer | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 |  |
| 2. Every answer names a source | 5 of 5 | 5 of 5 | 5 of 5 | 5 of 5 |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 |  |
| 4. Sampled chunks are complete `##` sections | 4 of 5 | 4 of 5 | 4 of 5 | 4 of 5 |  |
| 5. Cited source contains the fact (4 named questions) | 4 of 4, all 3 runs | 4 of 4 | 4 of 4 | 4 of 4 |  |

How I counted each row:

- **1.** Retrieval is deterministic, so the same chunks come back every run. For each question I checked that its `expects` text appears in one of the five retrieved chunks: `a third less`, `9pm`, `every 8 minutes`, `November to February`, `does not run on Sundays`. All five do.
- **2.** Every one of the 15 answers (5 questions, 3 runs) names a source file in its own text. The refusals in row 3 are not answers, so they have no source.
- **3.** One deterministic pass, so the same value in all three columns: the gate refused all five `OUT_OF_SCOPE` questions.
- **4.** Also deterministic. Of the five chunks printed by `python app.py chunks -n 5`, chunks 2 to 5 are each one complete `##` section. Chunk 1 (`guide_accessibility.md#0`) is a title and intro paragraph with no `##` heading, so I did not count it, even though no sentence in it is cut off.
- **5.** I scored on the files each answer cites in its own text, and checked each one's source file for the `expects` text: Corry Lane price cites `guide_eating.md` and `guide_brightwater.md` (both contain `a third less`); trams cites `guide_marchwood.md` (`every 8 minutes`); market cites `guide_eating.md` and `guide_kestrelford.md` (`November to February`); bus cites `guide_regional_transport.md` (`does not run on Sundays`). This held in all three runs.

Real output, produced by `run_eval.py::main` (results file
`results/run_2026-09-30_2024_before.md`, chunks from
`chunker.py::split_documents`, cutoff 0.6, top-k 5). Run 1 of each question:

```
How much cheaper is comparable food on Corry Lane than on Brightwater's riverside strip? — run 1
- Best distance: 0.3549 (passed the gate)
- Sources retrieved: guide_brightwater.md, guide_eating.md, guide_pellew_sands.md, guide_regional_transport.md

Comparable food on Corry Lane costs about a third less than on Brightwater's riverside strip (guide_eating.md and guide_brightwater.md).

What time do kitchens outside Marchwood usually stop serving food? — run 1
- Best distance: 0.2907 (passed the gate)
- Sources retrieved: guide_brightwater.md, guide_eating.md, guide_elder_ness.md, guide_kestrelford.md, guide_marchwood.md

Kitchens across the region outside Marchwood stop serving at 9pm and often earlier (guide_eating.md, guide_brightwater.md).

How often do Marchwood trams run on weekdays? — run 1
- Best distance: 0.3842 (passed the gate)
- Sources retrieved: guide_eating.md, guide_kestrelford.md, guide_marchwood.md, guide_regional_transport.md

Marchwood trams run every 8 minutes on weekdays.

Source: guide_marchwood.md

During which months is Kestrelford's Saturday market much reduced? — run 1
- Best distance: 0.2596 (passed the gate)
- Sources retrieved: guide_eating.md, guide_kestrelford.md, guide_regional_transport.md, guide_seasons.md

Kestrelford's Saturday market is much reduced from November to February (from `guide_eating.md` and `guide_kestrelford.md`).

Does the Kestrelford bus service run on Sundays? — run 1
- Best distance: 0.2217 (passed the gate)
- Sources retrieved: guide_brightwater.md, guide_eating.md, guide_givens_mill.md, guide_regional_transport.md

No, the Kestrelford service does not run on Sundays (guide_regional_transport.md).
```

Criterion 3, the relevance gate on the out-of-corpus questions, from the same
results file (produced by `run_eval.py::check_out_of_scope`):

```
Produced by `run_eval.py::check_out_of_scope`, cutoff 0.6. Refused 5 of 5.

| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.803 | refused |
| How do I change the oil in a diesel engine? | 0.892 | refused |
| Who won the 1994 World Cup? | 0.975 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.846 | refused |
| How do I write a for loop in Rust? | 0.813 | refused |
```

Criterion 4, the exact output of `python app.py chunks -n 5` (chunks produced
by `chunker.py::split_documents`):

```
94 chunks total. Showing 5, spread across the corpus.

======================================================================
Chunk 1  |  source: guide_accessibility.md#0  |  produced by: chunker.py::split_documents
======================================================================
# Getting around the region with limited mobility

An honest assessment rather than a promotional one. Some of these places are
difficult and it is better to know in advance.

======================================================================
Chunk 2  |  source: guide_corry_vale.md#5  |  produced by: chunker.py::split_documents
======================================================================
## Where to stay

Perhaps thirty beds in the entire valley, spread across two pubs and a handful of farmhouse rooms. In summer these are booked months ahead. Camping is permitted on two marked fields and nowhere else.

======================================================================
Chunk 3  |  source: guide_givens_mill.md#2  |  produced by: chunker.py::split_documents
======================================================================
## Getting around

Everything is on one street along the river. The mill is at one end and the church at the other, eight minutes apart. The riverside path continues in both directions for as far as you want to walk.

======================================================================
Chunk 4  |  source: guide_kestrelford.md#4  |  produced by: chunker.py::split_documents
======================================================================
## What to see

The market square on a Saturday morning is the main event and has run continuously since the 1400s. The parish church has a 13th-century tower you can climb for £2. The old trackbed walk runs six miles to the next village along an easy gradient and is the best half-day here.

======================================================================
Chunk 5  |  source: guide_pellew_sands.md#6  |  produced by: chunker.py::split_documents
======================================================================
## When to go

June and September for the beach without the crowds. July and August are busy and the town is at its most itself, for better and worse. Winter is bleak, largely closed, and has a following among people who like that sort of thing.
```

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunks contain the answer | MET | Each of the three runs scored 5 of 5, which exceeds the 4-of-5 target. |
| 2 | Every answer names a source | MET | All 15 generated answers named at least one source document, meeting the 5-of-5 target in every run. |
| 3 | Gate stops out-of-corpus questions | MET | The deterministic gate refused all 5 out-of-scope questions, exceeding the 4-of-5 target. |
| 4 | Sampled chunks are complete sections | MET | Four of the five fixed sample chunks were complete `##` sections with no sentence cut off, exactly meeting the 4-of-5 target. |
| 5 | Cited source contains the fact | MET | For all four named questions, every run cited a document containing the expected fact, meeting the 4-of-4 target across all three runs. |

## Diagnoses

No criteria were missed in the before run, so there is no failed pipeline stage
(loading, chunking, embedding, retrieval, or generation) to diagnose. I am not
going to invent a failure to fill this section.

The finding I do have is that at least one target looks conservative.
Criterion 1 was set at 4 of 5: at least 4 of my 5 questions should have a
retrieved chunk that contains the answer. It scored 5 of 5 in all three runs.
The answer chunk was also near the top each time: it ranked first for four of
the questions and second for the kitchen-hours question, where the Marchwood
chunk (which says kitchens serve until 10:30pm) ranked just ahead of the
`guide_eating.md` chunk that contains the `9pm` answer. If I wrote this
criterion again, I would set the target to 5 of 5, because this corpus has
clearly headed sections and all five factual questions retrieved the supporting
section consistently.

This is an observation, not a revision. Criterion 1 in `criteria.md` stays
exactly as I wrote it before I had results.

## The Improvement

**What I changed:** I added hybrid reranking to `store.py::search`. Semantic
search still returns its top five chunks. I then score those same five with
BM25 keyword matching, built over all 94 chunks so word rarity reflects the
whole corpus, and reorder them by `0.7 * cosine similarity + 0.3 * BM25`, each
min-max normalised across the five. Every result keeps its raw cosine distance,
so `gate.py` and the 0.6 cutoff are untouched. I fixed the 0.3 weight before
running the after evaluation and did not tune it on my test questions.

**Why I picked it:** The one finding in Diagnoses is that for the kitchen-hours
question the Marchwood chunk (10:30pm, the exception) ranked first and the
`guide_eating.md` chunk that holds "outside Marchwood ... 9pm" ranked second, a
cosine gap of only 0.0019, and exact words like "outside" and "stop serving"
should separate two chunks that embeddings treat as near-identical. I rejected
raising `TOP_K` (the answer chunk was already in the top five, so it does not
touch ordering) and re-chunking (section chunks already kept every answer
intact).

### Run Log — After

Produced by `run_eval.py::main`, results file
`results/run_2026-09-30_2043_after.md`, same five criteria and three runs.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 2. Every answer names a source | 5 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Sampled chunks are complete `##` sections | 4 of 5 | 4 of 5 | 4 of 5 | 4 of 5 | MET |
| 5. Cited source contains the fact (4 named questions) | 4 of 4, all 3 runs | 4 of 4 | 4 of 4 | 4 of 4 | MET |

Rank of the chunk containing the `expects` text, before and after (position in
the five returned chunks, from `search`):

| Question | Rank before | Rank after |
|---|---|---|
| Corry Lane price | 1 | 1 |
| Kitchens outside Marchwood | 2 | 1 |
| Marchwood trams | 1 | 1 |
| Kestrelford market | 1 | 1 |
| Kestrelford Sunday bus | 1 | 1 |

**Did it help?** Not on the five criteria, and it could not have: reordering
the same five chunks leaves criterion 1 ("any of the five"), the gate (which
uses the minimum distance), the chunks, and the out-of-scope refusals
unchanged, and all five rows came out exactly as in the before run, with all
cosine distances identical. It did what I aimed it at: the kitchen-hours chunk
that holds the 9pm answer moved from rank 2 to rank 1, and the other four
answer chunks stayed at rank 1, so nothing regressed. The effect on the final
answers is small to none. The kitchen answer was already correct 3 of 3 before
because the prompt includes all five chunks, and the after answers read the
same and cite `guide_eating.md`. The change would matter more with a larger
`TOP_K` or a long prompt, where order decides what the model sees, and I did not
test that.

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
