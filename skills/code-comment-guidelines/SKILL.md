---
name: code-comment-guidelines
description: Decides whether a code comment earns its place, what a surviving comment must contain, and where it belongs. Use when writing or editing any comment, reviewing a diff that adds comments, or refactoring code that carries existing comments. Covers the keep-or-delete gate, content required when a comment is warranted, test and machine-directive boundaries, generated and vendored code, ecosystem-mandated doc blocks, safety and lint-suppression rationale, deprecation notes, and how to report findings in review. Language-specific doc-block rules live in references/languages/ and are read only when that language is involved. Not for commit messages, PR descriptions, or user-facing copy.
---

# Code comments

No comment is the default, and the default is scoped. Library and public API code
starts the other way: an exported surface with no contract is undocumented, not
clean. Application internals start at zero comments; a module header is optional
unless the boundary or non-goals are non-obvious. Test code documents the
fixture and why the case exists, not the mechanics of the assertion.

A comment earns its place only by saying something the code cannot.

The gate governs writing. It does not license deleting an existing why-comment:
when in doubt about a comment you did not write, keep it.

## The gate

Ask two questions before writing a comment, against the surface the reader
actually sees.

1. **Does it carry non-local or non-obvious information?** Would a competent
   reader have to reconstruct context from outside this file — another module, a
   caller, history, an external system — or guess at a fact the code does not
   state? Local narration fails this question. A module responsibility passes it:
   the file does not state its own boundary.
2. **Does it prevent a wrong move?** Does it prevent a wrong use, a wrong change,
   a wrong expectation, or a wrong placement — a unit, a boundary, a side effect,
   a removal condition, or a costly investigation?

Keep the comment only when both answers are yes; otherwise delete it.

For a public or exported symbol, question 1 is asked of the declaration and its
generated docs, not the body: the caller cannot see control flow.

Content that passes falls into five kinds:

- **Why** — the decision, constraint, or domain rule that produced the code,
  including the alternative that was rejected.
- **Contract** — preconditions, invariants, postconditions, concurrency and
  ownership rules.
- **Warning** — a consequence outside this file, such as "callers rely on this
  being sorted".
- **Pointer** — a spec, ticket, runbook, or upstream issue a reader cannot
  reconstruct from the repo. The pointer must be reachable by the code's
  audience: in a public repository, do not cite an internal-only tracker; state
  the fact inline and link a public issue if one exists.
- **Safety** — why an unsafe block, raw pointer, or permission bypass is sound,
  and what must stay true for it to remain sound.

A module or file header is orientation rather than one of the five: what the unit
is for, and the non-goals that keep code out of it. It passes question 2 through
that boundary clause.

## What this skill does not govern

- **Generated and vendored code** (`*.gen.*`, `dist/`, `vendor/`, minified
  bundles, generated clients): do not add, delete, reformat, or "clean" their
  comments. Change the generator, the template, or the upstream dependency.
- **License, SPDX, and copyright headers**: policy, not banners. Never delete,
  reformat, or "clean" them.
- **Tool directives and pragmas** (build tags such as `//go:build`, linter,
  coverage, and formatter pragmas such as `// eslint-disable`,
  `// prettier-ignore`, `/* istanbul ignore next */`, `// clang-format off`, and
  folding regions such as `// region`): machine-read, not prose. Keep them and
  do not reformat them. A suppression still needs its reason (C9); a build or
  formatter directive is self-describing and needs none.
- **Documentation outside source files** (README, ADRs, guides): use the
  project's docs convention; this skill governs comments in code.
- **Commit messages, PR descriptions, and user-facing copy**: out of scope.

## Content required when a comment is warranted, by location

