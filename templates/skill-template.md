# Skill: [Name]

## Role

[What role the AI is playing. One paragraph. Be specific: name the seniority level and
the exact responsibility. State what the AI is NOT doing (e.g. "not implementing") to
make the boundary clear.]

## Inputs expected

Before proceeding, confirm you have received:

1. **[Input name]** (mandatory / optional) — description. If mandatory and missing,
   include the exact message to say: "I need [X] before I can [do this]. Please [action]."
2. **[Input name]** (optional) — description.

[Keep the list short. If an input is truly optional, say so. If missing a mandatory input,
the skill must ask rather than guess.]

## Process

[Numbered step-by-step instructions. If the skill reads from engineering-docs, specify:
- The exact file path to read
- The MCP call: "Use the GitHub MCP server to read [path] from rawsharklives/engineering-docs (ref: main)"

Steps should be concrete and sequenced. Do not describe goals — describe actions.]

1. [First action]
2. [Second action]
3. [...]

## Outputs

[Describe exactly what artefacts to produce. Include the structure or format where
relevant. If producing a document, include the section headings. If creating GitHub
resources, describe what fields to populate.]

[Be specific enough that output from two different AI models running this skill would be
structurally identical, even if the content differs.]

## Quality bar

Before delivering, check every item:

- [ ] [Specific, verifiable check — not vague like "output is good"]
- [ ] [...]

[Every item must be checkable without running the feature. "Output is clear" is not a
valid item. "Every acceptance criterion has at least one test scenario" is.]

## Stage gate

[One paragraph. Tell the AI exactly what to do at the end: stop, deliver the output, and
say what the engineer should do next.]

Say exactly: "[The exact message the AI should deliver, including the name of the next
skill to run and what input to provide it.]"

## References

[List doc pages in engineering-docs that this skill retrieves via MCP. Use the format:]

- [Description]: `[path]` in rawsharklives/engineering-docs
