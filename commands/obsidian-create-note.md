---
description: Create a new Obsidian note from the Meeting template (date + attendees frontmatter)
argument-hint: [folder] [title]
allowed-tools: mcp__mcp-obsidian__obsidian_append_content, mcp__mcp-obsidian__obsidian_list_files_in_dir, mcp__mcp-obsidian__obsidian_get_file_contents, mcp__mcp-obsidian__obsidian_batch_get_file_contents
---

Use the **obsidian-notes** skill, mode `create-note`.

Arguments: $ARGUMENTS
- `$1` = folder (may be absent)
- `$2` onward = title (may be absent)

If either is missing, follow the skill's interactive fallback and ask rather than guess. Otherwise proceed directly through the skill's create-note steps.
