---
name: obsidian-daily-note
description: Create today's (or a given date's) daily note in my Obsidian vault from `Templates/Daily Note.md`, and optionally carry unfinished `- [ ]` tasks over from the previous daily note. Use when the user asks to "start my daily note", "create today's note", "new daily note", "open today's daily", "carry over my tasks", "move yesterday's todos to today", or invokes /obsidian-daily-note or /obsidian-carry-over-tasks.
---

# Daily Note

Create a daily note that matches the vault's existing ones exactly, so that [[obsidian-split-daily-note]], [[obsidian-weekly-review]] and the Dataview dashboards keep working. Optionally move unfinished tasks forward from the last daily note.

## Tools

`mcp-obsidian` MCP server (same as [[obsidian-notes]]):
- `obsidian_list_files_in_dir` on `Daily Notes/YYYY/MM/` — confirm the filename pattern and find the previous daily note (check the previous month's folder too near the start of a month).
- `obsidian_get_recent_periodic_notes` (period `daily`) — optional shortcut for the same thing. Returns 404 unless the Periodic Notes community plugin is installed — then fall back to `obsidian_list_files_in_dir` on `Daily Notes/YYYY/MM/`.
- `obsidian_get_file_contents` — read the template, check whether the target note exists (a 404 means it doesn't), read the previous note.
- `obsidian_append_content` — create the new note (creates missing `YYYY/MM/` folders).
- `obsidian_put_content` — rewrite the previous note after removing carried-over tasks (only after confirmation, and only on that one note).

Never use `obsidian_put_content` on a note that already exists for the target date — that would wipe it.

## Conventions (verify, don't assume)

- Path: `Daily Notes/YYYY/MM/YYYY-MM-DD.md` (core Daily Notes plugin, format `YYYY/MM/YYYY-MM-DD`). Before creating anything, list the target month's folder and check that existing notes follow this pattern; if they don't, follow what the vault actually uses and tell the user.
- Content: the template body verbatim. No frontmatter and no `# YYYY-MM-DD` heading — existing daily notes have neither. The template has no `{{date}}` placeholders; if a future version adds `{{date}}`, `{{date:FORMAT}}` or `{{title}}`, substitute them the way Obsidian's core Templates plugin would.
- The template is a single starter section (`## `, `#parent: `, `#attendees: `, `- `) for the user to fill in. Copy it as-is, blanks included.
- If a daily note starts with a `%% … %%` comment block (older notes and older templates did), `obsidian_patch_content` heading lookups fail on it (see the gotcha in [[obsidian-split-daily-note]] step 6): read the whole note, edit it in memory, and write it back. Notes without that block can be patched by heading as normal.

## Mode: create

Arguments (slash command): `$1` = date `YYYY-MM-DD`, optional, defaults to today in the user's local time zone.

1. Work out the path for the date.
2. `obsidian_get_file_contents` on that path.
   - **Exists** → don't touch it. Tell the user it already exists, show its first few lines, and offer carry-over (below) if it has no `## Carried over` section yet.
   - **Missing** (error / 404) → continue.
3. Read `Templates/Daily Note.md`. If it's missing, ask the user whether to create an empty note instead; don't invent a template.
4. Create the note with `obsidian_append_content` using the template content.
5. Ask whether to carry over unfinished tasks from the previous daily note (skip the question if invoked via /obsidian-carry-over-tasks, or if the user already said yes/no).
6. Report the path created.

## Mode: carry-over

Arguments (slash command): `$1` = target date, optional, defaults to today. The target note must exist; if it doesn't, run **create** first.

1. **Find the source note**: the most recent daily note *before* the target date. List `Daily Notes/YYYY/MM/` for the target month (and the previous one if needed) and take the latest date before the target. Don't go back more than 14 days without asking.
2. **Collect open tasks** from the source:
   - Lines matching `^\s*[-*] \[ \] ` (unchecked). Include each task's indented child lines (sub-bullets, sub-tasks) as part of the task.
   - Skip anything inside a `%% … %%` comment block.
   - Skip tasks that are already stubs from an earlier carry-over.
   - Remember which `## ` section each task came from.
3. **Show the list** grouped by source section, and ask which to carry (default: all). Nothing is written before the user confirms.
4. **Write to the target note**: add a section at the top of the note (after a leading `%% … %%` block if there is one), before the first `## `:

   ```
   ## Carried over
   *From [[YYYY-MM-DD]]*
   - [ ] task text
       - child line
   - [ ] another task (from: {source section heading})

   ---
   ```

   Keep task text, child lines, Tasks-plugin emoji fields (📅 due, ⏫ priority, etc.) and wikilinks exactly as written. Append `(from: {heading})` only when the heading adds context the task text lacks. If a `## Carried over` section already exists, append to it instead of adding a second one. Insert it by reading the note, adding the text, and writing it back with `obsidian_put_content` (a top-of-file insert can't be done with a heading patch) — the one case where overwriting the target note is allowed, and only after re-reading it immediately before writing so you don't clobber edits.
5. **Update the source note** so each task lives in exactly one place (otherwise the Action Items dashboard shows duplicates): replace each carried task block with a single stub line

   ```
   - Carried over to [[YYYY-MM-DD]]
   ```

   (one stub per section, not per task). Re-read the source just before writing it back with `obsidian_put_content`.
6. Report how many tasks moved and from where.

## Edge cases

- **No open tasks** in the source → say so; nothing to do.
- **Target date is in the past** → fine, but carry from the note before *that* date, not from today's.
- **Tasks in non-daily notes** (project/meeting notes) are never moved — they already live where they belong and show up on the dashboard.
- **Recurring tasks** (Tasks plugin 🔁) → carry them like any other open task, keeping the recurrence text.