| Rule | Where                                  | What the comment must state                                                                                                                                                                                                                                                                        |
|------|----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| C1   | Module or class header                 | Core responsibility, explicit non-goals, concurrency guarantees when the code is shared.                                                                                                                                                                                                           |
| C2   | Public or exported function            | Preconditions, boundary constraints (valid range, null behavior), side effects, and the conditions that raise. Never only restate the name or signature; a syntax-mandated name-first sentence is not a restatement by itself, but the comment as a whole must add a constraint, including every required tag. |
| C3   | Non-obvious branch                     | The business rule or policy behind it, especially when the branch looks wrong without that context.                                                                                                                                                                                                |
| C4   | Workaround                             | The upstream bug, legacy data, or platform limit being bypassed, the symptom it causes, and the condition under which the code can be deleted.                                                                                                                                                     |
| C5   | Unsafe or privilege-sensitive block    | The invariant that makes it sound, who maintains it, and what would invalidate it.                                                                                                                                                                                                                 |
| C6   | Performance-sensitive code             | For callers: observable complexity, limits, or latency bounds in the doc comment. For maintainers: the bottleneck avoided, the trade-off taken, and a reference for any non-trivial algorithm, inline. Distinguish a contract limit or budget (a constraint) from a current-state measurement: the measurement carries its date and conditions; the limit does not need one. |
| C7   | Constant or threshold                  | The unit when the name does not carry it, and why this value rather than a rounder one. A pointer to a nearby note satisfies this only if that note exists in the same file and really gives the reason.                                                                                           |
| C8   | Regex or complex query                 | The rule being matched, one matching example, one non-matching example.                                                                                                                                                                                                                            |
| C9   | Suppression                            | The rule suppressed and why the violation is intentional, in one comment; use the tool's single-comment form (`// eslint-disable-next-line rule -- reason`) where it exists.                                                                                                                       |
| C10  | Deprecated API                         | What replaces it, and what breaks if a caller stays. Add the since-version when the project's doc convention records one; otherwise give the removal condition or timeline.                                                                                                                                 |
| C11  | Exported field, enum value, config key | Meaning, unit, sentinel value, and invariant that the type does not carry.                                                                                                                                                                                                                         |
| C12  | Exported type, interface, or alias     | What each instance represents, invariants the type does not carry, zero value, thread safety, and lifecycle.                                                                                                                                                                                       |

C1 is conditional. A library or package whose boundary is part of its contract
needs a module header; a small internal module needs one only when its boundary
or non-goals are non-obvious. Report missing C1 only for a public or library
module.

C/C++ has no doc toolchain for internal functions: when the gate calls for a
contract comment, a short what-and-why overview at the head of the function is
its stand-in, not narration. See
[references/languages/c-cpp.md](references/languages/c-cpp.md).

## Doc comments and inline comments

Two surfaces, two jobs.

- A **doc comment** (JSDoc, rustdoc, godoc, Javadoc, docstring) states the
  caller-facing contract: what a caller must know before calling.
- An **inline comment** states the maintainer-facing rationale: why this shape,
  what was rejected, what must not change.

So a contract never lives only in a comment inside the body; implementation
rationale does not enter generated docs unless it changes what callers can do
(caller-observable complexity and limits do belong in the doc comment); one fact
lives in one place; and a doc block sits directly above the declaration it
documents. A module-level block above the first declaration is not a doc comment:
in TypeScript it attaches to that declaration. Use a plain comment block for
file-level notes.

Python is the exception to "above the declaration": a docstring is the first
statement inside the module, class, or function body, indented under the
`def`/`class`. A string literal above the declaration is not attached to the
object and does not reach `help()`. Module and package docstrings sit at the top
of the module or `__init__.py`.

Place an inline comment above the statement it explains. A short end-of-line note
is acceptable for a unit or a sentinel value; a rationale gets its own block.

## Tests

A test name states the behavior; a comment carries what the name cannot.

- Document the fixture, why this case exists, and any non-obvious assertion.
- `Arrange`, `Act`, and `Assert` markers in the language's comment syntax are
  structural, not narration: keep them when the file uses them, but do not
  narrate each step.
- Do not comment assertion mechanics (`// expect true`). If the assertion needs
  a comment, improve the test name or extract a helper.
- A comment that restates the test name is noise.

## Never write

Rules are numbered N1–N12 for review reports.

