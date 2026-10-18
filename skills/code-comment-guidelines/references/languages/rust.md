# Rust doc comments

Read when writing rustdoc, a `# Safety` section, or a `// SAFETY:` block
comment. The gate and the never-write list stay in `SKILL.md`.

## Shape

- Public items get `///` docs; the crate or module gets `//!`. Keep internal
  items out of the generated docs with `#[doc(hidden)]` or `pub(crate)`.
- Sections use `#` headings: `# Safety`, `# Errors`, `# Panics`, `# Examples`.

## Content

- `# Safety` on unsafe functions: every invariant the caller must uphold. The
  API guidelines (C-FAILURE) say unsafe functions should have one;
  `clippy::missing_safety_doc` enforces it on public ones.
- `# Errors` on fallible public API: which error variants come back and when.
- `# Panics` on functions that panic for caller-visible reasons.
- `# Examples`: executable doctests. Show why a caller would use the item, not
  only how to call it; a mechanical example is a poor example.
- Link other items with intra-doc links (`` [`Vec`] ``); a link is cheaper than a
  restatement.

## Unsafe blocks

- Put a `// SAFETY:` comment immediately above each `unsafe` block, stating why
  the block is sound and what must stay true. `clippy::undocumented_unsafe_blocks`
  (allow-by-default) checks for it; a comment placed elsewhere does not satisfy
  the lint.
- A safety argument in the function doc does not cover the block; the block
  needs its own reason.

## Deprecation

- `#[deprecated(since = "1.4.0", note = "use `settle` instead")]` carries the
  replacement and the version; the doc comment explains what breaks.

## Toolchain

- `cargo doc`, docs.rs, and doctests consume these comments.
  `#![warn(missing_docs)]` enforces public documentation.

## Sources

- Rust API Guidelines, Documentation: https://rust-lang.github.io/api-guidelines/documentation.html
- rustdoc book: https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html
- Clippy: https://rust-lang.github.io/rust-clippy/master/index.html#missing_safety_doc
- Checked 2026-10.
