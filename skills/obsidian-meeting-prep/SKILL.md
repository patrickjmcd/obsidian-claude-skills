---
name: obsidian-meeting-prep
description: Build a short pre-meeting brief from my Obsidian vault — past notes with the same attendees or from the same recurring series, open action items involving those people, and what was decided last time. Use when the user asks to "prep for my meeting with X", "what did we talk about last time with X", "brief me for the weekly sync", "what do I owe X", or invokes /obsidian-meeting-prep.
---

# Meeting Prep

Pull together what the vault already knows before a meeting, so the user walks in knowing what was discussed, decided and promised. Read-only by default.

## Tools

`mcp-obsidian` MCP server:
- `obsidian_list_files_in_dir` — find attendee hub notes under `Attendees/` and recurring-meeting folders named `(R) {Meeting Name}`.
- `obsidian_simple_search` — find notes that mention an attendee wikilink or a topic.
- `obsidian_complex_search` — find notes whose `attendees:` frontmatter contains a given person (regexp on `content`).
- `obsidian_batch_get_file_contents` — read the matching notes in one call.
- `obsidian_get_frontmatter` — read `date:` / `attendees:` cheaply when ranking notes.
- `obsidian_append_content` — only if the user asks to save the brief.

## Inputs

Arguments (slash command): free text — attendee names, a meeting name, or both (e.g. `Tony Stark`, `Weekly Sync`, `Lucius Fox budget review`). If nothing is given, ask who or what the meeting is.

## Steps

1. **Resolve attendees.** For each name, find the hub note `Attendees/{Company}/{Name} ({Company}).md` by listing `Attendees/` subfolders. Accept partial matches (first name only) but confirm if more than one person matches. The wikilink form `[[Name (Company)]]` is what other notes use, so search on that exact string.
2. **Resolve a recurring series** if the input names a meeting: look for a folder `(R) {Meeting Name}` anywhere in the tree (project folders are nested, e.g. `Work/WE/Stark Project/(R) Weekly Sync/`). Its notes are named `YYYY-MM-DD.md`; take the latest 3.
3. **Find related notes.**
   - Notes that link each attendee: `obsidian_simple_search` for `[[Name (Company)]]`, plus `obsidian_complex_search` with a regexp on content for the name inside `attendees:` frontmatter.
   - Daily-note sections that mention them but were never split (search results under `Daily Notes/`).
   - Rank by `date:` frontmatter (fall back to the date in the filename), newest first. Keep the latest ~5 overall.
4. **Read** those notes with `obsidian_batch_get_file_contents`.
5. **Extract**:
   - **Last time**: date, note link, and 2–4 bullets of what was discussed or decided in the most recent meeting.
   - **Open action items**: unchecked `- [ ]` lines in those notes, plus any open task anywhere that mentions an attendee's wikilink or first name (search `- [ ]` with the name). Split into *I owe them* vs *they owe me* when the wording makes it clear; otherwise list as *open*.
   - **Recurring threads**: topics that show up in two or more of the recent notes.
   - **Questions left open**: lines ending in `?` or marked TBD / follow up / unclear.
6. **Present the brief** in chat:

   ```
   ## Prep: {meeting} — {attendees}
   **Last time** ([[note]], YYYY-MM-DD)
   - …
   **Open action items**
   - [ ] … ([[source note]])
   **Threads to pick up**
   - …
   **Open questions**
   - …
   ```

   Link every claim to its source note. Don't pad: if a heading has nothing, drop it. Never invent context that isn't in the notes.
7. **Offer to save** the brief. If yes, add it as a new section to today's daily note (create it via [[obsidian-daily-note]] if missing), shaped for [[obsidian-split-daily-note]]:

   ```
   ---
   ## {Meeting name}
   #parent: {project folder or (R) series path}
   #attendees: [[Name (Company)]]
   - Prep: …
   ```

   so the user can take notes under it during the meeting and split it afterward.

## Edge cases

- **No history found** → say so plainly and list where you looked; suggest creating attendee notes via [[obsidian-notes]] `create-user` if the person has none.
- **Very common name** (many notes) → limit to the last 90 days unless the user asks for more.
