---
name: obsidian-vault-health
description: Check my Obsidian vault for clutter and breakage — empty notes, Untitled and "(1)" copies, near-duplicate notes, broken wikilinks, misnamed or misfiled daily notes, loose attachments, stray files at the vault root, attendee notes outside Attendees/, and stale unsplit daily notes — then propose fixes and apply only the ones the user approves. Use when the user asks to "check my vault", "clean up my notes", "find broken links", "find duplicates", "vault health", or invokes /obsidian-vault-health.
---

# Vault Health

Find problems, explain them, propose a fix for each, and change nothing until the user picks which fixes to apply. The vault syncs to other devices, so every change propagates — be conservative.

## Tools

Prefer the `mcp-obsidian` MCP server:
- `obsidian_list_files_in_vault` / `obsidian_list_files_in_dir` — walk the tree.
- `obsidian_batch_get_file_contents` — read notes in batches (≈ 25 per call).
- `obsidian_complex_search` — cheap pre-filters, e.g. `{"glob": ["*.md", {"var": "path"}]}` or a regexp on content.
- `obsidian_get_frontmatter` — check `attendees:` / `date:`.
- `obsidian_put_content`, `obsidian_append_content`, `obsidian_patch_content` — fixes that edit text.
- `obsidian_delete_file` — only for approved deletions, one file at a time.

If the session also has shell access to the vault folder (it's a plain folder on disk), reading files directly is much faster for the scan. Writes should still go through Obsidian when it's running, so sync plugins pick them up.

**The MCP has no move or rename.** Recreating a file elsewhere and deleting the original breaks every link to it and loses its history. For moves and renames, give the user the exact list and have them do it in Obsidian (drag in the file explorer, or right-click → Rename), which updates links automatically. Only script moves on disk if the user asks *and* Obsidian is closed *and* you rewrite links yourself and check that none break.

## Checks

Run all of them unless the user names specific ones. For each finding, record: path, what's wrong, proposed fix.

| # | Check | How | Usual fix |
|---|---|---|---|
| 1 | **Empty notes** | `.md` with no content beyond whitespace/frontmatter | Delete, **unless** other notes link to it (then it's a placeholder: leave it, report it) |
| 2 | **Untitled / copies** | Name starts with `Untitled`, or ends in ` (1)`, ` (2)`, ` 1` | Compare with the original (below) |
| 3 | **Near-duplicates** | Same basename in different folders, or names differing only by spelling/punctuation; compare normalized lines (strip bullets `*`/`-`, whitespace, case) | If one note's lines are all contained in the other → delete the smaller. If each has unique lines → report both; never merge without approval |
| 4 | **Broken wikilinks** | `[[target]]` / `![[target]]` that resolves to no file (match on basename, or full path if it contains `/`) | Show the closest existing name; fix the link text if the user agrees. Ignore links inside code blocks and template placeholders like `[[Name (Company)]]` |
| 5 | **Daily-note hygiene** | Files under `Daily Notes/` not named `YYYY-MM-DD.md` or not in their `YYYY/MM/` folder; daily-looking files (`YYYY-MM-DD.md`) elsewhere | Rename/move (user does it in Obsidian), or merge with an existing same-date note |
| 6 | **Loose attachments** | Images/PDFs at the vault root or directly in a top-level folder instead of an `attachments/` folder | Move next to the note that embeds them; unreferenced ones → ask |
| 7 | **Root clutter** | Notes at the vault root other than dashboards and indexes the user keeps there | Suggest the best-fitting folder for each, based on content |
| 8 | **Attendee hygiene** | Notes named `{Name} ({Company}).md` outside `Attendees/`; attendee hubs missing the dataview block; `attendees:` entries that aren't wikilinks | Move into `Attendees/{Company}/`; restore the block per [[obsidian-notes]] `create-user` |
| 9 | **Stale unsplit daily notes** | Daily notes older than 7 days with a section that still has `#parent:` and a body | Offer [[obsidian-split-daily-note]] for those dates |
| 10 | **Naming consistency** | Mixed conventions for the same thing, e.g. `[R]` vs `(R)` for recurring meetings | Normalize to the majority form (user renames in Obsidian, or approve a link-safe script) |

## Report

Group findings by check, most impactful first, with counts:

```
## Vault health — {n} findings
### Empty notes (3)
1. `Work/WE/Untitled.md` — empty, no backlinks → delete
...
```

Number every finding so the user can answer "fix 1–4, 7, skip the rest".

## Applying fixes

- Apply only the numbers the user approves. Re-read each file right before changing it.
- **Deletes**: say how many files and that it can't be undone through the MCP (it may or may not land in Obsidian's trash depending on settings). One `obsidian_delete_file` per file.
- **Content edits** (link fixes, attendee blocks): `obsidian_patch_content` for scalar frontmatter and headings; for list-valued frontmatter, follow the rewrite rule in [[obsidian-notes]] (patch_content breaks YAML lists).
- **Moves/renames**: hand the user a checklist to do in Obsidian.
- Finish with what was changed, what was skipped, and what's left for the user.
