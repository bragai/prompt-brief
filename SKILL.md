---
name: prompt-brief
description: "How to write a brief — a prompt the user pastes back as the task — for a feature, a surface or a rewrite in any codebase. Use when the user asks for \"a brief\", \"a prompt brief\", \"write me the prompt for X\", or \"the same prompting approach\": map the code with read-only subagents first, then write the brief in a fixed shape with every claim traceable to the map. Do not build; deliver the brief."
license: MPL-2.0
metadata:
  author: bragai
  version: "1.0"
---

# Prompt brief

A brief is a prompt the user pastes back as the task. It has to be good
enough that the run which follows needs no clarifying questions, and
specific enough that a second reader could check the result against it.
The ones that work are built the same way every time: learn the project's
rules, map the code with subagents, then write about what the thing is and
does, in a fixed shape, with every claim traceable to the map.

## 1. Pin the subject

From the ask and any screenshots or links, name in one line: the surface
or feature, who it is for, and the single job it has. If two things are
one subject to the user, the brief covers them as one. Say this line back
to the user only if the ask is genuinely ambiguous; otherwise proceed.

## 2. Learn the house rules before the map

Read, in this order, whatever exists: `CLAUDE.md` / `AGENTS.md`, a design
system or contributing doc (`DESIGN.md`, `CONTRIBUTING.md`, `docs/`), the
component library or registry index, the lint config for rules that bite
(banned imports, required aliases, formatting). These become the brief's
standing non-negotiables. Do not invent rules the project does not have;
do not restate the ones it does at length — cite the doc and section.

## 3. Map, do not remember

Never write a brief from memory of the code. Launch read-only explore
agents, **one per surface or module, in parallel**, using
`references/mapping-prompt.md`. Each asks for the same things: every state
and its exact copy, the data and endpoints behind it, the debts with
`file:line`, how it is mounted or called, the backend routes or schema it
touches, and the shared pieces and doc sections that will govern the
rewrite. Ask for structured bullets, quoted strings, a word cap, and **no
recommendations** — the map is facts, the brief is the opinion.

Add the survey variant when the brief must decide what to expose (a
provider's capabilities, an API, a schema).

While the agents run, read yourself the doc sections and shared modules
the work will live under. Tell the user the maps are being gathered and
that the brief follows when they land. Do not start writing until every
map is in; if one fails, rerun it rather than guessing that part.

The only exception: the surface is a single file you have already read in
full this session. Then say so and skip the agent.

## 4. Write the brief

Markdown, `# Brief: <what it becomes>`, then these sections in this order.
900–1700 words; longer means the scope is two briefs.

**Thesis.** Two paragraphs. First, what is wrong today, in concrete
particulars from the map: the file names, the number of layers, the copy
that leaks internals, the borrowed defaults, the missing record. No
adjectives doing the work of facts. Second, what it becomes: one bold
phrase naming the idea, then the person's experience of it, ending with
what they should be able to tell in two seconds.

**Approach.** The technical bet and why: what is built, what is reused
untouched (name the stores, hooks, endpoints, tables), what is deleted at
the end, and the alternative that was rejected with the reason. This is
where "data layer stays as it is" and "shared library first, add to it
when a piece is needed twice" live.

**Scope.** Bold-led paragraphs, one per thing the person sees or calls: a
screen, a state ("Overview, before a database"), a dialog, an endpoint, a
table. Organise by what it is and does, not by file — a brief organised by
file reads as a diff and gets rejected. Each paragraph names the states,
the exact copy in backticks, the shared components or modules by name,
the sizes and limits, and the behaviour at the edges (narrow widths,
empty, error, someone else's data). Say what is withheld and where it says
"coming" once.

**Non-negotiables.** Eight to ten numbered rules, each a bold lead and a
sentence, each checkable. Standing ones in most codebases: use the shared
library, never raw controls beside it; every colour and size from the
system's tokens; copy written for the person, not the pipeline, in
sentence case, with the internal nouns banned by name; secrets never shown
back; destructive actions confirm; withheld features say so once with no
dead handlers; the data layer untouched unless Approach says otherwise;
ownership misses answer not-found; existing tests keep passing. Add the
ones this work needs (a type scale, a keyboard rule, a streaming rule, a
size gate).

**What to cover.** The regression surface: a fixture route, a story, a
seed script or a test file that renders or exercises every state without
a live session, and the list of those states. Themes and widths that
matter, or the inputs and tenants that matter.

**A focused section, when one piece deserves its own spec** — a component
API, a table, a route contract.

**Verification.** Bullet checks a reader can run: greps that must come
back empty, a measurement in devtools, a query that must answer, a spec
that must exist, the fixture clean in every state; then the project's own
tail: its lint, typecheck, test and build commands by name; stage; ask
before committing; the doc section the work adds.

**Close** with at most one decision the user must make before the run
starts, stated with a default.

## 5. Quality bar

- Every file, store, endpoint, copy string and number in the brief comes
  from the map or from files you read. If the map did not say it, do not
  write it.
- Name things by what the person controls, in the product's words, never
  the system's.
- Sentences carry one instruction each. No "consider", no "maybe", no
  "should probably". A brief is a spec, not a suggestion.
- The house rules are cited, not restated; the brief adds only what is new
  for this work.
- Comments in code stay short; the brief is the long form.

## 6. Deliver

One message: a single-sentence lead ("Here is the brief. Paste it back
when you're happy with it."), then the brief. Do not begin building. When
the user pastes it back, that is the task: execute it in full, in order,
and verify against its own Verification list; report any deviation from
its rules as a deviation, not silently.

`references/exemplar-billing-page.md` is one brief written this way, for an
invented product; read it for the shape and the register, not the rules.
