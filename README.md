# Doc Assistant Skill

`doc-assistant` is an installable Agent Skill for preparing semantically coherent
`chunkvec` ingest and query workflows.

## Features

It provides two modes:
- `store`: chunk text, assign stable topic labels, and store it with `cvstore`
- `search`: search the stored database with `cvquery`, using filters only when
  the user explicitly asks to narrow scope

This skill is designed for source material that needs to be stored and searched
through `chunkvec` while preserving document ids, source/derived kind,
positions, and reusable topic labels.

It uses one internal workspace database and one stable doc-id scheme. Users
should think in terms of `store` and `search`, not database paths.

## Requirements

- **`cvstore` and `cvquery`**: Required to ingest and query `chunkvec` data.
- **DeepInfra API Key**: Required by `cvstore` and `cvquery`.
  - Set it via `DEEPINFRA_API_KEY` (recommended).
  - Or provide it via `config.json` next to the real `cvstore` and `cvquery`
    executables.

## Installation

### Using Codex
Codex recommends installing non-built-in skills using the `$skill-installer`.
Prompt Codex with:

```text
$skill-installer install the skill from repo planetis-m/doc-assistant with path .
```

### Manual Install
Clone directly into your agent's scanned skills path:

```bash
git clone https://github.com/planetis-m/doc-assistant.git ~/.agents/skills/doc-assistant
```

## Modes

### Store
Use `store` when the user wants to add material to the internal workspace
database. One store run creates one logical artifact.

```text
Use $doc-assistant in store mode on notes.md.
```

```text
Use $doc-assistant to store chapter1.md.
```

```text
Use $doc-assistant to store derived notes for chapter1.md.
```

Store mode writes sparse chunk markers and passes shared metadata globally on
`cvstore`. Chunks carry `pos` and optional `label`; `doc` and `kind` are
not repeated in every marker.

Example shape:

```text
<chunk pos=1 label="Intro">
Embeddings map text into vectors where similar meanings stay close.

<chunk pos=2>
Nearest-neighbor search compares a query vector with stored vectors.
```

Example command shape:

```text
cvstore --doc=chapter1-source --kind=source INPUT.txt ./.doc-assistant/chunkvec.sqlite
```

### Search
Use `search` when the user wants to retrieve from the internal workspace
database.

```text
Use $doc-assistant in search mode for: how do embeddings help search?
```

```text
Use $doc-assistant in search mode and search only in my chapter1 notes for regularization.
```

```text
Use $doc-assistant in search mode and search within the chapter1 source for vector search.
```

Semantic search is the default. The skill should translate filters only when
the user explicitly asks to limit scope with wording such as `only`, `within`,
`restrict`, `filter`, or `search in`.

For example, `search only in my chapter1 notes for regularization` should map
to the stable `doc="chapter1-notes"` id plus `kind=derived`, and may add a
topic filter because the scope request is explicit. `cvquery` now takes one raw
query string positional argument, not a query file.

## Stable Doc IDs

The skill should generate the same `doc` ids every time from the same source
identity.

- source, original, transcript -> `<base>-source`
- notes, summary, study notes, lecture, ELI5 -> `<base>-notes`
- quiz -> `<base>-quiz`
- flashcards -> `<base>-flashcards`
- essay -> `<base>-essay`

Examples:

- `chapter1.md` stored as source -> `chapter1-source`
- derived notes for `chapter1.md` -> `chapter1-notes`
- quiz from `chapter1.md` -> `chapter1-quiz`

If the user wants multiple artifact types, the skill should create separate
ingest files and separate `cvstore` runs rather than mixing types in one file.
