---
name: gong-call-intelligence
description: >
  Use this skill whenever the user wants to search, retrieve, analyse, or extract insights from Gong sales calls via the gtmService MCP. Trigger for any of these intents:
  - Finding calls for a customer or account ("show me Tesco calls", "last calls with HSBC")
  - Getting highlights or keypoints from a specific call ("what was discussed in the Vodafone call?", "summarise the last Deutsche Telekom call")
  - Keyword or theme analysis across multiple calls ("did we mention pricing in any Unilever calls?", "which calls covered security concerns?")
  - Pre-meeting research ("I have a call with Tesco tomorrow, what's the history?")
  - Account trend analysis ("how have conversations with HSBC evolved this year?")
  - Post-call follow-up or action extraction ("what were the next steps from the last Vodafone call?")
  Always use this skill when the user references Gong, call history, call transcripts, or past conversations with a named account — even if they don't say "Gong" explicitly.
---

# Gong Call Intelligence

This skill guides Claude in using the `gtmService` MCP to search, retrieve, and analyse Gong calls. The MCP exposes four tools which are typically chained together in sequence.

---

## Tools Available

| Tool | Purpose |
|------|---------|
| `gtmService:search_gong_calls` | Find calls by customer name and optional date range |
| `gtmService:select_gong_call` | Select a specific call from search results |
| `gtmService:get_gong_call_details` | Fetch highlights and keypoints for a call |
| `gtmService:analyze_keywords_in_calls` | Search for keywords/themes across multiple calls |

---

## Workflows

### 1. Find Calls for an Account

**When to use:** User asks for a list of calls with a customer, recent call history, or pre-meeting research.

**Steps:**
1. Call `search_gong_calls` with the customer name and an appropriate date range
   - If the user says "this year", use `fromDate: <current year>-01-01`
   - If the user says "last month" or similar, use the `dateRange` param (e.g. `"last month"`)
   - If no date is specified, omit date params to get all available calls
2. Present results as a table: title, date, duration, and Gong URL
3. Offer to fetch details or run keyword analysis on any of the calls

**Tips:**
- The tool uses fuzzy matching on call titles — short account names (e.g. "BT") may return noise; prefer the full name where possible
- Always include the Gong URL for each result so the user can jump directly to the call
- Always remind the user that the call search uses the title of the call for fuzzy searches

---

### 2. Get Highlights from a Specific Call

**When to use:** User wants to know what was discussed, key decisions, or next steps from a particular call.

**Steps:**
1. If you already have a `callId` from a previous search, call `get_gong_call_details` directly
2. If not, call `search_gong_calls` first, then use `select_gong_call` to resolve the right call, then call `get_gong_call_details`
3. Present the highlights clearly — use sections for: **Key Topics**, **Action Items / Next Steps**, **Decisions Made**, **Notable Quotes** (if available)
4. Always include the Gong URL so the user can listen to the full call

**Tips:**
- If multiple calls match (e.g. two "Tesco & Miro" calls), ask the user which one they mean before fetching details — or present both and let them choose
- When summarising for pre-meeting prep, frame the output as: "Last time you spoke about X, agreed Y, and the open items were Z"

---

### 3. Keyword / Theme Analysis Across Calls

**When to use:** User wants to know if a topic came up across multiple calls, track how a theme evolved, or do account-level research.

**Steps:**
1. Search for the account to get `callId`s (use `search_gong_calls`)
2. Extract the list of `callId`s from the results
3. Call `analyze_keywords_in_calls` with the `callIds` array and relevant `keywords`
4. Present findings per keyword: which calls mentioned it, whether it was substantial or passing, with timestamps/citations where available

**Tips:**
- Good keywords to suggest if the user is vague: pricing, timeline, competition, security, integration, legal, procurement, budget, renewal
- For trend analysis, sort findings chronologically and narrate how the theme evolved across calls
- You can run keyword analysis on a single call if the user wants a focused topic drill-down
- Whenever possible include clickable links to citations.

---

### 4. Full Account Research (Pre-Meeting Brief)

**When to use:** User has an upcoming meeting and wants a comprehensive picture of the account's call history.

**Steps:**
1. Search for all calls with the account (no date filter, or YTD)
2. Fetch details for the 2–3 most recent calls using `get_gong_call_details`
3. Optionally run keyword analysis on all calls for common themes (pricing, blockers, competitors, next steps)
4. Produce a structured brief:
   - **Account snapshot** — number of calls, date range covered
   - **Recent discussions** — summary of the last 2–3 calls
   - **Recurring themes** — topics that keep coming up
   - **Open items / commitments** — anything unresolved or promised
   - **Suggested talking points** — based on what's been discussed

---

## Output Formatting

- Always present call lists as **tables** (title, date, duration, link)
- For highlights/summaries, use **named sections** with clear headers
- For keyword analysis, use a **per-keyword breakdown** showing which calls and whether mentions were substantial
- Always include **Gong URLs** so the user can verify or listen to the source
- Keep summaries concise — lead with the most actionable information
- Whenever possible include clickable links to citations.

---

## Edge Cases

| Situation | How to handle |
|-----------|--------------|
| No calls found | Tell the user clearly; suggest broadening the date range or checking the account name spelling and, if using `search_gong_calls` remind the user that it uses fuzzy matching on the Gong call title and the Customer name might not be available in the call title |
| Many calls returned (10+) | Ask the user to narrow by date range, or show the most recent 5 and offer to paginate |
| Ambiguous account name | Ask for clarification before searching (e.g. "BT" vs "BT Group") |
| User asks for "the last call" | Fetch the top result from search (most recent) and get its details |
| User wants all calls analysed | Use `analyze_keywords_in_calls` with all `callId`s — it supports up to 100 |