# C and C++ comments

Read when writing a declaration, definition, or kernel-doc block. The gate and
the never-write list stay in `SKILL.md`; read this file with the project's style
guide. The guide sets density; this file and `SKILL.md` set content.

## Shape

- Google C++: when the project follows Google style, almost every function
  declaration gets a comment describing use; Google exempts obvious accessors,
  and private methods in `.cc` follow the same rule only when they are not
  obvious. Definition comments describe operation (tricks, a step overview, why
  this approach).
- Linux kernel: at the function head, comment WHAT the code does and possibly
  WHY; per-statement WHAT is N2 narration. Put the comment at the head, not
  inside the body; keep small inline notes for something clever or ugly.
- File comment: what the file declares and what does not belong there. LLVM
  requires a header on every source file; Google expects a file comment. When
  the project follows either guide, write one; otherwise C1 stays conditional.
- Class comment: purpose, usage, synchronization assumptions.
- Do not duplicate a doc comment in the header and the implementation. Put the
  public contract in the header; put implementation notes in the `.cc`.
- Prefer `//` for C++ and `/* ... */` for C89, headers included from C, or the
  kernel's multi-line style. Match the file: Linux opens `/*` on its own line
  and aligns each line with a leading ` * `; do not reflow an existing block
  into a different style.
- Use an inline parameter comment only when the argument name cannot carry the
  intent: `emitName(/*Prefix=*/nullptr)`.
- Every `#endif` carries its condition: `#endif  // CONFIG_FOO`. The guard is a
  navigation aid, not narration.

These density expectations come from the project's style guide, not from the
gate. The guide can require a comment; the gate decides what that comment must
say.

## Content

- Declarations: inputs and outputs, nullability, ownership and lifetime of
  pointer or reference arguments, mutation of output arguments, performance
  implications.
- Data: comment each member that carries a sentinel (`-1`, `nullptr`) or an
  invariant the type does not state. Kernel style declares one item per line so
  each can carry its own comment.
- kernel-doc for exported kernel API; do not add boilerplate that repeats the
  signature. Doxygen tags only when the project runs Doxygen, and each tag must
  add a constraint.
- Preprocessor-disabled code: `#if 0` is not a comment, so N5 does not apply.
  Treat it as a temporary disable only: name the removal condition, prefer
  deletion, and do not leave it indefinitely (SKILL.md, N5).

## Deprecation and TODO

- Google's TODO format is `TODO(identifier):` plus a specific date or event,
  where the identifier is a person, bug, or other owner. Go uses
  `TODO(uid):`; the Linux coding style defines no TODO shape, so use the
  project's marker (`TODO`, `FIXME`, `XXX`) and tracker convention.
- A deprecation comment names the replacement, the removal condition or
  timeline, and what breaks for a caller who stays; add the since-version when
  the project's doc convention records one.

## Toolchain

- Doxygen, kernel-doc, and clang-tidy consume these comments. License and SPDX
  headers are policy and are never reformatted.

## Sources

- Google C++ Style Guide, Comments: https://google.github.io/styleguide/cppguide.html#Comments
- Linux kernel coding style, Commenting: https://docs.kernel.org/process/coding-style.html#commenting
- LLVM Coding Standards, Commenting: https://llvm.org/docs/CodingStandards.html#commenting
- Checked 2026-10.
