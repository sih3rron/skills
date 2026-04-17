---
name: deal-brief
description: >
  Produces a structured deal review brief for a named customer account by pulling
  the latest information from Gong calls, Salesforce, and Slack. Use when the user
  asks for a deal brief, account overview, pre-demo research, or pre-call prep for
  a named customer — even if they don't use those exact words. Trigger phrases:
  "brief on [customer]", "what do we know about [customer]", "prep me for [customer]",
  "deal review for [customer]", "account summary for [customer]".
---

# Deal Brief Skill

Produces a dated, cited deal review brief for a named account. All data comes from
Glean (`user-Glean` MCP server) — Gong, Salesforce, and Slack. Always include source
links and dates so the user can judge recency.

## Audience & tone

**Readers:** SEs, AEs, and Sales Leadership doing pre-call prep or deal reviews.
They want signal, not prose. Write accordingly:

- **Terse over verbose** — one clause per bullet, no scene-setting sentences
- **No filler** — cut "it's worth noting that", "importantly", "as mentioned above"
- **Shorthand is fine** where unambiguous — "SF" for Salesforce, "opp" for opportunity,
  "EB" for economic buyer, "DRI" for directly responsible individual
- **Lead with the fact** — "No confirmed champion" not "It appears that a champion has
  not yet been formally identified"
- **Quotes over paraphrase** — if the customer said it clearly in a Gong call, quote it
  directly rather than summarising; it's faster to read and harder to misinterpret
- **Numbers over adjectives** — "$47K ARR, 175 seats, 114% activation" not
  "a healthy enterprise account with good adoption"

---

## Step 0 — Confirm the account name

Before searching, ask the user to confirm the exact account name and any known aliases:

> "To make sure I pull from the right Salesforce record, can you confirm the exact
> account name — and note any aliases it might appear under (e.g. subsidiary names,
> domain variants)?"

Use their answer as the primary search term. If they provide aliases, run searches
for each and merge the results, deduplicating by opportunity ID and call URL.

---

## Step 1 — Parallel search (run all three at once)

Use `user-Glean` → `search` tool. Run all three calls in a single message batch:

```
search(query="<AccountName>", app="gong",       sort_by_recency=true)
search(query="<AccountName>", app="salescloud", sort_by_recency=true)
search(query="<AccountName>", app="slack",      sort_by_recency=true)
```

Vet results before using them:
- **Relevance**: does the result actually relate to this account, or just contain the name as a keyword?
- **Freshness**: prefer results < 3 months; flag anything 3–6 months; treat 6–12 months with caution; exclude 12+ months unless nothing more recent exists.

---

## Step 1b — Surface opportunities and confirm focus

After Step 1, extract all open opportunities from the Salesforce results and present
them to the user as a simple list:

> "I found [N] open opportunities for [AccountName]:
> 1. [Name] — Stage: X, Value: $Y, Close: Z
> 2. ...
>
> Which should I focus the brief on? (Or say 'all' to cover everything.)"

Wait for their answer before proceeding. This avoids building a deep brief on the
wrong opportunity when an account has multiple active deals.

---

## Step 2 — Deepen the most useful sources

Run these in parallel once Step 1 results are back:

1. **Gong full transcripts** — use `read_document` with `urls: [<gong_url_1>, <gong_url_2>]` for the 2 most recent calls. Extract: confirmed requirements, pain points, objections, agreed next steps, named attendees.

2. **Slack dealroom threads** — channel naming is inconsistent across the org. Run three searches in parallel covering the known variants (all lowercase, spaces replaced with hyphens):

   ```
   search(query="<AccountName>", app="slack", channel="dealroom-<accountname>")
   search(query="<AccountName>", app="slack", channel="<accountname>-dealroom")
   search(query="<AccountName>", app="slack", channel="deal-room-<accountname>")
   ```

   If none of those return results, fall back to a general Slack search without a channel filter and look for any channel whose name contains the account name and a variant of "dealroom" or "deal-room". Note in the brief which channel name was found.

   Once the channel is identified, use `read_document` on any threads that look substantive (multi-reply threads, manager feedback, post-meeting debriefs).

3. **Salesforce opportunities** — the search result snippets usually contain JSON. Parse out: Stage, Amount, Close Date, Description, Manager's Notes, Last Activity, created contacts.

---

## Step 3 — Write the brief

Use the template below. Include every section even if some data is missing — note "not found" rather than omitting. Date each source inline.

