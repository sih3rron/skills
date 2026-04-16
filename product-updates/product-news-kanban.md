---
name: product-news-kanban
description: Extract and format product updates from the Miro #product_news Slack channel into structured Kanban cards for a Miro board. Use this skill whenever the user asks to pull product updates, news, or releases from Slack and format them for a Kanban board, Miro board, or similar visual tracker. Trigger whenever the user mentions "#product_news", "product updates", "release notes for Miro board", or "Kanban from Slack". Always use this skill even if the user only vaguely references "getting updates from Slack" in a Miro context.
---

# Product News → Miro Kanban Skill

Extracts all product updates from the `#product_news` Slack channel over the last 14 days and formats them as structured Kanban cards for a Miro board.

---

## Step 1: Fetch Messages from Slack

Use the Slack MCP tool to read `#product_news`. Target: **last 14 days**, retrieve all messages (paginate if needed).

- Channel: `#product_news`
- If Slack is unreachable or the tool errors, **do not fabricate any cards**. Instead, output a single Unclassified card:
  - Title: Slack Unavailable
  - Status: Unclassified
  - Description: Could not reach Slack. Please check your connection or permissions.
  - Date: [today's date]
  - Slack Link: N/A

---

## Step 2: Fetch Thread Replies

For every top-level message that has replies, fetch the thread using the Slack thread tool. Synthesize technical details from replies into the **Good to Know** field.

---

## Step 3: Classify Each Update

Apply classification logic based on exact keyword matching in the message text:

| Status | Keywords to look for |
|---|---|
| **Internal** | "Mironeer release", "live for Mironeers", "internal rollout", "employee-only" |
| **Beta** | "Early Access", "Beta", "public beta", "private beta", "skateboard", "limited pilot" |
| **GA** | "General Availability", "100% rollout", "public release" |
| **Mainstage** | Leave **empty** — for manual entry only |
| **Unclassified** | Anything that doesn't clearly match the above |

- If a message is ambiguous, default to **Unclassified**.
- Keyword matching is case-insensitive.

---

## Step 4: Split into Separate Cards

If a **single Slack post** mentions **multiple distinct features**, create a **separate card for each feature**. Do not bundle unrelated features into one card.

---

## Step 5: Output as a Table

The Miro MCP does not have a dedicated Kanban tool. Instead, output all cards as a **Miro-compatible table**. Once pasted into a Miro board, the table can be converted into a Kanban board natively using Miro's table-to-Kanban feature.

### Table Format

Output a single markdown table with the following columns:

| Status | Title | Date | The Problem | The Solution | Good to Know | Timeline | Slack Link |
|---|---|---|---|---|---|---|---|
| Internal / Beta / GA / Mainstage / Unclassified | Feature name | e.g. Jan 14, 2025 | Pain point addressed | What was shipped | Constraints, thread details, feedback requirements | Next milestones | Raw Slack permalink |

### Formatting Rules
- One row per feature (multi-feature posts = multiple rows).
- Keep cell content **concise** — brief phrases or short sentences only. These become Kanban card fields.
- **Slack Link**: Copy the raw permalink exactly from Slack message metadata. Do **not** reconstruct or guess URLs.
- If a field has no information, write `N/A`.
- The **Mainstage** status is reserved — do not pre-populate any rows with it. Leave that column value blank or note `[Manual]` so the team can assign it on the board.
- Sort rows **chronologically** (oldest first).

---

## Step 6: Add a Miro Usage Note

After the table, include this note for the user:

> 💡 **To convert to a Kanban board in Miro:** Paste this table onto your Miro board → select the table → use the "Convert to Kanban" option from the table toolbar. The **Status** column will become your swim lanes automatically.

---

## Execution Checklist

- [ ] Fetched all messages from `#product_news` for the last 14 days
- [ ] Fetched thread replies for any message with replies
- [ ] Every message has been classified (none skipped)
- [ ] Multi-feature posts have been split into individual cards
- [ ] All Slack permalinks are copied verbatim from metadata (not reconstructed)
- [ ] Mainstage column is empty (not pre-filled)
- [ ] Output is a single markdown table (not grouped cards)
- [ ] Rows are sorted chronologically within the table
- [ ] Miro usage note included after the table