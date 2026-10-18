# JSDoc and TSDoc

Read when writing a `/** */` block in TypeScript or JavaScript. The gate and the
never-write list stay in `SKILL.md`.

## Shape

- `/** */` attaches to the declaration that follows. A block at the top of the
  file before the first declaration attaches to that declaration; file-level
  notes use a plain `//` block.
- JSDoc tags are optional. TSDoc defines syntax, not a requirement to tag every
  parameter. `eslint-plugin-jsdoc` or API Extractor can enforce them; when they
  do, each tag must add a constraint.
- `@remarks` for caller-relevant detail that would bloat the summary, `@example`
  for usage, `@deprecated` for retirement.
- `@privateRemarks` for internal-only rationale that must not appear in public
  docs; use it when a public symbol needs a maintainer note.

## Content

- Exported functions: preconditions, null/undefined behavior, side effects,
  thrown errors. The signature already carries the types; do not restate them.
- Exported types and fields: meaning, unit, sentinel (`-1`, `undefined`), and
  invariants. Use `@internal` / `@public` release tags when API Extractor is in
  the build.
- `@throws` only for errors the caller must handle; list the condition.
- `@param` when the name does not carry the meaning (range, unit, null rule),
  not as a restatement.

## Suppressions

- `// eslint-disable-next-line rule -- reason` is one comment; ESLint supports
  the `--` description.
- `// @ts-expect-error: reason` needs the reason on the same line or directly
  above. A bare suppression is a defect.

## Toolchain

- `tsc`, API Extractor, TypeDoc, and ESLint consume these comments. Enable the
  repo's jsdoc rules rather than hand-enforcing tag style.

## Sources

- TSDoc: https://tsdoc.org/
- TypeScript JSDoc reference: https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html
- Checked 2026-10.
