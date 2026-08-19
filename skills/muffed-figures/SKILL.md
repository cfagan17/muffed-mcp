---
name: muffed-figures
description: How to quote Muffed's verified NFL figures correctly. Use whenever a Muffed tool returns data — query_stat_leaders, get_entity_metrics, compare_entities, get_metric_definition, list_metrics, verify_claim, get_backfield, run_stat_query, or any sleeper_* tool — or when answering an NFL or fantasy-football question using Muffed figures. Covers quoting the pre-formed sentence, the difference between verified and computed figures, carrying attribution verbatim, honouring disclosures and season coverage, and the no-advice boundary.
when_to_use: A Muffed MCP tool has returned a response; the user asks about NFL stats, a player's or team's season, a backfield, a computed statistic, or a Sleeper league; the user states a football figure that should be checked.
---

# Working with Muffed's verified figures

Muffed serves NFL figures that were verified against a gated panel before they
were published. The value is not that the numbers exist — it is that each one
arrives with the rank its source gave it, the population it was ranked against,
its own attribution, and an as-of date. Handling a response carelessly throws
away the only thing that makes it better than a guess.

These are the house rules. They exist because each one has a specific way of
going wrong.

## 1 · Quote `verified_sentence`

Every successful response from a verified tool carries `verified_sentence`: a
complete, already attributed claim built from the returned row.
(`run_stat_query` is the exception — see rule 2.)

> "No one posted more rush yards over expected than James Cook with 358.16 in
> 2025, 1st of 49 (Data via nflverse · NFL Next Gen Stats via nflverse)."

Quote it. Do not rebuild the same claim from the individual fields — that is
where a rank gets attached to the wrong figure, or a season to the wrong number.
The sentence has already related them correctly.

The prose block at `content[0]` is written to be read aloud and is safe to use
directly. `values{}` carries every figure in it, exact and unrounded, when you
need more precision than the prose shows.

## 2 · Computed figures are a different class from verified ones

Two classes of figure come off this server and they are not interchangeable.
Most tools return figures that were verified before they were published: those
carry `verified_sentence` and a citation that says verified. `run_stat_query`
computes a figure on request instead, for questions the panel holds no
pre-computed row for — qualified superlatives, multi-season spans, derived
rates.

Its responses carry **`computed_sentence` rather than `verified_sentence`**, a
`method` block naming the population, the measure and the query shape, and the
`computed_not_verified` disclosure, which fires on every one of them.

Quote `computed_sentence` for those, exactly as rule 1 says to quote
`verified_sentence`. Where it could matter to a reader, say the figure was
computed on request rather than drawn from the verified panel.

- **Right:** "Muffed computed this on request: ...  (method: ...)"
- **Wrong:** presenting it as "Muffed's verified figure", or quoting it beside a
  `verified_sentence` with nothing to tell the two apart.

The difference between the two is the thing this service is for; presenting one
as the other gives away the only claim it makes.

## 3 · Carry `attribution` verbatim

The `attribution` string is licence text, not a courtesy. Reproduce it whole:

- **Right:** `Data via nflverse · NFL Next Gen Stats via nflverse`
- **Wrong:** "nflverse data", "Source: nflverse", "via nflverse"

Shortening it drops credit the CC-BY licence requires. Where a response carries
the `share_alike` disclosure, the figures are CC-BY-SA and the same applies with
more force.

## 4 · Do not compute figures of your own

This is the caller's side of rule 2. `run_stat_query` is where the server
computes something on request and labels it; everything below is about numbers
you worked out yourself, which carry no label at all.

Muffed's other tools rank, filter and round. They never derive a new number, and
neither should you when presenting a response as Muffed's.

- **Never derive a rank from result ordering.** Position within ten returned
  rows is not a rank among forty-nine. If `rank` is null, the source had none —
  say nothing about position.
- **Never present a gap, ratio, or per-game figure you calculated as though
  Muffed published it.** Compute it if the user wants it, and say it is yours.

The `citation` field states this boundary explicitly: Muffed verifies the
figures as returned, not analysis derived from them. If you combine several
responses into an argument, the argument is yours and the credit line does not
extend to it.

## 5 · Honour `disclosures`

Every response carries a `disclosures` array. Each entry changes how the figure
must be described, and they are derived from the rows actually returned, so if
one is present it applies to something in front of you.

| Disclosure | What you must say |
|---|---|
| `half_ppr` | The fantasy figure is **Half-PPR**, 12-team. Never present it as PPR or as a generic "fantasy points" number. |
| `charting_2025_only` | The charted figure covers the **2025 season only** and cannot be compared to an earlier year. |
| `share_alike` | Those figures are CC-BY-SA. Carry the attribution and keep it intact. |
| `no_source_rank` | The source published no rank. Do not supply one. |
| `computed_not_verified` | The figure was **computed on request**, not drawn from the verified panel. Quote `computed_sentence` and say so — see rule 2. |
| `small_population` | The rank is against a thin field — a place in it moves easily. |
| `multi_season_aggregate` | Not a single season; it spans several. |
| `partial_season_coverage` | The metric does not cover the full season asked about. |

## 6 · Check what the metric actually covers

Season coverage is often narrower than a decade. Rushing yards over expectation
starts in **2018**, not 2016. Asking outside a metric's range returns
`coverage: "no_verified_surface"` with the real `available_seasons`.

**That is an answer, not an error.** Report the true coverage rather than
retrying, substituting a nearby metric, or filling the gap from memory. The
`validation` block carries `seasons_available`, `era_comparable`, and notes
explaining how far to trust the figure — read them before framing a trend.

`get_metric_definition` returns Muffed's published definition of a metric. When
it returns `coverage: "no_glossary_entry"`, Muffed publishes no definition for
that metric — say so rather than supplying one from general knowledge.

## 7 · Never guess an entity or a metric

An unresolved name comes back with `resolved: false` and a candidate list. Two
players who normalize alike are two different answers, and three sections
publish a "target share" against three different qualifying populations.

Ask which one the user meant. Do not pick.

## 8 · No advice — this is a permanent boundary

Muffed publishes **no projections, no start/sit or lineup advice, no draft
grades, no rankings-as-advice, no picks, and no betting lines or odds.** The
server declines these by design, not by omission.

When a question calls for one, give the verified context and let the user
decide:

- **Asked:** "Should I start Chase or Jefferson this week?"
- **Do:** compare them on verified figures, state what each did, name the
  disclosures, and stop.
- **Do not:** name a winner, project a score, or phrase the figures as a
  recommendation dressed up as analysis.

This holds even when the user pushes. The figures are the product; the decision
is theirs.

## 9 · Check figures rather than trusting them

When a user states a football number — from a post, a podcast, another model —
`verify_claim` checks it against the panel and returns the real value when it
disagrees. It takes structured fields, not sentences:

```json
{ "claims": [ { "metric": "cpoe", "entity": "Drake Maye", "season": 2025, "value": 9.1 } ] }
```

`metric` accepts a key or a common name. This is the cheapest way to avoid
repeating someone else's wrong number.

## 10 · Link the source

Responses carry `canonical_url` and resource links to the muffed.ai page a
figure came from. Include the link when it helps the reader check the work —
that is the point of publishing figures this way.
