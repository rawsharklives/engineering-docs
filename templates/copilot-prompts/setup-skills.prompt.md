---
description: "Install the standard AI Skills prompt files into this repo. Fetches all 7 skill definitions from engineering-docs and writes them to .github/prompts/skills/."
agent: agent
tools: ['github-mcp-server/get_file_contents', 'github-mcp-server/create_or_update_file']
mode: agent
---

Use the GitHub MCP server to install the standard AI Skills prompt files into this repo.

Steps:
1. Fetch each of the following files from rawsharklives/engineering-docs (ref: main) and
   write them to `.github/prompts/skills/<filename>` in the current repo — create the
   directory if it does not exist, preserve content exactly:
   - `templates/copilot-prompts/skills/prd-writer.prompt.md`
   - `templates/copilot-prompts/skills/system-designer.prompt.md`
   - `templates/copilot-prompts/skills/ticket-definer.prompt.md`
   - `templates/copilot-prompts/skills/test-case-writer.prompt.md`
   - `templates/copilot-prompts/skills/qa-reviewer.prompt.md`
   - `templates/copilot-prompts/skills/doc-producer.prompt.md`
   - `templates/copilot-prompts/skills/retrospective.prompt.md`

2. Confirm which files were written and tell the user:
   - Skills are now available — reference any skill file in Copilot Chat to invoke it
   - To use a skill: in Copilot Chat, type `#` and select the skill file, or reference it
     by name (e.g. `.github/prompts/skills/prd-writer.prompt.md`)
   - Commit the `.github/prompts/skills/` directory so the Copilot cloud agent can use
     the skills when assigned GitHub Issues
   - Re-run this setup prompt periodically to pick up updates from engineering-docs
   - Full skill documentation: https://rawsharklives.github.io/engineering-docs/skills/
