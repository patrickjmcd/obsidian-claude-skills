---
description: Move unfinished tasks from the previous Obsidian daily note into today's (or a given date's), leaving a link stub behind
argument-hint: [YYYY-MM-DD]
allowed-tools: mcp__mcp-obsidian__obsidian_get_recent_periodic_notes, mcp__mcp-obsidian__obsidian_get_periodic_note, mcp__mcp-obsidian__obsidian_get_file_contents, mcp__mcp-obsidian__obsidian_list_files_in_dir, mcp__mcp-obsidian__obsidian_append_content
---

Use the **obsidian-daily-note** skill, mode **carry-over** (create the target note first if it doesn't exist).

Argument: $ARGUMENTS
- `$1` = target date `YYYY-MM-DD`. Optional; defaults to today.

Show the tasks you'd move and wait for confirmation before editing either note.
