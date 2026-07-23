---
name: obsidian-split-daily-note
description: Split a daily note in my Obsidian vault into standalone notes based on `---` separators and `#parent`/`#attendees` inline tags. Sections with `#parent` become new notes filed under the resolved parent folder; the daily note keeps a wikilink stub in place of the moved content. Sections without `#parent` stay put unless they look important enough to suggest a move. Use when the user asks to "split my daily note", "process today's note", "extract notes from the daily", "organize my daily note", or invokes the /obsidian-split-daily-note command.
---

# Split Daily Note

Take a daily note under `Daily Notes/YYYY-MM-DD.md`, break it into individual sections by `---`, and promote the sections that carry a `#parent` inline tag into standalone notes. Leave everything else in the daily note. The mechanics rely on conventions from [[obsidian-notes]] — read that skill's frontmatter/filename rules before creating any notes here.

## Tools

Same `mcp-obsidian` MCP server as [[obsidian-notes]]:
- `obsidian_get_periodic_note` — fetch today's daily note directly.
- `obsidian_get_file_contents` — for a specific date, or to inspect target folders / existing files.
- `obsidian_list_files_in_dir` / `obsidian_list_files_in_vault` — resolve the folder that a `#parent:` value points to.
- `obsidian_append_content` — create each new standalone note (also autocreates missing folders).
- `obsidian_patch_content` — surgically rewrite the daily note (replace body under each moved heading with a wikilink stub). Never `obsidian_delete_file` the daily note.

## Section anatomy

A "section" in a daily note is bounded by lines containing only `---` (or the file start/end). Inside a section, expect:

```
## {Heading — often "Attendee Name - Topic" or just "Topic"}
#parent: {Folder or project name}         # optional
#attendees: [[Name (Company)]]            # optional, may repeat or be a list
- body content...
```

- Tags (`#parent:`, `#attendees:`) can appear in any order after the heading, on their own lines.
- Attendees may appear as one wikilink per line, comma-separated, or as a YAML list. Preserve every one found.
- Body = everything below the heading that isn't a `#parent:`/`#attendees:` line.

## Rules

1. **`---` demarcates a section.** Split on lines matching exactly `---`.
2. **No `#parent` tag → leave in place** by default. If the section looks *substantive* (multiple bullets, decisions, action items, named stakeholders, or a topic the user is clearly tracking over time), surface it to the user with a proposed folder + title and ask before moving. Never move it silently.
3. **If the heading starts with an attendee's display name, drop that name from the title.** Match the display name portion of an `#attendees:` wikilink (`Tony Stark` from `[[Tony Stark (Stark)]]`) against the leading portion of the heading, up to the first ` - ` / ` — ` / `:` separator. If it matches, strip the prefix and separator.
   - `Tony Stark - Long-Term Wayne Enterprises + Stark` + attendee `[[Tony Stark (Stark)]]` → title `Long-Term Wayne Enterprises + Stark`.
   - `Lucius Fox - IS Support` + attendee `[[Lucius Fox (WE)]]` → title `IS Support`.
   - If the heading is *just* the attendee's name (a 1-1 or check-in), keep the attendee's name — otherwise the note has no title. Ask the user for a more descriptive title in that case.
4. **Replace the moved section in the daily note with a stub link.** Keep the `## Heading` intact so the daily note is still scannable; replace the body (tags + content) under it with the wikilink to the new note:
   ```
   ## Tony Stark - Long-Term Wayne Enterprises + Stark
   [[Long-Term Wayne Enterprises + Stark]]
   ```
   The `---` separators between sections stay put.

## Steps

### 1. Load the daily note
- Default target: today's daily note via `obsidian_get_periodic_note(period="daily")`.
- If the user names a date, load `Daily Notes/{YYYY-MM-DD}.md` directly with `obsidian_get_file_contents`.
- Capture the date (`YYYY-MM-DD`) for the frontmatter of every note we create.

### 2. Split and classify
- **First, strip Obsidian comment blocks (`%% ... %%`) from the file.** These are template-instruction blocks that render invisibly in preview mode and must never be treated as content. Remove every `%% ... %%` (including multi-line ones) before splitting into sections. The Daily Note template puts an instruction block at the very top (directly followed by the first `## ` heading — no leading `---`); without stripping it, the parser treats those instruction bullets as section content and can even trip the `patch_content` bare-`---` gotcha below.
- Split on `\n---\n` (and treat file start/end as boundaries).
- For each section:
  - Parse the H2 heading (first `## ` line).
  - Collect `#parent:` value and all `#attendees:` wikilinks.
  - Everything else is the body.
