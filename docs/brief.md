← [assayer](../README.md) · [the verdict funnel](funnel.md) · [retrieval,
measured](retrieval.md)

# Brief

The reading surface. Everything the pipeline collected since you last looked,
in one panel, with its sources attached.


## What it is for

A corpus is only as good as what goes into it. The brief is where incoming
material gets read and triaged *before* it becomes a note — so the vault stays
a record of what was judged worth keeping, not a dump of what happened to
arrive.

## Five tabs, five jobs

**To read** — the feed itself. Every item carries its source, its age, and a
link to the original. Filters narrow by theme; search narrows by text. Nothing
here is generated: it is what the pollers fetched.

**Sources** — the accounts and feeds being watched, and whether each one is
still producing. A source that goes quiet is a fact about the source, not about
the day.

**Market** — the numeric series that only make sense as a series: prices,
funding, on-chain aggregates. Kept separate from prose on purpose, because a
number with no timestamp is a rumour.

**Topics** — the cross-theme bridges the graph proposed but nobody has ruled on
yet. This tab is a queue, and it is meant to look like one.

**Process** — the same health data as the dashboard, reachable without leaving
the reading context.

## Design rules

**Age is always visible.** Not in a tooltip. A count with no date reads as
fresh, and that is exactly how a stopped pipeline hides — one of ours ran for
six days writing a "fresh" state while every item inside it failed.

**Sources are attributed inline**, never aggregated into an anonymous stream.
If two outlets covered the same event, that is worth seeing.

**No model is called to render this panel.** Fetching, deduplication, filtering
and ordering are ordinary code. A model writes prose when a note is created,
and does nothing else in this path.

**Reading is local.** The feed is served from the local store, so the panel
works with no network and leaves no trace with the publishers.

## What it is not

Not a recommender. Nothing is scored for engagement, nothing is hidden for
being unpopular, and the order is chronological unless you filter it.

Not an inbox that empties itself. Items stay until they become a note or are
dismissed — the count of untriaged items is a real number, and it is allowed to
be uncomfortable.
