---
description: Create or refresh the Action Items dashboard note in my Obsidian vault. Optionally scope to a folder or attendee.
argument-hint: [folder-path OR attendee-name]
allowed-tools: mcp__mcp-obsidian__obsidian_get_file_contents, mcp__mcp-obsidian__obsidian_append_content, mcp__mcp-obsidian__obsidian_delete_file, mcp__mcp-obsidian__obsidian_simple_search, mcp__mcp-obsidian__obsidian_list_files_in_dir
---

Use the **obsidian-action-items** skill.

Arguments: $ARGUMENTS
- `$1` = scope (may be absent).
  - A folder path (e.g. `WE/Stark Project`) → build a folder-scoped dashboard.
  - An attendee reference (e.g. `Pepper Potts (Stark)` or `[[Pepper Potts (Stark)]]`) → build an attendee-scoped dashboard.
  - Absent → build/refresh the main vault-wide dashboard.

Follow the skill's rules exactly: check for an existing dashboard before writing, confirm before overwriting, and always report the final path.
