# Javadoc

Read when writing Javadoc, a Markdown `///` doc comment, `@deprecated`, or
overriding docs. The gate and the never-write list stay in `SKILL.md`.

## Two forms

Java has two documentation-comment forms. Pick the one the file or project
already uses; do not mix them in one doc comment.

- **Traditional `/** ... */`**: HTML plus Javadoc tags. Use it when the build's
  javadoc runs on JDK 21 or earlier, or when the file already uses it.
- **Markdown `///`** (JEP 467, JDK 23+): each line of the doc comment starts with
  `///`; the content is CommonMark 0.30, with GFM pipe tables. Inline and block
  Javadoc tags still work; links use `[Element]` or `[text][Element]`; fenced
  code blocks are allowed and may contain `*/`.

`///` is documentation only. `//` and `/* */` remain implementation comments;
do not turn a `///` doc comment into a plain `//` note when editing.

Markdown whitespace: leading whitespace and the three slashes are removed from
each line, then the block is shifted left by the least-indented non-blank line.
Extra indentation and trailing whitespace are preserved because they are
significant (indented code blocks, list continuation, hard line breaks).

Toolchain: `javadoc` must run on JDK 23 or later to render `///`. On an older
javadoc the lines are ordinary comments, so the generated API docs silently lose
them. doclint applies to both forms.

## Shape

- Javadoc on public and protected types and members. Package-private and private
  members only when the gate passes.
- Class comment: purpose, usage, and thread-safety assumptions. Method comment:
  the contract, not the implementation.
- `@param`, `@return`, `@throws` when the build enforces them (doclint, a strict
  `maven-javadoc-plugin`). Each is a complete sentence that adds a range, unit,
  null rule, or thrown condition, never just the parameter name.
- Javadoc lines follow the project's line length (often 80 columns); let the
  formatter wrap them. `@author` is optional in modern Javadoc; add it only when
  the project's doc policy requires it.

## Content

- State preconditions, null handling, mutation of arguments, and thrown
  conditions.
- `@throws` names the exception type and the condition; "if invalid" is not
  enough.
- Overrides: `{@inheritDoc}` or document only the specifics. Do not repeat the
  superclass contract.
- Use `{@code ...}` and `{@link ...}` where the project already does; in a
  Markdown comment prefer backticks and `[Element]` links.

## Deprecation

- `@Deprecated` annotation plus `@deprecated` tag: replacement, the version or
  date the deprecation takes effect when the project's doc convention records
  one, and what breaks. Keep both in sync.

## Toolchain

- `javadoc` and doclint report missing tags on public API; `-Xdoclint:all` can
  fail the build. Satisfy the tool, then add the constraints above.

## Sources

- JEP 467, Markdown Documentation Comments: https://openjdk.org/jeps/467
- Oracle javadoc guide: https://docs.oracle.com/en/java/javase/21/docs/specs/man/javadoc.html
- Checked 2026-10.