```markdown
## [AccountName] — Deal Brief
*Compiled: [today's date]*

---

### Account Snapshot
| Field | Source | Detail |
|---|---|---|
| Account | Salesforce account record — `Account Name` | |
| Industry | Salesforce account record — `Industry` | |
| Employees | Salesforce account record — `Employees` | |
| Region / GEO | Salesforce account record — `GEO` / `Region` | |
| AE | Salesforce account record — `AE` | |
| Account Manager | Salesforce account record — `Account Owner` | |
| SE | Salesforce account record — `EISR`, or SE participant named in Gong calls | |
| Current licence | Salesforce account record — `Total Enterprise Licenses Contracted` and `Enterprise ARR` | |
| ICP Bucket | Salesforce account record — `ICP Bucket` | |

---

### Open Opportunities
| Opportunity | Stage | Value | Close Date | Updated |
|---|---|---|---|---|
| | | | | |

[Link to each opportunity in Salesforce]

---

### Key Contacts
| Name | Title | Email |
|---|---|---|
| | | |

---

### Confirmed Demo / Meeting Requirements
*Source + date for each item*

Numbered list. Direct quotes from Gong or Salesforce description where available.
Note which items are confirmed vs inferred. **Cap at 6 items** — if more exist,
prioritise: (1) explicitly stated by the customer, (2) repeated across multiple calls,
(3) most recently mentioned.

---

### Supporting Context & Pain Points
*Source + date for each item*

Bullet list. **Cap at 5–7 items.** If there are many Gong calls, do not summarise
every call — instead extract the pain points that are:
- Named explicitly by the customer (not inferred by Miro team)
- Repeated across more than one call or source
- Specific and actionable (e.g. "trainees find swim lane creation slower than Visio"
  not "they want better diagramming")

Deprioritise: vague statements, Miro-team observations without customer confirmation,
points only mentioned in passing.

---

### Strategic Context
*Source: #dealroom-[account] Slack*

Bullet list, newest first. **Cap at 6 items.** Focus on: deal stage transitions,
manager coaching notes, positioning flags, open questions about C-level ownership
or business objective. Include Slack links. Skip Rattle bot messages unless they
signal a meaningful stage change.

---

### Key Watch-outs
Numbered list. **Cap at 5 items — ruthlessly prioritise.** These are the flags an SE
must know before walking into a call. Derive them by cross-referencing sources:

- **Stage vs. activity mismatch** — compare Salesforce stage against the most recent
  Gong next steps. If the stage implies progression but Gong shows unresolved discovery
  questions or no follow-up, flag it.
- **Champion / economic buyer gaps** — check Salesforce `Have Champion` and
  `Have Economic Buyer` fields. If either is false, flag it.
- **Scope creep signals** — if Gong calls reference technical asks (custom apps, APIs,
  integrations) that go beyond the stated opportunity, flag the risk.
- **Stale next steps** — if the most recent Gong call agreed a next step that has no
  evidence of follow-through in Slack or subsequent calls, flag it.
- **AI / compliance gates** — if the customer mentioned approval boards, security reviews,
  or procurement processes in any call, flag these as potential blockers.

Only include a watch-out if there is a specific piece of evidence for it. Do not
include generic risks that apply to any deal.
```

---

## Output rules

- **Always cite**: every substantive claim gets a source (Gong URL, Salesforce URL, or Slack URL) and a date.
- **Freshness indicators** — apply to every cited source:
  - `🟢 < 3 months` — current, use freely
  - `🟡 3–6 months` — recent, note the date
  - `🟠 6–12 months` — aging, flag explicitly and treat with caution
  - `🔴 12+ months` — stale, include only if nothing more recent exists and mark clearly as stale
- **Do not fabricate**: if a field is not found in any source, write "not found" — do not infer or guess.
- **Vetting**: exclude Gong calls where the account name appears only in passing (e.g. mentioned as a reference in a call about a different account).
- Write the brief directly in chat first, then proceed to Step 4.

---

## Step 4 — Export hook

After writing the brief in chat, ask:

> "Want me to push this to a Miro board? (Say 'yes', 'markdown' for a local file, or 'no' to keep it here.)"

Treat a bare "yes" or no response as Miro. Only prompt for clarification if the answer is ambiguous. Then:

### Option 1 — Miro board
Read the `miro-geometry` skill (`~/.claude/skills/miro-geometry/SKILL.md`) before placing
anything on the board. Follow its layout guidance to avoid overlap and containment errors.

Use the `user-miro-mcp` MCP server. Read tool schemas from
`~/.cursor/projects/empty-window/mcps/user-miro-mcp/tools/` before calling any tool.

#### Layout

Create the Doc first (placed at `x=0, y=0`), then place tables to the right with
sufficient spacing to avoid overlap. Doc width is typically 600–800px; use a minimum
gap of 300px before the first table. Stagger tables vertically if there are multiple.

```
Doc (x=0, y=0)  |  300px gap  |  Table 1 (x=1100, y=0)
                               |  Table 2 (x=1100, y=600)
```

#### Doc (`doc_create`)

- The Doc is the primary narrative — all sections, citations, and context go here
- `doc_create` does NOT support tables or code blocks — convert all tables to bullet lists
- Where a section has a corresponding table, leave a placeholder comment in the Doc
  content (e.g. `[table link — see link-back pass below]`) so you know where to insert it

#### Tables (`table_create`)

Create one table per structured data section. Recommended tables:

1. **Account Snapshot** — columns: Field, Detail
2. **Open Opportunities** — columns: Opportunity, Stage, Value, Close Date, Updated
3. **Key Contacts** — columns: Name, Title, Email

Place each table to the right of the Doc.

#### Link-back pass (`doc_update`) — do this last

Once all tables are created and their `miro_url` values are known, make a single
`doc_update` call to replace each placeholder with the real deep-link:

`[View as table](<miro_url returned by table_create>)`

Do this as one final step after all tables exist — not inline during Doc creation —
to avoid editing the Doc multiple times and risking missed or broken links.

Use the account name as the board name.

### Option 2 — Markdown file
Always ask the user where to save the file before writing:

> "Where would you like me to save the Markdown file? (e.g. Desktop, a specific project folder)"

Write the brief verbatim from chat to the path they specify using the Write file tool.
Do not assume a default path.
