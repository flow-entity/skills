---
name: code-comment-guidelines
description: Decides whether a code comment earns its place and forces the ones that stay to explain why. Use before writing or editing any comment, when reviewing a diff that adds comments, and when refactoring code that already carries comments. Specifies the content required in module headers, public interfaces, domain rules, workarounds, performance trade-offs, thresholds, and regexes; bans step narration, syntax translation, chat-context residue, commented-out code, and bare TODOs. Not for commit messages, PR descriptions, or user-facing copy.
---

# Code comments

No comment is the default. Code with clear names and small functions carries its own meaning; a comment earns its place only by saying something the code cannot.

## The gate

Ask one question before writing a comment:

> What does this tell a future reader that the identifiers, types, and control flow do not?

If the answer is "what the code does", delete the comment. Rename the variable, extract the function, or add a type instead.

Only three kinds of content pass:

- **Why**: the decision, constraint, or domain rule that produced this code.
- **Warning**: a consequence that lives outside this file, such as "callers rely on this being sorted".
- **Pointer**: a spec, ticket, or upstream issue a reader cannot reconstruct from the repo.

## Required content, by location

| Where                       | What the comment must state                                                                                                                                                                                                    |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Module or class header      | Core responsibility, the explicit non-goals (what this module does not handle), and concurrency guarantees when the code is shared.                                                                                            |
| Public or exported function | Preconditions, boundary constraints (valid range, null behavior), side effects (mutates an argument, performs I/O, changes global state, invalidates a cache), and the conditions that raise. Never restate the function name. |
| Non-obvious branch          | The business rule or product policy behind it, especially when the branch looks wrong without that context.                                                                                                                    |
| Workaround                  | The upstream bug, dirty legacy data, or platform limit being bypassed, and the condition under which the code can be deleted.                                                                                                  |
| Performance-sensitive code  | The bottleneck avoided, the trade-off taken, and a reference for any non-trivial algorithm or formula.                                                                                                                         |
| Constant or threshold       | The unit of measurement when the name does not carry it, and the reason for this value rather than a rounder one.                                                                                                              |
| Regex or complex query      | The rule being matched, with one matching example and one non-matching example.                                                                                                                                                |

## Never write

1. **Step narration**: `// Step 1: validate input`, `// Step 2: fetch the order`.
2. **Syntax translation**: `// check if the user is active` above `if (user.isActive)`.
3. **Chat and change residue**: `// added as requested`, `// fixed the bug from earlier`, `// new logic`, dated edit banners.
4. **Placeholders**: `// ... rest of the code unchanged`, `// TODO: implement the logic here`. Write real code unless the request is explicitly for a scaffold.
5. **Commented-out code**: delete it. Version control holds the history.
6. **Boilerplate docstrings**: `@param o the order`, `@returns a boolean`. They repeat the signature without adding a constraint.
7. **File banners**: author, date, and changelog headers. Git already records them.

## When editing existing code

1. **Keep high-value comments.** Refactoring the syntax never licenses deleting a domain rule, a legacy-compatibility note, a workaround, or a performance warning.
2. **Rewrite on behavior change.** If you change a threshold, precondition, side effect, or branch, update its comment in the same edit. A comment that contradicts the code is a defect, not a leftover.
3. **Refactor instead of explaining.** If you feel the need to describe what a block does, extract it into a named function and delete the comment.
4. **Clean the scope you touch.** Remove narration and dead code from the function you are already editing. Do not sweep untouched files.
5. **Make TODOs actionable.** A surviving `TODO` names the blocker and the trigger: `// TODO(PROJ-892): switch to async once the Q4 queue cluster is live`. A bare `// TODO: optimize later` is not a TODO.

## Style

- Match the comment language and density the file already uses. Do not mix languages inside one file.
- Write in the active voice, one idea per sentence, exact terms, real units.
- State the reason instead of hinting at it. Do not clip two short clauses together with a dash.
- Let length follow content: one line when one line covers it, more when the reasoning needs it.

## Self-check before output

Scan every comment you produced:

1. Can a reader derive it from the code? Delete it.
2. Does it restate a name, a type, or a signature? Delete it.
3. Does it describe how the code got here? Delete it.
4. Does it state a unit, a boundary, a side effect, or a removal condition that the code cannot? Keep it.

When the task is refactoring an existing file's comments, read [references/worked-example.md](references/worked-example.md) for the full before/after calibration.
