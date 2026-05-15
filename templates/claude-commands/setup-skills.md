Use the GitHub MCP server to install the standard AI Skills commands into this repo.

Steps:
1. Fetch each of the following files from rawsharklives/engineering-docs (ref: main) and write them to `.claude/commands/skills/<filename>` in the current repo — create the directory if it does not exist, preserve content exactly:
   - `templates/claude-commands/skills/prd-writer.md`
   - `templates/claude-commands/skills/system-designer.md`
   - `templates/claude-commands/skills/ticket-definer.md`
   - `templates/claude-commands/skills/test-case-writer.md`
   - `templates/claude-commands/skills/qa-reviewer.md`
   - `templates/claude-commands/skills/doc-producer.md`
   - `templates/claude-commands/skills/retrospective.md`

2. Fetch each of the following files from rawsharklives/engineering-docs (ref: main) and write them to `.github/prompts/skills/<filename>` in the current repo — create the directory if it does not exist, preserve content exactly:
   - `templates/copilot-prompts/skills/prd-writer.prompt.md`
   - `templates/copilot-prompts/skills/system-designer.prompt.md`
   - `templates/copilot-prompts/skills/ticket-definer.prompt.md`
   - `templates/copilot-prompts/skills/test-case-writer.prompt.md`
   - `templates/copilot-prompts/skills/qa-reviewer.prompt.md`
   - `templates/copilot-prompts/skills/doc-producer.prompt.md`
   - `templates/copilot-prompts/skills/retrospective.prompt.md`

3. Confirm which files were written and tell the user:
   - Claude Code skills are available as `/skills/<name>` (e.g. `/skills/prd-writer`)
   - Copilot skills are available in VS Code as `#<filename>` in Copilot Chat or via `.github/prompts/skills/`
   - Commit the new files to the repo so the Copilot cloud agent can use them
   - Re-run `/setup-skills` in future to pick up updates to the Copilot prompt files (Claude Code picks up updates automatically)
