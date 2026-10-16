# Workflow: How We Build a Tutorial

This document describes the step-by-step process for creating or updating a
tutorial in this repository. Follow these steps in order.

## The Tools

OpenCode uses these custom skills and tools during the workflow:

| Tool | Type | Used in |
|---|---|---|
| `technical-researcher` | Skill | Step 1 — Research |
| `backend-tutorial-writer` | Skill | Step 2 — Write |
| `technical-diagram-creator` | Skill | Step 3 — Diagrams |
| `tutorial-reviewer` | Skill | Step 4 — Review |
| Context7 | MCP tool | Step 1 — current library docs |
| Web search | Built-in tool | Step 1 — general concepts |

A skill is a set of written instructions, not a separate program. OpenCode reads
the skill file and follows it while working.

## Before You Start

1. Confirm the topic is not already covered. Search the repository first.
2. Read `content.md` for the topic catalog and `main.md` for the learning order (which stage the topic belongs to).
3. Pick the target location based on `main.md` stage order. Do not create a new top-level folder without agreement.
4. Decide the level: beginner, intermediate, or advanced.

## Step 1 — Research

Use the `technical-researcher` skill.

1. Write the specific question you are investigating.
2. List the technical claims that must be verified before writing.
3. Use primary sources first (official documentation, specifications).
4. Use Context7 for current library and framework documentation.
5. Use web search for general engineering concepts.
6. Cross-check important claims with a second reliable source.
7. Save the useful source links.

Do not start writing while facts are still unverified. Never invent an API, a
setting, a benchmark, or a source.

## Step 2 — Write

Use the `backend-tutorial-writer` skill.

1. Follow the topic structure:
   - What It Is
   - Why It Matters
   - How It Works
   - Failure Modes
   - Example
   - Key Points
2. Use simple English. Short sentences. Define technical terms on first use.
3. Start with a real problem when a story helps understanding.
4. Keep code examples focused and explain the important lines.
5. Mark the difference between a learning example and a production example.

Do not add a section just to make the tutorial longer. Do not add links without
explanation.

## Step 3 — Diagrams

Use the `technical-diagram-creator` skill. Add diagrams only where they explain
better than text.

1. Choose the diagram type (flowchart, sequence, entity relationship).
2. Write Mermaid code. Mermaid is the default format.
3. Verify the diagram renders. If you cannot render it, say so.
4. Place the diagram near the text it explains.
5. Explain the important parts in the surrounding text.

## Step 4 — Review

Use the `tutorial-reviewer` skill. Review your own draft or ask for a review.

Group findings by severity:

1. Critical: incorrect or misleading technical information.
2. Important: missing explanations or big learning gaps.
3. Minor: wording, formatting, optional improvements.

Fix all Critical and Important findings. Fix Minor findings when the change is
small. Do not invent problems to fill the review.

## Step 5 — Final Checks

Confirm all of the following before finishing:

- [ ] Technical claims are accurate and verified.
- [ ] English is simple and clear.
- [ ] The concept is explained, not only defined.
- [ ] Failure modes are covered for substantial topics.
- [ ] Code examples are correct. Run them when practical.
- [ ] Diagrams render (or the limitation is stated).
- [ ] Source links work and are relevant.
- [ ] Related tutorials are linked with relative links.
- [ ] `main.md` and `README.md` are updated if the topic is new.

## Updating an Existing Tutorial

1. Use the `tutorial-reviewer` skill first.
2. Change only what the review requires.
3. Keep the existing structure and tone.
4. Preserve content that is still accurate.

## Commit Rules

1. Do not commit without permission.
2. Use this message format: `type(scope): short message`.

Examples:

- `docs(stage-3): add connection pooling tutorial`
- `docs(stage-1): fix TLS handshake diagram`
- `docs(workflow): document tutorial workflow`

3. Keep one logical change per commit.
4. Keep commits small.

## When Something Is Unclear

Ask before continuing when the topic location, the technical facts, or the
expected result are unclear. A wrong assumption wastes more time than a
question.
