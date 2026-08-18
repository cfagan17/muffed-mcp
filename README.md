# Muffed MCP server

Verified NFL stats and Sleeper league context, callable from ChatGPT, Claude,
Claude Code, Cursor, or any MCP-compatible client.

```
https://muffed.ai/api/mcp
```

Free. No account, no API key, no OAuth. Read-only.

Full documentation: **<https://muffed.ai/mcp>**

---

## What it is

An AI asked a football question today either answers from whenever its training
stopped, or finds a blog post. It cannot realistically download ten seasons of
play-by-play and run a rushing-yards-over-expected model.

Muffed already did that. This server makes the result callable.

Every figure comes back with the rank its source gave it and the population it
was ranked against, so a claim arrives as *"358.16 rushing yards over
expectation, 1st of 49"* rather than as a number with nothing behind it. Each
response also carries a complete, quotable sentence, its own attribution, and an
as-of date.

**Coverage:** 520 verified metrics, 2016–2025 — EPA splits, CPOE, rushing yards
over expectation, pressure and blitz rates, coverage splits, snap and target
share, red zone and down-conversion rates, and more. Each metric reports the
seasons it actually has.

## Install

**Claude** (web, desktop, mobile) — one click:

[**Add Muffed to Claude**](https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=Muffed&connectorUrl=https%3A%2F%2Fmuffed.ai%2Fapi%2Fmcp)

That opens the Add-custom-connector dialog with the name and URL already filled;
you still review and confirm. By hand: Settings → Connectors → Add custom
connector → paste `https://muffed.ai/api/mcp`. There is no sign-in step, because
there is nothing to authorize.

**Claude Code** — install the plugin rather than the bare connector:

```bash
/plugin marketplace add cfagan17/muffed-mcp
/plugin install muffed@muffed
```

The plugin ships the endpoint *and* a skill that teaches the model how to handle
what comes back: quote the pre-formed verified sentence instead of recomposing
it, carry the attribution verbatim because the credit line is the licence,
honour the Half-PPR and charting disclosures, never derive a rank from result
ordering, and never turn verified context into a start/sit call. A bare
connector gives a model the data; the plugin also gives it the discipline the
data was published under. See
[`skills/muffed-figures/SKILL.md`](skills/muffed-figures/SKILL.md).

Without the plugin:

```bash
claude mcp add --transport http muffed https://muffed.ai/api/mcp
```

**ChatGPT** — Settings → Connectors (or Developer mode) → add an MCP server →
paste the same URL, no authentication.

**Cursor, VS Code, other MCP clients**

```json
{
  "mcpServers": {
    "muffed": {
      "type": "http",
      "url": "https://muffed.ai/api/mcp"
    }
  }
}
```

## Tools

| Tool | What it answers |
|---|---|
| `get_current_insights` | What Muffed thinks is actually real about the NFL right now — verified, dated, with the evidence |
| `query_stat_leaders` | League leaders on any verified metric, each with the rank its source gave it |
| `get_entity_metrics` | Every verified figure Muffed holds for one player or team |
| `compare_entities` | Two players or two teams side by side on the metrics both have |
| `get_metric_definition` | What a metric measures, and which seasons it actually covers |
| `list_metrics` | Which metrics can be asked about at all — keys and coverage, no figures |
| `verify_claim` | Checks a stated figure against the verified panel and returns the real one if it disagrees |
| `get_backfield` | Who backs up a starting running back — and in-season, who replaces an injured one |
| `sleeper_find_leagues` | Your Sleeper leagues, from your username |
| `sleeper_get_league` | Your league's setup — teams, scoring format, roster slots, managers |
| `sleeper_matchup_preview` | Verified context on both sides of a matchup |
| `sleeper_roster_stories` | Your roster's players and what's verified about each |
| `sleeper_standings_story` | Where your league stands, derived from Sleeper's roster records |

Every tool is read-only. Nothing writes, and no tool returns anything about a
Muffed user.

## Three rules it is built on

**1 · The panel is a reshaping, never a computation.** Figures are normalized
from a gated verified layer. A build gate re-derives every row back to its
source coordinate and fails the build on any drift, so a source refresh that
moves a number breaks CI rather than a published sentence.

