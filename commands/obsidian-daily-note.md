---
description: Create today's (or a given date's) Obsidian daily note from the Daily Note template, then offer to carry over unfinished tasks
argument-hint: [YYYY-MM-DD]
allowed-tools: mcp__mcp-obsidian__obsidian_list_files_in_dir, mcp__mcp-obsidian__obsidian_get_recent_periodic_notes, mcp__mcp-obsidian__obsidian_get_file_contents, mcp__mcp-obsidian__obsidian_list_files_in_dir, mcp__mcp-obsidian__obsidian_append_content
---

Use the **obsidian-daily-note** skill, mode **create**.

Argument: $ARGUMENTS
- `$1` = date `YYYY-MM-DD`. Optional; defaults to today.

Never overwrite an existing daily note. Ask before carrying tasks over.