- Bucket sections into:
  - **Move** — has `#parent`.
  - **Suggest** — no `#parent` but the section is substantive (see rule 2).
  - **Skip** — no `#parent`, trivial/personal, OR the section is empty after stripping comment blocks (this is what happens to the preamble section above the template's first `---`).

### 3. Confirm the plan with the user
Before touching anything, print a preview:
```
Move:
  [Stark Project] Long-Term Wayne Enterprises + Stark  ← from "Tony Stark - …"
  [WE]           IS Support                          ← from "Lucius Fox - …"

Suggest (has no #parent — want to move?):
  Pepper Potts - Batmobile MK II   → propose: WE/… ?

Skip:
  British Open
```
Wait for user confirmation (or per-section adjustments) before writing anything.

### 4. Resolve each `#parent:` to a folder
- Start by listing vault root + `WE/` for candidate folders.
- Exact match by folder name (case-insensitive tolerated but flag mismatch) — e.g. `Stark Project` → `WE/Stark Project/`.
- If multiple candidates or none: ask the user which folder to use, offering the closest matches.
- Cache the resolution per `#parent` value for the run.

### 5. Create each moved note
Follow [[obsidian-notes]] `create-note` conventions:
- Filename: `{resolved-folder}/{title}.md`, where `title` has any attendee prefix stripped.
- Check for a name collision with `obsidian_get_file_contents` first. If it exists, ask before appending/overwriting — do not silently overwrite meeting notes.
- Content:
  ```
  ---
  attendees:
    - "[[Name (Company)]]"
  date: {daily-note-date}
  ---
  {body from the section, unchanged}
  ```
  - Omit `attendees:` if the section had none.
  - No H1/H2 title inside — the filename is the title, matching sibling notes in `WE/Stark Project/` etc.
  - Preserve internal `---` inside the body (uncommon, but leave alone if present).
- Create via `obsidian_append_content` (also creates the folder if new).

### 6. Rewrite the daily note
For each moved section, replace the body under its H2 with the wikilink stub. **Important:** `obsidian_patch_content` with `target_type=heading, operation=replace` replaces everything under that heading up to the next same-or-higher-level heading — including the `---` separator that visually followed the section. So the stub must include that trailing separator, or the daily note collapses into one long section.

Unless the section being replaced was the very last one in the file, use:
```
obsidian_patch_content(
  filepath = "Daily Notes/{date}.md",
  operation = "replace",
  target_type = "heading",
  target = "{original H2 text, exactly as it appeared}",
  content = "[[{new-title}]]\n\n---\n"
)
```

For the last section (no trailing `---` originally), omit it:
```
  content = "[[{new-title}]]\n"
```

- `target` must be the *original* heading text (before any attendee-prefix stripping), so it matches the daily note as-is.
- Verify with `obsidian_get_file_contents` afterward that separators are intact between remaining sections; repair by re-patching with the corrected content if the tool ate one.
- Sections in the **Skip** bucket are not touched.
- **Suggest** sections: only patch if the user accepted the suggestion in step 3.

**Gotcha: `patch_content` cannot edit list-valued frontmatter.** If you need to fix the `attendees:` list on a just-created note (e.g. wrong wikilink), `obsidian_patch_content` with `target_type=frontmatter, target=attendees, operation=replace` will coerce whatever you pass (YAML block, JSON array, comma list) into a single quoted **string** — breaking the list format sibling notes and dataview queries depend on. The reliable fix is `obsidian_delete_file` + `obsidian_append_content` to rewrite the file with the correct YAML list.

**Gotcha: `patch_content` returns `invalid-target` when the file has a bare `---` line before the first real content.** The Local REST API parser treats such a `---` as YAML frontmatter start and then can't find any headings at all (every `target_type=heading` call fails, not just one). This shows up when a `%% ... %%` comment block is followed by a blank line and then `---` before the first `## ` heading — a shape older versions of the Daily Note template used to ship. Two ways to handle it:
- **Preferred:** ensure the file has no bare `---` between the top and the first H2. The current template puts the `%% %%` block *directly* above the first `## ` (no leading `---`), so this case shouldn't reappear.
- **Fallback if it does:** delete the daily note and re-append the whole file with all H2 stubs baked in. Preserve the leading `%% %%` block verbatim. Slower and heavier than patching, but reliable.

### 7. Confirm what changed
Report back:
- Each new note path created (with its wikilink for pasting elsewhere).
- Which sections in the daily note were reduced to stubs vs. left alone.
- Anything the user was asked about but declined.

## Edge cases

- **No `---` separators at all.** Treat the whole file as a single section; the rules above still apply (usually results in Skip).
- **Template preamble at the top of the file.** The Daily Note template puts a `%% ... %%` instruction block directly above the first `## ` heading (no leading `---`). Strip comment blocks first (step 2). What's left in the first section is just that H2 and its body — process it like any other section. The `%%` block itself is discarded from parsing but stays in the daily note on disk untouched.
- **Section has `#parent` but no heading.** Ask for a title — a moved note without a filename can't be created.
- **Multiple `#parent:` values in one section.** Ask which one wins; don't guess.
- **Attendee wikilink refers to a note that doesn't exist yet.** Offer to create it via [[obsidian-notes]] `create-user` mode before finishing the split, so backlinks resolve.
- **Daily note has H1 or no H2 (only bullets under the separators).** Ask the user how to title the promoted note — never fabricate one.
- **Title collides with an existing note in the target folder.** Show the user the existing note's first few lines and ask whether to append to it, rename the new one, or skip.
