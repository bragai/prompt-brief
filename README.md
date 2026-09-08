# prompt-brief

An agent skill for writing a brief: a prompt you paste back to the agent
as the task. In the [Agent Skills](https://agentskills.io) format, so it
works with Claude Code, Codex, Cursor and every other agent the `skills`
CLI supports. From the team building [BragAI](https://bragai.dev).

## Install

```bash
npx skills add bragai/prompt-brief
```

For one agent only:

```bash
npx skills add bragai/prompt-brief -a claude-code
```

## What it does

Ask for "a brief for …" or "the same prompting approach", and the agent
researches before it writes: one read-only subagent per surface maps the
states, the copy, the data and the debts with `file:line` references. Then
it writes the brief in a fixed shape — thesis, approach, scope,
non-negotiables, what to cover, verification, and one closing decision.
Every claim traces to the map, so the run that follows needs no
clarifying questions and a second reader can check the result against it.

The skill delivers the brief and stops. Pasting it back starts the work.

## Layout

```
SKILL.md                          the skill
references/mapping-prompt.md      the subagent prompt template
references/exemplar-billing-page.md one brief written this way
LICENSE                           MPL-2.0
```

## License

Mozilla Public License 2.0. Use it anywhere, including in closed products;
if you change these files, the changed files must be shared under the same
license. See [LICENSE](LICENSE).
