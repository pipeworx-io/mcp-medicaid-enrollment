# @pipeworx/medicaid-enrollment

Medicaid and CHIP enrollment by state: the ACA new adult group, renewal and unwinding outcomes, separate CHIP, enrollment by eligibility group and dual status, income eligibility limits, and home and community-based waiver participants.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2015+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `medicaid_eligibility_levels(state?)` — income eligibility limits (percent of federal poverty level) for infants, children, pregnant women, parents and expansion adults. A 2018 snapshot.
- `medicaid_new_adult_group(state, from_month?, to_month?)` — monthly expansion-group enrollment, newly eligible vs not, beside total Medicaid enrollees.
- `medicaid_renewal_outcomes(state, from_month?, to_month?, vintage?)` — monthly renewals due, renewed (ex parte vs form), disenrolled (ineligible vs procedural), pending, with shares of renewals due.
- `medicaid_unwinding_metrics(state, metric?, from_month?, to_month?)` — CMS's own unwinding metrics (counts and percent shares) from the 2024 releases.
- `medicaid_chip_enrollment(state, from_month?, to_month?)` — separate CHIP enrollment by month.
- `medicaid_enrollment_by_group(state, dimension?, group?, from_month?, to_month?)` — monthly enrollment by major eligibility group, or by dual-eligible status.
- `medicaid_hcbs_waiver_participants(state?, year?, breakdown?)` — Section 1915(c) waiver participants, share of enrollees, by age, dual status, eligibility category and race/ethnicity.

For monthly total Medicaid and CHIP enrollment with application counts, and for managed-care enrollment, use `medicaid-intelligence` (`medicaid_enrollment_operations`, `medicaid_managed_care_*`); this pack does not repeat them.

## Auth

Keyless.

## Data sources

- CMS, Medicaid open data catalog: Medicaid and CHIP Eligibility Levels; Medicaid Enrollment - New Adult Group; State Medicaid and CHIP Eligibility Processing Data; Medicaid and CHIP Updated Renewal Outcomes; Separate CHIP Enrollment by Month and State; Major Eligibility Group and Dual Status Information for Medicaid and CHIP Beneficiaries by Month; Section 1915(c) waiver program participants.

Every call reads the publisher live; nothing is stored. Every response carries `data_through` (the latest period in the rows returned) and `data_as_of` (the date CMS last modified that dataset).

## Notes

- A null count means the state did not report or CMS suppressed the value. It is never converted to zero.
- Counts arrive as text with thousands separators and trailing spaces; they are parsed to numbers here.
- An unmatched or out-of-range filter returns `found: false` with the valid states, labels, metric names or month range, never a bare empty list.
- The eligibility-group series ends in 2021 and the dual-status series in 2022; the renewal series runs to 2026. Check `data_through`.
- The new adult group dataset carries several CMS update vintages per month; the latest is kept.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "medicaid-enrollment": {
      "url": "https://gateway.pipeworx.io/medicaid-enrollment/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/medicaid-enrollment/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 2015+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/medicaid_eligibility_levels \
  -H 'Content-Type: application/json' \
  -d '{"state":"Texas"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/medicaid_eligibility_levels`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-medicaid-enrollment` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/medicaid-enrollment/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "medicaid-enrollment": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-medicaid-enrollment"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Medicaid Enrollment data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
