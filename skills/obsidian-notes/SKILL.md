---
name: obsidian-notes
description: Create or extend notes in my Obsidian vault via the mcp-obsidian MCP server. Covers two note types — general/meeting notes (date + attendees frontmatter, filed in a project folder) and "attendee" hub notes (a person's dataview backlink page, filed under Attendees/{Company}/). Use this skill whenever the user asks to create an Obsidian note, log a meeting, capture notes from a call, add a new contact/attendee/person note for backlinking, or references the /obsidian-create-note or /obsidian-create-user commands — even if they don't use those exact words, e.g. "make a note for my sync with X" or "add Jane from Acme as a contact."
---

# Obsidian Note Creation

This skill encodes the conventions already used across my vault, discovered from existing notes. Follow them exactly rather than inventing new formatting — consistency across the vault is the whole point (backlinks and dataview queries depend on exact filename/wikilink matches).

## Tools

All operations go through the `mcp-obsidian` MCP server:
- `obsidian_list_files_in_vault` — list vault root
- `obsidian_list_files_in_dir` — list a folder (use to check for existing folders/notes before creating)
- `obsidian_get_file_contents` / `obsidian_batch_get_file_contents` — read existing notes (use to check for duplicates, or to mimic a sibling note's tag conventions)
- `obsidian_append_content` — creates a new file (and any missing parent folders) if it doesn't exist, or appends if it does. This is the create mechanism for both note types below.
- `obsidian_patch_content` — insert content relative to a heading/block/frontmatter field in an *existing* note (useful for later edits, not typically needed for initial creation)

**Gotcha: `patch_content` cannot edit list-valued frontmatter.** Calling it with `target_type=frontmatter, target=attendees` (or any other list field) and `operation=replace` coerces whatever you pass (YAML block, JSON array, comma list) into a single quoted **string**, breaking the list format that sibling notes and dataview queries depend on. To correct an attendees list on an existing note, use `obsidian_delete_file` + `obsidian_append_content` to rewrite the file with the correct YAML list. `patch_content` is still fine for scalar frontmatter fields (`date`, `title`) and for heading/block edits.

Never use `obsidian_delete_file` as part of note creation.

## Two note types

Decide which mode applies (or ask the user if unclear):

| Mode | Trigger | Filed at | Content |
|---|---|---|---|
| `create-note` | meeting notes, project notes, general captures | `{folder}/{title}.md` | `date:`/`attendees:` frontmatter + H1 title |
| `create-user` | new person/contact/attendee for backlinking | `Attendees/{Company}/{Name} ({Company}).md` | dataview backlink block only |

---

## Mode: create-note

Arguments, if invoked via slash command: `$1` = folder, `$2`+ = title. Either or both may be absent — ask rather than guess.

### 1. Gather what's missing
- **Title** — required.
- **Folder** — required (relative to vault root, e.g. `WE/Cyberdyne Systems`). List the vault root or `Attendees`-adjacent folders with `obsidian_list_files_in_dir` if the user wants a reminder of what exists, so they reuse folders instead of fragmenting similar ones.
- **Date** — default to today; only ask if backdating.
- **Attendees** — ask if this note has any. For each name given:
  - Check whether a matching note exists under `Attendees/` (list the relevant company subfolder, or the whole `Attendees/` tree if company is unknown).
  - If it doesn't exist, offer to create it now via **Mode: create-user** before finishing this note — attendees should be real wikilinks, not plain text, or backlinking breaks.
- **Tags** — only ask if the destination folder already uses tags on sibling notes (spot-check one with `obsidian_get_file_contents`). Don't impose tags where the vault doesn't already use them.

Don't force this template onto notes that clearly aren't meeting/project notes (packing lists, recipes, reference notes). If the title/context suggests otherwise, ask whether the user wants frontmatter at all, or just a bare note.

### 2. Build content

```
---
attendees:
  - "[[Name (Company)]]"
date: {date}
---
# {title}
```

- Frontmatter order: `attendees:` first, then `date:` — matches the vault's existing convention.
- Omit the `attendees:` key entirely if there are none.
- Wikilinks must exactly match the target attendee note's filename (minus `.md`).
- **Omit the `# {title}` H1 by default; only include it if a sibling note in the destination folder already uses one.** my vault convention (verified across `WE/Stark Project/` and its `(R) *` subfolders) is: frontmatter → straight into H2 sections, no redundant H1. Spot-check one sibling with `obsidian_get_file_contents` before deciding. Never assume the H1 belongs.
- **Match the folder's existing filename conventions.** If siblings use `Bat&Robin` (no spaces around `&`), don't create `Bat & Robin ...` even if the user typed spaces — flag it and offer to match instead.
- Leave the body empty below the frontmatter/H1 — never invent note content.
- Add `tags:` only per the rule above, lowercase, matching sibling convention.

### 3. Create
Path: `{folder}/{title}.md`. If unsure whether it already exists, check with `obsidian_get_file_contents` first and confirm with the user before overwriting anything. Otherwise call `obsidian_append_content` with the full content above.

### 4. Confirm
State the exact path created.

---

## Mode: create-user

Arguments, if invoked via slash command: `$1` = name, `$2`+ = company/organization.

These notes are backlink hubs only — never add biographical content, roles, or context here. That content belongs in the meeting/project notes that link to the person.

### 1. Gather what's missing
- **Name** — required.
- **Company/organization** — required. List `Attendees/` first so the user reuses an existing folder rather than creating a near-duplicate (`WE` vs `we` vs `Wayne Enterprises`).

### 2. Check for duplicates
List `Attendees/{Company}/` and check for an existing note matching the name (allow for minor formatting variance). If found, tell the user and ask whether to reuse it instead of creating a new one.

### 3. Build content

Filename: `Attendees/{Company}/{Name} ({Company}).md`

Content (verbatim — this is the standardized attendee-note template. Older notes may still use a `table` or `LIST file.path` form; leave them alone unless the user asks to normalize):

````
```dataview
TABLE date AS "Date", file.folder AS "Project"
FROM [[]]
SORT date DESC
```
````

- Shows the meeting date (from the note's `date:` frontmatter) and its containing folder, most-recent-first.
- A backlinking note with no `date:` renders a blank Date cell — that's fine; it degrades gracefully.

### 4. Create
Call `obsidian_append_content` with that filepath and content. This creates the company subfolder too if it's new.

### 5. Confirm
Report the path created and give the user the exact wikilink to paste elsewhere:

[[{Name} ({Company})]]
