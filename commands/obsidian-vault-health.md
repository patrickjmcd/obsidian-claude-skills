---
description: Scan my Obsidian vault for empty notes, duplicates, broken links, misfiled notes and loose attachments, then fix only what I approve
argument-hint: [check numbers or names, optional]
allowed-tools: mcp__mcp-obsidian__obsidian_list_files_in_vault, mcp__mcp-obsidian__obsidian_list_files_in_dir, mcp__mcp-obsidian__obsidian_batch_get_file_contents, mcp__mcp-obsidian__obsidian_get_file_contents, mcp__mcp-obsidian__obsidian_complex_search, mcp__mcp-obsidian__obsidian_simple_search, mcp__mcp-obsidian__obsidian_get_frontmatter
---

Use the **obsidian-vault-health** skill.

Argument: $ARGUMENTS — optionally limit to specific checks (e.g. `broken links`, `3,4`). Default: all checks.

Report first. Change nothing until I say which numbered findings to fix.
