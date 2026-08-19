# Contributing

This repository publishes a page and its evidence: the prose, the screenshots,
and [`claims.json`](claims.json) — the register each published figure is
replayed against. The engines themselves are not here yet, so there is no build
to run and very little a patch can compile.

**Issues on the measurements are the ones worth opening.** That is what the
register is for. The shapes that help:

- a figure that does not survive its own stated bound, or a bound so loose it
  could never fail;
- a statistical reading done wrong — [`docs/retrieval.md`](docs/retrieval.md)
  argues *against* the retrieval work here, and the `grep` column is a standing
  invitation to argue it harder;
- an arm the benchmark should have run before it drew its conclusion.

Name the figure, say how it should have been measured, and say what result
would settle it. One limit you will hit immediately: **the corpus is one
person's private notes and cannot be published**, so nothing here is re-runnable
on our data. A report naming a public corpus that would settle the question is
worth more than one asking for ours.

Patches are welcome on the pages themselves — a dead link, a wrong number, a
proof that reads backwards.

**One thing to know before you send one.** This page is published as a single
commit, force-pushed: the history here is the page, not the workshop. A merged
change is rewritten under the repository's own author rather than replayed, so
your name survives in the issue and its thread, and not in the git history. It
is a deliberate choice about what this repository is, and a bad deal for anyone
who wants the commit. Say so if it does not suit you — an issue costs you
nothing and is worth more here anyway.
