← [assayer](../README.md) · [the reading surface](brief.md) · [retrieval,
measured](retrieval.md)

# The verdict funnel

A knowledge base that only accumulates is a backlog with a search bar. This is
the machinery that drains it — three stages, each with a different cost and a
different authority, and a hard rule that only the last one may write.

The measured problem, 2026-08-07: 258 items in the queue, 240 of them inbox
notes, median age 8 days. The owner had rendered **one verdict in two days**.
The bottleneck was not will, it was arithmetic — emptying the queue by hand
meant 258 readings.

---

## Stage 1 — triage, $0

No model, no network, and no write into the vault at all. It partitions the
queue and derives signals; it never judges.

**The maturation threshold is 5 days**, and the number is derived, not chosen.
The synthesis pass groups 2–5 similar notes into a trend note and archives the
sources it absorbs; it runs on every ingestion cycle, four times a day. A note
younger than five days has had roughly twenty chances to leave on its own —
disturbing a human for it is work that did not need to exist. Past that, a note
still in inbox is a **durable singleton**: nothing will cluster it, and only a
verdict moves it.

Measured on the real queue that day: 78 notes aged 0–4 against 162 aged 5+. A
third of the queue leaves without anyone reading a line, and without risk — a
maturing note is not lost, it returns on the next run.

**Notes tagged `etincelle` leave in their own partition**, whatever their age.
The threshold's reasoning — five days unabsorbed means durable singleton, so a
case to settle — is false for them: a spark ages on purpose, waiting to be
connected, and its age is evidence that it held, not that it died. Stage 2
reads only the `mures` partition, so the exemption holds **by the shape of the
output**, not by an instruction in a prompt that a model can ignore.

## Stage 2 — recommendation, batched

Reads the mature partition, writes one file, and calls a model. It does not
call the write path, does not touch the vault, and changes no note's status.
Two ratchets make the violation impossible rather than unlikely: every write
goes through a guard that refuses any path resolving inside the vault, and
every subprocess goes through a guard that refuses any command line mentioning
the write path or `--apply`. Both are tested **by mutation** — breaking a guard
must redden a test, or the guard is wallpaper.

Three independent bounds hold the cost, each for a different drift:

| Bound | Value | What it stops |
|---|---|---|
| Batch size | 12 notes per call | title, claims and signals only — **never the body**, so a batch is ~3–4k input tokens and the price is predictable |
| Cache | keyed on a byte hash of the note | a note already recommended and unchanged is never paid twice; a re-run on an unchanged queue is 0 calls, $0 |
| Ceiling | $3.00 per run, checked cumulatively *before* each call | reached → clean stop, exit 0; unrecommended notes simply carry to the next run |

Excluding the body is not only a price decision: it forces the model to doubt.
Low confidence is mandatory when title and claims do not suffice.

A separate circuit breaker handles the failure that is not a cost: three
consecutive call failures, or a stderr signature meaning expired session or
quota, stops the run. A failed batch is never a lost batch: its notes stay eligible and are retried on the next run, cache intact. Failure here is expensive in
nothing but latency, which is the point of putting the ceiling before the call
rather than after it.

## Stage 3 — ratification, human

Dry-run by default. Without `--apply` the command lists and stops: it opens no
file for writing, takes no vault lock, and does not call the write path even in
dry-run. That is proved by the **absence of a call**, not by an intact vault —
a write that fails leaves the same vault as a write never attempted.

**It runs in no pipeline, no scheduled job, no git hook, no cron.** Not an
intention — a property: a repo-wide test sweeps the scheduled jobs, shell
scripts, hooks and Python modules and fails if any of them names this file. The
day someone wires it into ingestion, the suite reddens before the vault moves.

Five hard rules:

- **`--apply` refuses any confidence but `haute`** without a second explicit
  flag. Low confidence is opened by no flag at all: a low recommendation is an
  *I don't know*, and a batch of those is a rubber stamp.
- **A note whose hash moved since its recommendation is skipped**, by name. The
  hash is rechecked twice — at selection and immediately before the write —
  because a hundred-item batch takes seconds and ingestion runs on the same
  vault four times a day.
- **The batch stops at the first write error.** No silent half-batch: what was
  applied before the error is named on screen and in the journal, and the exit
  code is 1.
- **Each `--apply` run writes one summary line** to the journal through the
  existing mechanism, not a second journal. The per-item lines are written by
  the child writes themselves. The summary line honestly declares zero writes
  of its own: the batch writes nothing, it makes writes happen.
- **A batch touching a note with 3 or more inbound links refuses `--apply`**
  without an explicit flag. The first rule guards what the *machine* is unsure
  of; this one guards what the *corpus* holds. An item whose impact is unknown
  requires the flag on the same footing — the guard fails closed, otherwise
  *I don't know* would silently become *zero*.

The dry-run shows, per item, its inbound links — the count **and the themes
they come from**. The themes are not decoration: three inbound links on a trend
note is the mechanical pattern, it is cited by the notes it synthesises, all
from one theme. The same three from three different themes is a crossroads of
the corpus. A count alone confuses the two, which is why the rule opens a
reading instead of deciding in the human's place.

The vault lock is taken **once** for the whole series rather than once per
item, so ingestion cannot interleave in the middle of a ratification.

---

## What it has actually done

On 2026-08-10, two commands ratified **122 recommendations** — 98 kept, 24
archived, **0 skipped** — each written item by item through the journalled
path, taking the queue from 273 to 151. The 223 recommendations behind them
cost **$7.94 over four runs**: 3.6 cents a note, paid once.

Both figures are replayed from the journals by a check in the claims register;
a failing check blocks publication.

## What it does not do

- **No batch defer.** Pushing a hundred items out by a week decides nothing; it
  moves the same work at the same size. The recommendation filter accepts only
  the two terminal values.
- **No invented recommendation.** An item without a fresh recommendation never
  enters a batch, whatever the filter says.
- **No fallback on a path stem.** The queue id travels intact to the write
  path, which remains the containment guard.
