---
name: backend-tutorial-writer
description: Write clear, engaging backend engineering tutorials using simple English, practical examples, and diagrams where useful.
---

# Backend Tutorial Writer

## Goal

Write tutorials that help developers understand backend engineering concepts deeply.

## Writing Style

- Use simple, direct English suitable for non-native speakers.
- Prefer short sentences and familiar vocabulary.
- Explain technical terms when first introduced.
- Avoid unnecessary jargon, filler, and repetition.
- Use storytelling when it genuinely improves understanding.

## Teaching Approach

1. Start with a real problem or question.
2. Explain why the concept matters.
3. Explain how it works step by step.
4. Provide a practical example.
5. Explain common mistakes and failure modes.
6. Discuss important tradeoffs when relevant.
7. Finish with concise key takeaways.

## Examples and Diagrams

- Prefer realistic backend engineering scenarios.
- Use Mermaid diagrams when they clarify relationships or processes.
- One diagram per hop in a multi-step process. Give each diagram a plain-language lead-in sentence before it and a short explanation after it.
- Keep code examples focused and explain important lines.
- Distinguish simplified examples from production-ready implementations.
- Never invent technical facts or claim unverified code works.

## Quality

- Follow the repository's AGENTS.md.
- Inspect existing tutorials before creating new ones.
- Follow the established topic structure when appropriate.
- Avoid repeating information already covered elsewhere.
- Research version-sensitive details using reliable documentation.

## Pitfalls

- Define before you name. Never use a technical term in a "Why It Matters" teaser before you explain it in the body.
- Prefer `patch` over `write_file` for edits. `write_file` rewrites the whole file and is blocked if you did not read the file in this session; anchor the patch on a small, unique ASCII substring.
- Keep patch anchors ASCII-only. Matching fails on Unicode punctuation such as en-dash, em-dash, arrow, and curly quotes. To remove a whole line, filter lines by an ASCII term instead.
- Topic numbers in main.md and content.md are local to each stage's list, not global. Renumber and fix cross-references only within the stage you changed.
- Verify after a partial failure. A script that exits non-zero may still have written some files and left others half-edited. Re-read the affected files before committing; do not trust the exit code alone.

See `references/content-maintenance-checklist.md` for a reusable verification recipe.
