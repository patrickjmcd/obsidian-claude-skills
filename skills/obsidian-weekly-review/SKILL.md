---
name: obsidian-weekly-review
description: Run a weekly review of my Obsidian vault — summarize the week's daily notes and new or edited notes, list open, overdue and completed tasks, and flag daily notes with sections that were never split out. Use when the user asks for a "weekly review", "what did I do this week", "recap my week", "what's still open from this week", or invokes /obsidian-weekly-review.
---

# Weekly Review

A once-a-week pass that turns a week of daily notes into a short recap and a to-do list for cleanup. Read-only unless the user asks to save the review or act on a finding.

## Tools

`mcp-obsidian` MCP server:
- `obsidian_list_files_in_dir` on `Daily Notes/YYYY/MM/` — find the week's daily notes (a week can span two month folders).
- `obsidian_get_recent_periodic_notes` (period `daily`, `include_content: true`) — optional shortcut. Returns 404 unless the Periodic Notes community plugin is installed — then fall back to `obsidian_list_files_in_dir` on `Daily Notes/YYYY/MM/`.
- `obsidian_get_file_contents` / `obsidian_batch_get_file_contents` — read notes.
- `obsidian_get_recent_changes` (days ≈ 7–8) — notes created or edited this week.
- `obsidian_complex_search` — find tasks with due or done dates (regexp on content).
- `obsidian_append_content` — only to save the review if asked.

## Inputs

Arguments (slash command): `$1` = optional week, as any date inside it (`YYYY-MM-DD`) or `last`. Default: the current week (Monday–Sunday); if today is Monday, default to *last* week and say so.

## Steps

1. **Pin the date range** (Monday → Sunday) and say it.
2. **Daily notes**: read every `Daily Notes/YYYY/MM/YYYY-MM-DD.md` in range. Note which days have no daily note.
3. **Other activity**: `obsidian_get_recent_changes` for the window; drop daily notes, `Templates/`, `.trash/` and attachments. Separate *new* notes (date in filename or `date:` frontmatter inside the range) from *edited* ones where you can tell.
4. **Tasks**:
   - **Completed this week**: `- [x]` lines with a Tasks done date `✅ YYYY-MM-DD` in range, plus `- [x]` lines in the week's daily notes.
   - **Overdue**: `- [ ]` lines anywhere with `📅 YYYY-MM-DD` before today.
   - **Due next week**: `- [ ]` with `📅` in the next 7 days.
   - **Opened this week**: `- [ ]` lines in the week's daily notes and new notes.
   Use `obsidian_complex_search` with a regexp such as `- \[ \] .*📅 2026-11-` to narrow before reading files.
5. **Unsplit daily notes**: in each daily note, any `---`-bounded section that still has a `#parent:` line *and* a body (not just a `[[wikilink]]` stub) was never processed by [[obsidian-split-daily-note]]. List them by date and heading.
6. **Write the review** in chat:

   ```
   ## Week of {Mon date} – {Sun date}
   **Highlights** — 3–6 bullets: meetings held, decisions, things shipped (each linked to its note)
   **New notes** — [[…]], [[…]]
   **Done** — n tasks (list if ≤ 10)
   **Overdue** — - [ ] … 📅 … ([[source]])
   **Due next week** — …
   **Still open from this week** — …
   **Needs filing** — YYYY-MM-DD: "{section heading}" (#parent: …)
   **Gaps** — days with no daily note
   ```

   Highlights come only from what's in the notes; link sources; don't editorialize.
7. **Offer follow-ups**, each only on a yes:
   - Split the unsplit daily notes now (hand off to [[obsidian-split-daily-note]], one date at a time).
   - Carry open tasks into today's note ([[obsidian-daily-note]] carry-over).
   - Save the review as a note. Ask where the first time (suggest `Daily Notes/YYYY/Weekly/YYYY-[W]WW.md`) and follow whatever folder earlier saved reviews use after that.

## Edge cases

- **No daily notes in range** → say so; still report other activity and tasks.
- **Tasks without Tasks-plugin dates** can't be overdue — don't guess due dates from prose.
