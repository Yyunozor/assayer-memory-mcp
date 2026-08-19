← [assayer](../README.md) · [the verdict funnel](funnel.md) · [the reading
surface](brief.md)

# Retrieval

**Frozen 2026-07-29.** A dated scorecard for the retrieval path, measured on a
frozen corpus and a frozen question set. Numbers change only by re-freezing this
page with a new date.

The column that matters most is `grep`.

---

## The short version

A tokenized substring search — no index, no embeddings, no ranking model —
finds a correct note for **75.0%** of the benchmark questions. The dense
embedding arm finds one for **79.2%**. That gap is **one question out of 24**,
and a paired exact test puts it at **p = 1.00**.

The hybrid arm as measured here scores **87.5%**, three questions ahead of
substring search, at **p = 0.45**.

**That is the bench arm, not the served path.** The shipped retriever drops
index notes from the candidate set; this harness does not. One benchmark
question is answered only by an index note, so on the served path its ceiling
is 23/24 = 0.958 rather than 1.000, and the hybrid figure would fall to at most
0.833. The gap is a property of the deployment, not of the algorithm, and it is
not measured on this page.

**No arm on this page is statistically separated from substring search.** The
benchmark is too small to separate them, which is stated here rather than
omitted.

---

## What the benchmark *can* show (added 2026-07-31)

The section above says what cannot be concluded. Stating only that is safe for
the writer and useless for the reader, so here is the other half.

**The resolution floor is 4.17 points.** With 24 questions the smallest
representable difference on hit@k is 1/24. Any gap below that is not a gap,
and the +12.5 pt figure above is three questions — a fact worth holding in
mind before it is quoted as a margin.

**The two halves of the verdict do not overlap.** The retrieval numbers cover
all 24 questions; the model-judged pass covers a subset of 8. The hybrid arm's
entire advantage over the dense arm comes from two questions — **and neither
was judged.** On the eight questions that *were* judged, the two arms score
identically, question by question. A previous write-up read this as three
signals agreeing; they are two disjoint measurements that share a page.

**One effect has been demonstrated, and it is not the one anyone expected.**
Under a paired exact test, swapping the embedding model is the only change
this benchmark has ever separated from noise (7 wins, 0 losses, 17 ties,
p = 0.0156) — and the direction says *keep the current one*. Every other
comparison on this page is inside sampling noise: hybrid vs. dense p = 0.50,
hybrid vs. substring p = 0.45.

**The fused path is justified by coverage, not by ranking.** Measured on the
production index rather than the frozen corpus, with 40 stratified targets:
the mean-reciprocal-rank difference between the fused and lexical paths is
−0.017 with a 95 % interval of [−0.170, +0.136] — undecided, and the sample is
too small to decide it. What *is* solid is that the two retrievers fail on
different questions. Union recall@10 reaches **0.775**, against 0.550 for BM25
alone and 0.525 for dense alone. The dense arm recovers 22.5 points that
lexical search does not reach. That is the argument for running both.

**There is no usable confidence threshold.** The fused score is a sum of two
reciprocal ranks, bounded at 2/61 and independent of how well anything
matched; as a signal for "is there an answer at all" it scores AUC 0.594,
which is noise wearing a number. The raw dense cosine does better (AUC 0.861
overall, 0.772 on the hard case of a query whose answer is *nearly* present)
but no cutoff separates the two populations usefully: a threshold that rejects
90 % of answerless queries also discards 62.5 % of real ones. Both retrievers
therefore return five confident results for a question the corpus cannot
answer. Knowing when to say nothing remains unsolved here, and is not
presented as solved.

---

## The ceiling, before the scores

Every score below is bounded by the shape of the benchmark, not by the quality
of retrieval. Reading the table without this section produces the wrong
conclusions in both directions.

| Bound | Value | Why |
|---|---|---|
| Questions | 24 | 26 frozen, 2 excluded with dated reasons — see caveat below |
| Notes retrieved per question (k) | 4 | |
| Max attainable hit@4 | **1.000** | every question has at least one gold note present in the corpus — verified, not assumed |
| Max attainable precision@4 | **0.354** | 17 of the 24 questions have exactly one gold note, so their precision@4 cannot exceed 0.25 whatever is retrieved |
| Max attainable recall@4 | **0.992** | one question has 5 gold notes and k is 4 |
| Smallest representable difference | **4.17 pp** | one question. Any gap smaller than this does not exist as a measurement |