1. **N1 — Step narration**: `// Step 1: validate input`, `// Step 2: fetch the
   order`. A two-to-four line overview of phases at the head of a function is not
   narration; narrating each statement is.
2. **N2 — Syntax translation**: `// check if the user is active` above
   `if (user.isActive)`.
3. **N3 — Chat, change, and AI residue**: `// added as requested`, `// per the
   user`, `// generated by`, dated edit banners.
4. **N4 — Placeholders**: `// ... rest of the code unchanged`,
   `// TODO: implement the logic here`. Write real code unless the request is
   explicitly a scaffold.
5. **N5 — Commented-out code**: delete it. Version control holds the history.
   N5 covers `//` and `/* */` comments; a project-standard `#if 0` block is
   preprocessor-disabled code, not a comment, and follows the project's
   convention only with a removal condition (c-cpp.md).
6. **N6 — Boilerplate docstrings**: `@param o the order`. Required tags are
   handled by the language files; outside them they repeat the signature.
7. **N7 — File banners** (author, date, changelog). Git records them; license
   headers are not banners.
8. **N8 — Fabricated rationale**: never invent an issue number, a benchmark
   figure, an upstream bug, or a "this is faster" claim you cannot point to. A
   wrong why is worse than no comment. When the trigger is unknown, state the
   constraint and leave the ticket out.
9. **N9 — Secrets and personal data**: no credentials, tokens, internal
   hostnames, or customer data. Comments land in git history and in generated
   docs.
10. **N10 — Comments in generated or vendored files**: see "What this skill does
    not govern".
11. **N11 — Perishable measurements**: `~20 min build`, `currently`, `today`, and
    undated current-state counts rot silently. State the durable relationship,
    date the observation with its conditions, or replace it with a contract
    limit.
12. **N12 — Threats instead of reasons**: `DO NOT TOUCH`, `DO NOT REMOVE`, and
    all-caps warnings state a feeling, not a consequence. Name what breaks and
    the condition that retires the code (C4).

Dates are allowed when they qualify a fact (a deprecation date, a measurement
date); they are banned when they record who changed what. A date on a
measurement also needs the conditions that made it true.

## When editing existing code

1. **Keep high-value comments.** Refactoring the syntax never licenses deleting a
   domain rule, a compatibility note, a workaround, or a performance warning.
2. **Rewrite on behavior change.** Change a threshold, precondition, side effect,
   or branch, and its comment changes in the same edit. A comment that contradicts
   the code is a defect, not a leftover.
3. **Refactor instead of explaining.** If you want to describe what a block does,
   extract a named function and delete the comment. When the comment states an
   invariant or constraint, encode it in a type, an assertion, or a test first;
   keep the comment only for what the code still cannot carry.
4. **Clean the scope you touch.** Remove narration and dead code from the function
   you are already editing. Do not sweep untouched files, and never edit inside
   generated or vendored files.
5. **Verify what you touch.** Check each comment in the touched function against
   current behavior. A plausible but wrong comment is a defect.
6. **Make TODOs actionable.** Name the owner and the trigger:
   `// TODO(PROJ-892): switch to async once the Q4 queue cluster is live`. An
   owner name or email is acceptable when the project has no tracker. Use the
   project's marker (`TODO`, `FIXME`, `XXX`) and its tracker convention; do not
   convert one into another. A bare `// TODO: optimize later` is not a TODO, and
   an invented ticket number is worse than none.

## Prefer executable documentation

Where the toolchain runs it, an executed example beats prose: Rust doctests, Go
`Example` functions, Python doctests, golden tests. A verified example cannot rot;
a prose example can. An example should show why a caller would reach for the item,
not only how to invoke it. When a comment states a testable invariant, add or
point to the test that enforces it; an unenforced invariant is a wish.

## Style

- Match the comment language and density the file already uses. Do not mix
  languages inside one file. Noise removal outranks density matching.
- Write complete sentences in the active voice: one idea per sentence, exact
  terms, real units, correct punctuation and capitalization.
