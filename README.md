# prompt-brief

[![Installs](https://skills.sh/b/bragai/prompt-brief)](https://skills.sh/bragai/prompt-brief)

An agent skill that makes your coding agent write the brief before it
writes the code.

A brief is a prompt you paste back to the agent as the task. It is the
difference between "redesign the billing page" and a page-long spec that
names the files, the states, the copy, the rules that must hold, and how
to check the result. The agent researches the codebase, writes the brief,
and stops. You read it, edit it, and paste it back. The run that follows
needs no clarifying questions, and anyone can check the result against
the brief afterwards.

It follows the [Agent Skills](https://agentskills.io) format, so it works
with Claude Code, Codex, Cursor, Gemini CLI and every other agent the
[`skills`](https://github.com/vercel-labs/skills) CLI supports. From the
team building [BragAI](https://bragai.dev).

## Install

```bash
npx skills add bragai/prompt-brief
```

For one agent only, or into your user directory so every project has it:

```bash
npx skills add bragai/prompt-brief -a claude-code
npx skills add bragai/prompt-brief -g
```

## Use

Ask for a brief in plain words:

> Write me a brief for the settings page. It should feel like the rest of
> the app.

> I want thumbs up / thumbs down on every answer. Use the same prompting
> approach.

Or invoke it by name, with the ask after it:

```
/prompt-brief move invoice PDF generation off the request path: a queue with retries and a dead-letter table, idempotent on invoice id, a status endpoint the page can poll, a webhook when the file is ready, and a backfill for the invoices already stored
```

The agent replies with the brief and nothing else. When it reads right,
paste the whole thing back as your next message. That is the task.

## What the agent does

1. **Pins the subject.** One line: the surface or feature, who it is for,
   and its single job.
2. **Learns the house rules.** Reads `CLAUDE.md` or `AGENTS.md`, the design
   or contributing doc, the component library index, the lint config.
   These become the brief's standing rules, cited rather than restated.
3. **Maps the code with subagents.** One read-only subagent per surface,
   in parallel, each returning a factual map: every state and its exact
   copy, the data and endpoints behind it, the debts with `file:line`, how
   it is mounted, the backend routes and schema it touches, the shared
   pieces available. No recommendations; the map is facts, the brief is
   the opinion. The agent does not write until every map is in.
4. **Writes the brief** in a fixed shape (below).
5. **Delivers it and stops.** No building until you paste it back.

## The shape of a brief

| Section | What it holds |
| --- | --- |
| **Thesis** | What is wrong today, in particulars from the map; then what it becomes, in one bold phrase and the person's experience of it. |
| **Approach** | The technical bet, what is reused untouched, what is deleted, and the alternative that was rejected and why. |
| **Scope** | One paragraph per thing the person sees or calls, with states, copy in backticks, component names, sizes, edge behaviour. Organised by what it is, never by file. |
| **Non-negotiables** | Eight to ten numbered, checkable rules. |
| **What to cover** | The regression surface: a fixture, a story or a test that exercises every state, and the list of states. |
| **Verification** | Checks a reader can run: greps that must come back empty, a measurement, a query, the project's own lint, typecheck, test and build. |
| **One decision** | At most one thing you must decide before the run starts, with a default. |

A brief runs 900 to 1,700 words. Longer means it is two briefs.

See [`references/exemplar-billing-page.md`](references/exemplar-billing-page.md)
for a complete one, written for an invented product. The
[mapping prompt template](references/mapping-prompt.md) is what the agent
sends each subagent.

## Why it helps

- **Fewer clarifying questions.** The agent asks the codebase, not you.
- **Scope you can see before the work starts.** The brief is where you
  say "not that dialog" or "keep the old table", when it costs nothing.
- **A spec to check against.** Verification is written before the code,
  so "done" means something.
- **Consistency across runs.** Every brief has the same shape, so the
  second one is as good as the first.

## What it is not

It does not build anything, and it does not replace your own rules. If the
project has no design doc or component library, the brief says so and
works with what exists rather than inventing conventions.

## Layout

```
SKILL.md                              the skill
references/mapping-prompt.md          the subagent prompt template
references/exemplar-billing-page.md   one complete brief, invented product
LICENSE                               MIT
```

## License

MIT. See [LICENSE](LICENSE).
