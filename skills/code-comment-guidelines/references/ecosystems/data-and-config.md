# Data, schema, and config comments

Read when the change touches SQL, protobuf, OpenAPI, YAML/JSONC, shell, or
infrastructure-as-code. The gate and the never-write list stay in `SKILL.md`;
this file adds the surfaces where a comment is often the API contract or the
operational rationale.

## SQL migrations

- A migration comment states why the change exists, the lock or backfill risk,
  and whether it is reversible. Do not narrate the DDL.
- A column or table comment is part of the schema contract: unit, sentinel,
  nullability, and the business rule the constraint cannot express.
- Keep migration tool markers (`-- migrate:up`, `-- migrate:down`, `-- +goose
  Up`) as directives; do not reformat them.
- A backfill states the batch size, the throttle, and the stop condition.

## Protobuf and OpenAPI

- Comments in `.proto` and OpenAPI are generated API docs. Treat them as
  caller-facing contract: C11 for field meaning, units, and enum sentinels; C12
  for message semantics; C2 for RPC error conditions and compatibility notes.
- Do not edit generated `*_pb2.py`, `*.pb.go`, or client code; change the
  `.proto` or the spec and regenerate.
- Deprecation uses the schema's own mechanism (`deprecated = true`,
  `deprecated: true`) plus the replacement and removal condition.

## YAML, JSON, JSONC, and TOML

- YAML/JSON values are data, not comments; when the format has no comments, put
  the rationale in the nearest doc or schema `description` instead of inventing
  a key.
- JSONC/TOML comments follow the same gate; a config key's unit and default
  belong next to the key.
- Do not put secrets in comments; they are committed with the file.

## Shell and infrastructure

- `# shellcheck disable=SC2086` needs a reason (C9).
- Terraform/HCL: comment the non-obvious resource relationship, the provider
  quirk, and the destroy-order constraint; do not narrate every argument.
- Make targets and CI: comment the non-obvious ordering or the workaround, not
  the command name.

## Sources

- PostgreSQL `COMMENT`: https://www.postgresql.org/docs/current/sql-comment.html
- Protobuf style guide: https://protobuf.dev/programming-guides/style/
- OpenAPI specification: https://spec.openapis.org/oas/latest.html
- ShellCheck directives: https://www.shellcheck.net/wiki/Directive
- Checked 2026-10.