- State the reason instead of hinting at it. Do not join two clauses with a
  dash; use a connective (`because`, `so that`, `which means`, `to avoid`).
- Let length follow content: one line when one line covers it.

## Language overrides

The rules above are language-neutral. Doc-block syntax and toolchain requirements
are not: read one language file only when that language is involved, and read it
before writing the doc block.

| Language                | Read when                                              | File                                                |
|-------------------------|--------------------------------------------------------|-----------------------------------------------------|
| Go                      | Writing a package, exported, or `Example` comment      | [go.md](references/languages/go.md)                 |
| Rust                    | Writing rustdoc, `# Safety`, or `// SAFETY:`           | [rust.md](references/languages/rust.md)             |
| Java                    | Writing Javadoc or `@deprecated`                       | [java.md](references/languages/java.md)             |
| TypeScript / JavaScript | Writing JSDoc or TSDoc                                 | [typescript.md](references/languages/typescript.md) |
| Python                  | Writing a docstring                                    | [python.md](references/languages/python.md)         |
| C / C++                 | Writing a declaration, definition, or kernel-doc block | [c-cpp.md](references/languages/c-cpp.md)           |

Toolchain, linter, license, and a style guide the project explicitly adopts
outrank every rule in this skill. A style guide can require a declaration comment
or a file header; it does not require a comment on an obvious accessor unless it
says so. When the ecosystem demands a comment, write it, but make each required
content element carry a constraint instead of echoing the type. Syntax-mandated
elements (a Go name-first line, a JSDoc block marker) are exempt from that duty.

If the language is not listed, follow the project's existing doc convention and do
not invent tags. When the project has a doc tool (KDoc, DocC, XML docs), the
tool's required structure is an override, not a style preference.

## Other ecosystems

SQL migrations, protobuf, OpenAPI, YAML/JSONC, shell, and infrastructure code
carry comments that are often API documentation or operational rationale. Read
[references/ecosystems/data-and-config.md](references/ecosystems/data-and-config.md)
when the change touches one of them; otherwise follow the project's convention
and do not invent tags.

## When reviewing a diff

Report comments that fail a named rule, one line each, with a severity:

`path:line — [blocking] C4 workaround — missing removal condition; state which gateway version retires this wait.`

`path:line — [nit] N1 narration — the code states this; delete the comment.`

Report missing required comments the same way, only for symbols the diff adds or
changes:

`path:line — [blocking] missing C2 — exported function has no contract; add preconditions and thrown conditions.`

`path:line — [blocking] missing C1 — public package has no boundary or non-goals; add a module header.`

Use `[blocking]` for a wrong or missing contract, a false rationale, a safety
gap, or a machine directive that would break; `[nit]` for style and density.
Do not report comments that pass, do not rewrite style you were not asked to
touch, do not turn a passing comment into a preference, and do not scan
untouched files.

## Self-check

Before output, scan every comment you wrote or kept:

1. Can a reader derive it from the code? Delete it.
2. Does it only restate a name, a type, or a signature? Delete it. A name-first
   sentence is fine when the comment adds a constraint or a language file
   requires the element.
3. Does it describe how the code got here? Delete it.
4. Does it state a unit, boundary, side effect, or removal condition that the code
   cannot? Keep it.
5. Is every ticket number, version, and benchmark in it real? If not, remove it.
6. Does another comment already say this? Keep one.
7. Is it inside a generated or vendored file? Leave it alone.
8. Does it contradict the code? Fix the code or the comment; never keep both.
9. Is it on the right surface: a caller contract in the doc comment, a maintainer
   rationale inline?
10. Is it machine-read (build tag, pragma, region)? If so, leave it alone.
11. Is every measurement dated and conditioned, or replaced by the durable fact?
12. Can the intended reader open the pointer? If not, state the fact inline.

For calibration — writing a module header, converting a workaround note, judging a
before/after diff — read [references/worked-example.md](references/worked-example.md).
To check the gate itself against graded keep/delete cases, read
[references/evals.md](references/evals.md).
