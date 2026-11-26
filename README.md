# obsidian-claude-skills

A small, personal collection of [Claude Code](https://docs.claude.com/en/docs/claude-code) skills and slash commands I use to run my [Obsidian](https://obsidian.md) vault from the terminal — creating daily and meeting notes, splitting my daily note into per-topic notes, building attendee backlink hubs, prepping for meetings, running a weekly review, keeping the vault tidy, and generating an action-items dashboard.

This repo is set up as a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces): the plugin content (`skills/`, `commands/`) lives at the repo root alongside `.claude-plugin/marketplace.json` and `.claude-plugin/plugin.json`, so it can be installed globally with `/plugin` — no cloning required.

## What's in here

Skills (in `skills/`) — the documented behavior Claude follows:

| Skill | Purpose |
|---|---|
| `obsidian-notes` | Create meeting/project notes (`{folder}/{title}.md` with attendees + date frontmatter) or attendee "hub" notes (`Attendees/{Company}/{Name} ({Company}).md` with a dataview backlink block). |
| `obsidian-split-daily-note` | Split a daily note on `---` separators, promote sections tagged with `#parent:` to standalone notes in the right folder, and replace the section body with a `[[wikilink]]` stub. Substantive sections without `#parent:` get surfaced as suggestions rather than moved silently. |
| `obsidian-action-items` | Build a dashboard note listing every open `- [ ]` checkbox task across the vault (or scoped to a folder / attendee). Uses Obsidian's native `TASK` dataview query so checkboxes stay interactive and write back to the source note. |
| `obsidian-daily-note` | Create a daily note from `Templates/Daily Note.md` at the vault's daily-note path (never overwriting one that exists), and carry unfinished `- [ ]` tasks over from the previous daily note, leaving a link stub so each task lives in one place. |
| `obsidian-meeting-prep` | Pre-meeting brief: last discussion, open action items and recurring threads with the given attendees or recurring series, each linked to its source note. Optionally saved into today's daily note as a split-ready section. |
| `obsidian-weekly-review` | Recap a Monday–Sunday week: highlights, new notes, completed / overdue / upcoming tasks (Tasks-plugin dates), and daily-note sections that were never split. |
| `obsidian-vault-health` | Scan for empty notes, Untitled and `(1)` copies, near-duplicates, broken wikilinks, misfiled daily notes, loose attachments, root clutter and naming drift; report numbered findings and apply only the approved fixes. |

Slash commands (in `commands/`) — thin wrappers that dispatch to the skills:

| Command | Delegates to | Notes |
|---|---|---|
| `/obsidian-create-note [folder] [title]` | `obsidian-notes` (mode: `create-note`) | Falls back to asking if either arg is missing. |
| `/obsidian-create-user [name] [company]` | `obsidian-notes` (mode: `create-user`) | Same. |
| `/obsidian-split-daily-note [YYYY-MM-DD]` | `obsidian-split-daily-note` | Defaults to today. |
| `/obsidian-action-items [folder-or-attendee]` | `obsidian-action-items` | No arg = vault-wide dashboard. |
| `/obsidian-daily-note [YYYY-MM-DD]` | `obsidian-daily-note` (mode: `create`) | Defaults to today; offers carry-over afterward. |
| `/obsidian-carry-over-tasks [YYYY-MM-DD]` | `obsidian-daily-note` (mode: `carry-over`) | Moves open tasks from the previous daily note. |
| `/obsidian-meeting-prep [attendees / meeting]` | `obsidian-meeting-prep` | Read-only unless you ask to save the brief. |
| `/obsidian-weekly-review [YYYY-MM-DD \| last]` | `obsidian-weekly-review` | Defaults to this week (last week on Mondays). |
| `/obsidian-vault-health [checks]` | `obsidian-vault-health` | Report first; fixes only on approval. |

If you clone the repo locally, a `.claude/settings.local.json` you create yourself can allow-list the read-only Obsidian MCP calls so they don't prompt every run. Write-side calls (`append_content`, `delete_file`) still prompt. This isn't installed by the plugin itself since permissions are user/project-scoped, not plugin-scoped.

## Requirements

