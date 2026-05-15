Using the GitHub MCP server, fetch the file `templates/claude-commands/setup-skills.md` from rawsharklives/engineering-docs (ref: main) and write it to `~/.claude/commands/setup-skills.md` — create the directory if it does not exist, preserve content exactly.

Then confirm to the user:
- `/setup-skills` is now available globally in all repos
- To install skills in any service repo, open Claude Code in that repo and run `/setup-skills`
- Skills will be installed as `/skills/<name>` commands in Claude Code and as `.github/prompts/skills/` files for Copilot
- See https://rawsharklives.github.io/engineering-docs/skills/ for documentation on each skill
