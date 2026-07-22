---
description: Create an Obsidian "attendee" note (backlink hub for a person) under Attendees/{Company}/
argument-hint: [name] [company]
allowed-tools: mcp__mcp-obsidian__obsidian_append_content, mcp__mcp-obsidian__obsidian_list_files_in_dir, mcp__mcp-obsidian__obsidian_get_file_contents, mcp__mcp-obsidian__obsidian_batch_get_file_contents
---

Use the **obsidian-notes** skill, mode `create-user`.

Arguments: $ARGUMENTS
- `$1` = name (may be absent)
- `$2` onward = company/organization (may be absent)

If either is missing, follow the skill's interactive fallback and ask rather than guess. Otherwise proceed directly through the skill's create-user steps.
