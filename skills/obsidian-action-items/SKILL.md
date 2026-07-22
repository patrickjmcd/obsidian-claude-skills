---
name: obsidian-action-items
description: Create or refresh an action-items dashboard note in my Obsidian vault. Surfaces all open `- [ ]` checkbox tasks across the vault (or a scoped folder / attendee) as INTERACTIVE checkboxes — clicking them in the dashboard writes back to the source note. Also flags legacy `#task:` inline markers for optional conversion. Use whenever the user asks about action items, tasks, TODOs, "what do I owe", "what's on my plate", "who owes what", or invokes the /obsidian-action-items command.
---

# Action Items Dashboard

## Observed conventions

- **Primary format**: `- [ ]` (open) and `- [x]` (done) markdown checkboxes throughout the vault (~91k lines across the vault contain this pattern). This is Obsidian's native task syntax — dataview's `TASK` query renders them as interactive checkboxes that write back to the source file when clicked.
- **Structural**: many meeting notes have a `## Action Items` H2 heading with checkboxes underneath.
- **Assignment**: an inline wikilink to an attendee note (e.g. `- [ ] Investigate X [[Bruce Wayne (WE)]]`) is the pattern for assigning a task to a person. Not always used, but where present the dashboard can group by it.
- **Legacy**: at least one older note (`WE/Stark Project/Bat&Robin Design Alignment.md`) uses inline `#task: {description}` markers. These are NOT native Obsidian tasks and will NOT appear in `TASK` queries. Surface them separately as a nudge to convert.

## Tools

All via the `mcp-obsidian` MCP server:
- `obsidian_get_file_contents` — check whether the target dashboard note already exists
- `obsidian_append_content` — create the dashboard note (creates parent folders too)
- `obsidian_delete_file` + `obsidian_append_content` — refresh/overwrite an existing dashboard (only with explicit user confirmation, since delete is destructive)
- `obsidian_simple_search` — quick check for legacy `#task:` markers in a scoped folder

## Modes

### Default: create-or-refresh the main dashboard

Target path: `Action Items Dashboard.md` at vault root.

Behavior:
1. Check whether the file already exists via `obsidian_get_file_contents`.
   - If it does, ask the user whether to **skip**, **refresh (delete + recreate)**, or **place at a different path**. Never silently overwrite.
2. Build the dashboard content (template below).
3. Create via `obsidian_append_content`.
4. Report the exact path created.

### Scoped dashboard (folder or attendee)

Arguments (if invoked via slash command): `$1` = a folder path (e.g. `WE/Stark Project`) OR an attendee reference (e.g. `Pepper Potts (Stark)` or `[[Pepper Potts (Stark)]]`).

Disambiguation:
- If `$1` contains `/` or matches an existing directory → treat as a **folder scope**.
- If `$1` matches `Name (Company)` → treat as an **attendee scope**.
- Otherwise ask.

Default target paths (always confirm before creating):
- Folder scope: `{folder}/_Action Items.md`
- Attendee scope: `Attendees/{Company}/_Action Items — {Name}.md`

The leading `_` sorts the file to the top of Obsidian's file browser; if a sibling in the target folder already establishes a different convention, follow that instead.

## Dashboard content template

Below is the exact content to write for the **default (vault-wide) dashboard**. For scoped modes, swap `FROM ""` → `FROM "{folder}"`, and for attendee scope add `AND contains(text, "[[Name (Company)]]")` to the `WHERE` clause.

````markdown
# Action Items Dashboard

_Live view — check items off here and it updates the source note._

## Open — grouped by source

```dataview
TASK
FROM ""
WHERE !completed
GROUP BY file.link
SORT file.mtime DESC
```

## Recently completed (last 30 days)

```dataview
TASK
FROM ""
WHERE completed AND completion >= date(today) - dur(30 days)
GROUP BY file.link
SORT completion DESC
```

## Assigned to a specific person

_Edit the wikilink inside `contains(...)` to filter by any attendee._

```dataview
TASK
FROM ""
WHERE !completed AND contains(text, "[[Bruce Wayne (WE)]]")
GROUP BY file.link
```

## Legacy `#task:` markers (not native tasks — consider converting)

```dataviewjs
const rows = [];
for (const page of dv.pages()) {
    try {
        const file = app.vault.getAbstractFileByPath(page.file.path);
        if (!file || file.extension !== "md") continue;
        const content = await app.vault.read(file);
        for (const line of content.split("\n")) {
            if (/#task:?\s/.test(line) && !/^\s*-\s\[.\]/.test(line)) {
                rows.push([dv.fileLink(page.file.path), line.trim()]);
            }
        }
    } catch (e) { /* skip unreadable */ }
}
if (rows.length) {
    dv.table(["Source", "Item"], rows);
} else {
    dv.paragraph("_No legacy `#task:` markers found — nice._");
}
```
````

## Notes on convention going forward

When creating new action items in meeting/project notes, prefer:
- `- [ ] Do the thing` — native Obsidian task, appears in the dashboard as an interactive checkbox.
- `- [ ] Do the thing [[Bruce Wayne (WE)]]` — assign a person via attendee wikilink; the dashboard's "Assigned to a specific person" section can filter on this.
- `- [ ] Do the thing #followup #batcave` — add tags to group tasks across projects.

Avoid new `#task:` markers — they don't show up in dataview `TASK` queries and become invisible over time.
