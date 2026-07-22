---
description: Split today's (or a given date's) Obsidian daily note into individual notes based on `---` separators and `#parent`/`#attendees` inline tags
argument-hint: [YYYY-MM-DD]
allowed-tools: mcp__mcp-obsidian__obsidian_get_periodic_note, mcp__mcp-obsidian__obsidian_get_file_contents, mcp__mcp-obsidian__obsidian_list_files_in_dir, mcp__mcp-obsidian__obsidian_list_files_in_vault, mcp__mcp-obsidian__obsidian_append_content, mcp__mcp-obsidian__obsidian_patch_content, mcp__mcp-obsidian__obsidian_batch_get_file_contents
---

Use the **obsidian-split-daily-note** skill.

Argument: $ARGUMENTS
- `$1` = date `YYYY-MM-DD` to target. Optional; if absent, use today's daily note.

Follow the skill's steps in order — including the preview + confirmation step before writing anything to the vault.