- [Claude Code](https://docs.claude.com/en/docs/claude-code) CLI installed and logged in.
- Obsidian, with the [Dataview](https://github.com/blacksmithgu/obsidian-dataview) and [Tasks](https://github.com/obsidian-tasks-group/obsidian-tasks) community plugins (the dashboards and attendee hubs depend on them) and the [Local REST API](https://github.com/coddingtonbear/obsidian-local-rest-api) community plugin enabled — this is what exposes the vault to MCP.
- An MCP server pointed at that API. I use [MarkusPfundstein/mcp-obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) (that's why the tool names in these skills look like `mcp__mcp-obsidian__obsidian_*`). Any equivalent Obsidian MCP would work if the tool names match, otherwise you'd need to search-and-replace.

## Using it

### Install as a plugin (recommended)

No cloning needed. From within Claude Code:

```
/plugin marketplace add patrickjmcd/obsidian-claude-skills
/plugin install obsidian-tools@obsidian-claude-skills
```

This installs the skills and slash commands at the user level, available from any directory. To pick up updates later:

```
/plugin marketplace update obsidian-claude-skills
```

To uninstall:

```
/plugin uninstall obsidian-tools@obsidian-claude-skills
```

### Developing locally

If you've cloned the repo and want to test changes before they're pushed, add it as a local marketplace instead of the GitHub one:

```
/plugin marketplace add /path/to/obsidian-claude-skills
/plugin install obsidian-tools@obsidian-claude-skills
```

### Manual install, no plugin system

If you'd rather not use `/plugin` at all, copy the `skills/` and `commands/` folders straight into your user-level Claude Code config:

```bash
mkdir -p ~/.claude/skills ~/.claude/commands
curl -sL https://github.com/patrickjmcd/obsidian-claude-skills/archive/refs/heads/main.tar.gz -o /tmp/ocs.tar.gz
tar -xzf /tmp/ocs.tar.gz -C ~/.claude/skills --strip-components=2 obsidian-claude-skills-main/skills
tar -xzf /tmp/ocs.tar.gz -C ~/.claude/commands --strip-components=2 obsidian-claude-skills-main/commands
rm /tmp/ocs.tar.gz
```

Re-run the same commands to update. To uninstall, remove the specific files this added:

```bash
rm -rf ~/.claude/skills/obsidian-notes ~/.claude/skills/obsidian-split-daily-note ~/.claude/skills/obsidian-action-items \
  ~/.claude/skills/obsidian-daily-note ~/.claude/skills/obsidian-meeting-prep ~/.claude/skills/obsidian-weekly-review ~/.claude/skills/obsidian-vault-health
rm -f ~/.claude/commands/obsidian-create-note.md ~/.claude/commands/obsidian-create-user.md ~/.claude/commands/obsidian-split-daily-note.md ~/.claude/commands/obsidian-action-items.md \
  ~/.claude/commands/obsidian-daily-note.md ~/.claude/commands/obsidian-carry-over-tasks.md ~/.claude/commands/obsidian-meeting-prep.md ~/.claude/commands/obsidian-weekly-review.md ~/.claude/commands/obsidian-vault-health.md
```

Once loaded (by any of the methods above), either invoke by natural language ("split my daily note", "add Jane from Acme as an attendee") or by slash command.

## Heads-up: this is shaped to *my* vault

The skills encode conventions I actually use:

- **Folder taxonomy**: notes live under project folders such as `WE/{Project}/` (possibly nested, e.g. `Work/WE/{Project}/`), daily notes at `Daily Notes/YYYY/MM/YYYY-MM-DD.md`, attendee hubs under `Attendees/{Company}/`, recurring meetings under `(R) {Meeting Name}/YYYY-MM-DD.md`.
- **Frontmatter**: `attendees:` (a YAML list of wikilinks) then `date:`, no H1 title (the filename is the title).
- **Attendee filename convention**: `{Name} ({Company}).md` — the parenthesized company suffix is load-bearing for disambiguation and backlinking.
- **Daily note template** (`Templates/Daily Note.md`) is one blank starter section — `## `, `#parent: `, `#attendees: `, `- ` — rather than an instruction comment block. Older daily notes may still start with a `%% ... %%` block; the split skill strips it and avoids heading patches on those notes.

None of that is portable to other vaults without editing. The skill files (`skills/*/SKILL.md`) are the source of truth for every convention — if you're adapting, start there.

## License

[MIT](./LICENSE) — do whatever you want, no warranty.