**One exclusion does not hold against the frozen corpus.** Of the two excluded
questions, one was dropped because its only gold note had been archived out of
the vault. That is true of the live vault and false of the frozen corpus pinned
by the hashes below, where the note is present. The question was excluded the
day after the corpus was frozen, so the active set mixes two vault generations.
Including it would give `grep` 19/25 = 0.760 and move the precision ceiling to
0.350. Stated here rather than silently corrected, because re-freezing the page
is a dated act and this one has not been performed.

**What 24 questions can and cannot resolve.** Because both arms answer the same
questions, the comparison is paired: only the questions where the two arms
disagree carry information. At this size, roughly **six disagreements all in the
same direction** are needed to reach p < 0.05. Observed disagreement counts are
between 7 and 9 questions total, split in both directions.

To separate the hybrid arm from substring search at its observed effect size,
with 80% power, would take about **141 questions**; the plain dense arm, about
**1300**.

Read those two as orders of magnitude, not as figures. Both are estimated from
**seven discordant pairs**, and one question changing sides moves 141 to either
45 or 1314. The formula is also blind to direction: an arm three questions
*worse* than `grep` returns the same 1314. What they establish is that the gap
is far outside what 24 questions can resolve — not how far.

---

## Scorecard

Retrieval only. Deterministic, no model in the loop, no network, no spend — the
same inputs produce the same table every time.

`hit@4` is the share of questions for which at least one gold note appears in
the top 4.

### All questions (n = 24)

| Arm | hit@4 | precision@4 | recall@4 | vs `grep`, paired | p |
|---|---|---|---|---|---|
| **`grep` — tokenized substring, no index** | **0.750** | 0.208 | 0.681 | — | — |
| Dense cosine over local embeddings | 0.792 | 0.229 | 0.697 | +4 / −3 / =17 | 1.00 |
| Dense + hierarchy rerank | 0.708 | 0.208 | 0.635 | +4 / −5 / =15 | 1.00 |
| Kind-routed | 0.500 | 0.125 | 0.431 | +1 / −7 / =16 | 0.07 |
| **Hybrid BM25 + dense (RRF, k=60)** | **0.875** | 0.250 | 0.781 | +5 / −2 / =17 | 0.45 |
| *Attainable maximum* | *1.000* | *0.354* | *0.992* | | |

`+w / −l / =t` reads: questions the arm wins against substring search, questions
it loses, questions where both hit or both miss.

### Broad questions (n = 9)

| Arm | hit@4 | precision@4 | recall@4 |
|---|---|---|---|
| **`grep`** | **0.667** | 0.167 | 0.481 |
| Dense cosine | 0.889 | 0.250 | 0.637 |
| Dense + hierarchy rerank | 0.778 | 0.222 | 0.581 |
| Kind-routed | 0.111 | 0.028 | 0.037 |
| Hybrid BM25 + dense | 0.889 | 0.250 | 0.637 |
| *Attainable maximum* | *1.000* | *0.472* | *0.978* |

### Narrow questions (n = 15)

| Arm | hit@4 | precision@4 | recall@4 |
|---|---|---|---|
| **`grep`** | **0.800** | 0.233 | 0.800 |
| Dense cosine | 0.733 | 0.217 | 0.733 |
| Dense + hierarchy rerank | 0.667 | 0.200 | 0.667 |
| Kind-routed | 0.733 | 0.183 | 0.667 |
| Hybrid BM25 + dense | 0.867 | 0.250 | 0.867 |
| *Attainable maximum* | *1.000* | *0.283* | *1.000* |

**On narrow, fact-shaped questions, substring search beats the dense arm**
(0.800 against 0.733). Semantic similarity earns its place on broad questions
(0.667 against 0.889) and nowhere else on this corpus.

One more result worth stating because it is unflattering to the metric itself:
substring search and the dense arm **never return the same four notes** on any
of the 24 questions, and share on average 1.29 of 4. Two arms retrieving almost
disjoint material score the same. `hit@4` at n = 24 is not resolving what it
looks like it is resolving.

---

## Why `grep` is a permanent column

Substring search is registered in the harness as a first-class arm and runs by
default in every evaluation. It needs no index, no embeddings and no model, so
it cannot quietly stop being available.

The rule it enforces: **a retrieval change has to beat substring search to
ship.** An arm that does not beat `grep` is not paying for its index, its
embedder, its dependency, or the code it adds.

It is deliberately naive, and stays that way: no inverse-document-frequency
weighting, no length normalisation, no stopword list, no stemming. Each of
those would raise its score and make the comparison flattering by construction.
Its known bias is left uncorrected and stated here instead — without length
normalisation, a long note is mechanically more likely to contain any given
term.

---

## What is measured

**Retrieval, and nothing downstream.** These are retrieval scores against gold
note sets. They are not question-answering accuracy, and they do not describe
the quality of any answer written from the retrieved material.

