# skills

Agent skills by [flow-entity](https://github.com/flow-entity), installed with the [`skills`](https://github.com/vercel-labs/skills) CLI. A skill is a `SKILL.md` file with a `name` and `description`; every agent that follows the [Agent Skills](https://agentskills.io) convention reads the same file.

## Install

```bash
# See what the repository offers
npx skills add https://github.com/flow-entity/skills --list

# Install one skill into the current project
npx skills add https://github.com/flow-entity/skills --skill code-comment-guidelines

# Install globally, non-interactive, for one agent
npx skills add https://github.com/flow-entity/skills --skill code-comment-guidelines -g -a claude-code -y

# Install everything
npx skills add https://github.com/flow-entity/skills --all
```

`-a` accepts any agent the CLI supports (`claude-code`, `cursor`, `codex`, `opencode`, `windsurf`, ...). Omit it and the CLI installs to the agents it detects, or asks. Add `--copy` when symlinks are not supported on the target machine. Re-run the same command to update, or `npx skills update`.

## Skills

| Skill | Use it when |
| --- | --- |
| [code-comment-guidelines](skills/code-comment-guidelines/SKILL.md) | Writing or editing a comment; reviewing a diff that adds comments; refactoring code that carries existing comments. |

## Authoring rules

```text
skills/<skill-name>/SKILL.md              # required, frontmatter name + description
skills/<skill-name>/references/**/*.md    # optional, read only when SKILL.md points to it
```

- `name` is kebab-case and matches the directory name, so `--skill <name>` resolves to one directory.
- `description` states what the skill does, when to use it, and what it does not cover. It is the only text an agent sees before deciding to load the skill, so it does the routing work.
- `SKILL.md` stays short and decision-dense. Long examples, rulebooks, and reference tables move to `references/` with an explicit trigger for reading them.
- Keep language-specific rules in `references/languages/<language>.md` and ecosystem-specific rules in `references/ecosystems/<topic>.md`, with an index and trigger in `SKILL.md`. The core stays language-neutral, and a reference file loads only when its language or ecosystem is involved.
- When the catalog outgrows one level, add a category directory. The installer walks `skills/` three levels deep, so `skills/<category>/<skill-name>/SKILL.md` still installs by skill name.

## License

MIT
