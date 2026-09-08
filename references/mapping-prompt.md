# Mapping prompt template

One prompt per surface or module, to a read-only explore agent. Fill every
bracket from the ask and the project; delete lines that do not apply. Run
the agents in parallel.

```
You are mapping code in a <framework/stack> project at <absolute path>
(read-only; do not edit; no git commands that change anything). I need a
concise, factual map of the "<Surface or module>" so I can write a brief
for <the work>.

Cover:
1. <Entry folder or file> (and anything else under it). For each file:
   every state it renders or handles (loading, empty, list, editing,
   confirm, error, <domain states>), the exact copy strings (titles,
   descriptions, button labels, badges, hints, empty-state text, error
   messages), the data shape it uses, and which stores, services and
   endpoints it reads or calls (grep <store names>, <api client>, paths
   like <route prefix>). Note validation rules, masked or write-only
   values, and anything marked "coming soon", stubbed or feature-flagged.
2. Debts: hardcoded values, inline styles or colours, raw controls where a
   shared component exists, duplicated helpers, dead code, third-party
   tokens. Give file:line refs.
3. How it is mounted or called: the route, view, menu item, job or event
   that reaches it, and any other place that links to it.
4. The backend side, briefly: <routes folder or module> route list
   (paths + verbs, one line each), the tables or schema it touches
   (columns, constraints, row-level rules, indexes), and the auth or
   ownership pattern the routes use.
5. Shared pieces available for the work: the component or module index
   (names only) and the props or signatures of <the primitives the work
   will need>.
6. The doc sections that govern this kind of work (<CLAUDE.md / DESIGN.md
   / CONTRIBUTING.md> — line refs only).
7. Existing tests for this area (files and what they cover) and how tests
   are run here.

Report as structured bullets with file paths and line numbers; quote
strings exactly; keep it under ~<900–1500> words. No recommendations, just
the map.
```

## The survey variant

When the brief has to decide *what to expose* (a provider API, a
capability list, a schema), add one more agent that inventories rather
than maps:

```
You are surveying what <provider or system> can offer <the audience>, so
a designer can decide which capabilities to expose in <the surface>. Read
<the schema, types or docs> (search it with grep for <pattern>) and skim
<the hooks or client> to note what the project already calls. Produce a
concise report (no more than ~120 lines) grouping every capability by
area: <areas>. For each: the path or name, one line of what it does for
the person in plain language, whether the project already uses it
(yes/no), and read-only versus write. Do not propose designs; just the
inventory.
```
