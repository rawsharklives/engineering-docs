Use the GitHub MCP server to install the standard /docs and /work Claude Code commands into this repo.

Steps:
1. Fetch `templates/claude-commands/docs.md` from the rawsharklives/engineering-docs repository (ref: main)
2. Fetch `templates/claude-commands/work.md` from the same repository (ref: main)
3. Write the fetched content to `.claude/commands/docs.md` and `.claude/commands/work.md` in the current repo — create the `.claude/commands/` directory if it does not exist, and preserve the file content exactly
4. Confirm which files were written and tell the user that /docs and /work are now available