**2 · Verify the query template, not the sentence.** A handful of parameterized
queries are the templates; every instantiation inherits the gate the sources
already passed. There is no arbitrary-compute endpoint and there never will be —
a tool that computes a novel number on request returns a figure no gate has
seen, to a model that will round it, recombine it, and attribute the result.

**3 · The server ranks, filters and rounds. The model never computes.** Few
rows, pre-sorted, rank and population already attached. Most "the AI got the
stat wrong" failures are a model doing arithmetic on returned data.

Where the source had no unambiguous rank, **no rank is returned at all** rather
than one derived from the result set's ordering.

## Season coverage is answered honestly

Rushing yards over expectation exists from **2018**, not 2016 — it is
tracking-dependent and there is no pre-NGS version. Ask for a decade and the
server returns the metric's real coverage instead of eight rows labelled as ten.

Every metric's registry entry carries the seasons it actually has, and
`get_metric_definition` is how a model finds that out before making a claim.

## `verify_claim`

Hand it a figure your AI has already stated:

```json
{ "claims": [{ "metric": "rb_ryoe.rush_yards_over_expected",
               "entity": "James Cook", "season": 2025, "value": 361 }] }
```

It comes back `wrong`, with the actual value (358.16), the source rank, and the
attribution. Correct claims come back `confirmed`; a metric or season Muffed
does not hold comes back `unverifiable` with the real coverage rather than a
guess.

Structured input only — never a sentence. The caller does its own claim
extraction on its own tokens; the server does a pure lookup.

## What it does not do

No projections, no start/sit or lineup advice, no draft grades, no picks — in
any tool, in any phrasing. Where a question calls for one, the tools return
verified context and let you decide. That is a permanent position, not a v1
limitation.

No draft-market (ADP) data either. The sources for it are licensed to cite, not
to redistribute; a page may quote a price with credit, but piping someone else's
data into your AI is a different thing. Everything served here is data Muffed is
licensed to build on.

No ESPN or Yahoo league support. Sleeper publishes a public read API, so loading
a league needs no login. The alternative for the others would be screen-scraping
or handling your credentials, and neither is going to happen.

## Privacy

No account, and nothing about you is stored. A league id is used to answer the
request and cached briefly to be polite to Sleeper's API — never written to a
database, never linked to you. A salted one-way hash of it is kept for 30 days
to count roughly how many distinct leagues use the server. **Sleeper usernames
are not logged in any form.**

Full policy: <https://muffed.ai/privacy>

## Attribution

Data via nflverse · NFL Next Gen Stats via nflverse · FTN Data via nflverse.
League data via Sleeper's public API.

Every response carries its own attribution string verbatim — the credit line is
the licence. Charting-derived figures cover the 2025 season only and are dated
as such.

Muffed verifies the figures as returned, not analysis derived from them.

## Support

Issues on this repo, or <hello@muffed.ai>.

Individuals, use it freely. If you are calling it at volume or building it into
a product, get in touch first so it stays up for everyone. The underlying
nflverse data is CC-BY and free — what Muffed adds is verification, structure,
freshness, and the story layer.

## About this repository

This repo holds the server's public manifests, the Claude Code plugin, and the
documentation. The server itself runs as part of muffed.ai, which is
closed-source — the plugin is a pointer to the hosted endpoint plus the skill,
not a copy of the server.

```
.claude-plugin/marketplace.json   the marketplace `/plugin marketplace add` reads
.claude-plugin/plugin.json        the plugin manifest
.mcp.json                         the endpoint the plugin installs
skills/muffed-figures/SKILL.md    how to quote the figures correctly
server.json                       MCP Registry manifest (`ai.muffed/muffed`)
glama.json                        maintainer declaration for the Glama listing
```

Two separate listings, often confused: `server.json` is the
[MCP Registry](https://registry.modelcontextprotocol.io) entry, which is what
generic MCP clients and directory crawlers read. The `.claude-plugin/` files are
the Claude Code plugin, which is what `/plugin install` reads. Bumping one does
not bump the other.


## Listed on

[![muffed-mcp MCP server](https://glama.ai/mcp/servers/cfagan17/muffed-mcp/badges/score.svg)](https://glama.ai/mcp/servers/cfagan17/muffed-mcp)

- [MCP Registry](https://registry.modelcontextprotocol.io) — `ai.muffed/muffed`
[Glama](https://glama.ai/mcp/servers/cfagan17/muffed-mcp)
