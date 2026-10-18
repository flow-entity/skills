# Go doc comments

Read when writing or editing a package, exported, or `Example` comment. The gate
and the never-write list stay in `SKILL.md`; this file adds Go's shape and
toolchain rules.

## Shape

- Every exported name gets a doc comment directly above it, with no blank line
  between. In an application-internal package, add one only when the gate passes
  or a linter requires it. Unexported names are optional: add one only when the
  gate passes.
- The first sentence is complete and begins with the declared name:
  `// SettleOrder settles ...`, `// Package settlement ...`.
- Name-first is syntax, not content. It does not excuse a comment that only
  repeats the signature.
- `gofmt` canonicalizes doc-comment features (lists, links, headings) and
  preserves paragraph line breaks; it does not rewrap prose. Semantic linefeeds
  therefore survive, which is why one-sentence-per-line is safe when the file
  uses it.

## Content

- Funcs: what the function returns, or what it does when called for side effects.
  Boolean functions use "reports whether"; drop "or not".
- Document error conditions in prose (`It returns ErrInvalidTaxID when ...`); Go
  has no `@throws`.
- Document special cases, concurrency guarantees, and the meaning of the zero
  value when it is not obvious.
- Keep the internal algorithm out of the doc comment; leave it to a comment in
  the body. Caller-observable complexity may stay in the doc.
- `Example` functions are executed docs; prefer them to prose examples.
- Package comment: one per package, in one file, first sentence starting
  `Package x ...`.

## Deprecation and notes

- Add a `Deprecated:` paragraph before retiring an exported symbol. The prefix is
  machine-read: tools warn on use and pkg.go.dev hides the docs by default.
- Notes use `MARKER(uid): body`, for example `// TODO(user1): ...` and
  `// BUG(user2): ...`. The uid names who can explain it.

## Toolchain

- `go doc`, pkg.go.dev, and gopls render these comments; there are no tags.
- `golint`/`revive` and `staticcheck` flag missing or malformed doc comments.
  Satisfy them, then keep the constraints above.

## Sources

- Go Doc Comments: https://go.dev/doc/comment
- Go Code Review Comments: https://go.dev/wiki/CodeReviewComments
- Checked 2026-10.
