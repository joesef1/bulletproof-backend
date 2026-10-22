# Content Maintenance Checklist

Reusable recipe for editing this repo's cross-referenced curriculum
index (main.md / content.md). Run it after any topic removal or
renumbering.

## 1. Confirm the removal is complete
Grep `first-principles` across the repo (case-insensitive).
Expected: no matches outside caches and this file.

## 2. Renumber only the edited stage
Stage topic numbers are local to each stage's list, not global.
When one topic is removed:

- Renumber items in that stage's list only.
- Update only the cross-reference that names that stage (for example,
  "Needs topics 2-4 first" in the Stage-1 list becomes "Needs topics
  1-4 first").
- Leave other stages' numbers untouched.

Verify the Stage-1 list is 1-8 and has no First-Principles entry
(as of writing).

## 3. Update the Topic Index table
Confirm the removed topic is gone from the status table and that the
remaining rows still point to valid stage numbers.

## 4. Re-read before committing
After any non-zero script exit, read_file the edited files before
running `git add`. A non-zero exit can coexist with partial writes.

## 5. Tutorial sanity checks
- Every ```mermaid fence has a matching closing fence.
- Diagram count matches the walkthrough steps.
- No diagram exists without a plain-language lead-in sentence and a
  follow-up explanation.