**The corpus.** 241 notes — the vault as it stood on the freeze date, and a
fraction of what the front page reports today. The gap is not an error on
either side: this page is pinned to the corpus below, and unpinning it is a
dated act that has not been performed. Every score here therefore describes a
smaller and older vault than the one the retriever now serves.

Its shape: 933,466 characters of body text, 3,873 characters per
note on average. 161 leaf notes, 38 synthesis notes, 30 concept notes, 12 index
notes. 13 themes, unevenly filled: the largest holds 53% of the notes (42% of the
characters), and nine of the 13 hold five notes or fewer. Notes are embedded in 1,200-character
sliding windows with 200 characters of overlap, and a note scores as its best
window.

**The questions.** 26 frozen, 2 excluded with dated reasons recorded in the
question file, 24 active: 9 broad, 15 narrow, spanning 10 themes. 17 have a
single gold note, 5 have two, one has three, one has five.

**The embeddings.** A local ONNX model (`paraphrase-multilingual-MiniLM-L12-v2`
via fastembed). No API, no key, no bill.

---

## Reproduction

One command replays the whole table:

```
.venv/bin/python3 eval_retrieval_only.py --out /tmp/rerun.json
```

The virtual environment is not optional — the dense arms load a local model, and
the bare `python3` form fails on a clean interpreter. `--out` is not optional
either: without it the run overwrites the very artefact this page's numbers were
read from, and that file is not under version control.

It loads the frozen corpus and the frozen question set, runs every registered
arm including `grep` — six at the time of writing, five of which are tabled
above; the sixth (graph expansion) was added after this page was frozen and
scores 0.667 — computes the ceiling above, runs the paired tests, and
writes a JSON artefact with per-question detail. It needs the project's virtual
environment, because the dense arms load a local embedding model. The `grep`
column needs nothing at all and is recomputed from scratch, in about half a
second, every time the daily check on this page runs.

The two frozen inputs, so a future run can be confirmed to be the same
measurement and not a different one:

```
corpus.jsonl               sha256 3253554c342a35bb6a11313f06443d5c8400f42d42400ca297c508671cf7db85
questions.frozen.v2.yaml   sha256 4335185c574e8156fca58063d53725259b222cf242945046d6c677199424f061
```

These are fingerprints, not keys: a SHA-256 cannot be reversed, and one
changed byte changes all of it. They pin which inputs produced the numbers
above, so a quieter re-run on a friendlier corpus would be visible.

**They are not third-party verifiable, and that is worth stating.** Neither
file is published here — the vault is not in this repository. The hashes bind
us, not you: a re-freeze becomes a dated, visible act. An outside reader
cannot recompute them.

Both files are pinned at commit `98eea5d` in the working repository, which is
where the harness currently lives.

**What is not reproducible by a third party, and will not become so.** The
corpus is one person's private notes. It cannot be published, so nobody outside
can re-run this table on this data. That is a permanent limit of this page, not
a packaging delay. What a reader can check without the corpus is the shape of
the result: a substring baseline is tied with the dense arm, and the page says
so.

---

## Limits

Read these as part of the numbers, not as a disclaimer after them.

1. **The questions and their gold note sets were written by a model**, under
   delegation, and have never been line-edited by a human.
2. **The planned human spot-check has not been performed.** Not on this run and
   not on any of the runs before it.
3. **Model-written questions inherit the corpus's own vocabulary**, which
   structurally favours literal matching. The `grep` column is probably
   flattered by this. So is every other arm, to an unknown and unmeasured
   degree.
4. **One corpus, one author, 241 notes.** Nothing here says anything about how
   these arms behave on a different corpus, a larger one, or one written by
   several people.
5. **One domain family, unevenly sampled.** The largest theme is 53% of the
   corpus by note count; nine themes hold five notes or fewer. A routing or reranking strategy
   has almost nothing to work with outside the dominant theme, which caps what
   those arms could have demonstrated.
6. **n = 24.** See the ceiling section. No arm here is separated from substring
   search at p < 0.05.
7. **No third-party system is measured or compared.** Every number on this page
   is about arms in one harness on one corpus. Published scores from other
   projects are not reproduced here and are not comparable to these.

---

## What would make this page mean more

More questions, in this order of value: about 140 to separate the shipped arm
from substring search at the observed effect size; questions written by a human
rather than a model; gold sets verified by a human; and a second corpus with a
different author.

Until then this page says one thing, and it is deliberately a small thing: on a
small private corpus, a substring search is a serious opponent, and the
retrieval work here has not yet proven otherwise.
