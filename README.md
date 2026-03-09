# Doc Assistant Skill

`doc-assistant` is an installable Agent Skill for preparing semantically coherent
`chunkvec` ingest and query workflows.

## Features

It provides two modes:
- `store`: chunk text, assign stable topic labels, and store it with `cvstore`
- `search`: search the stored database with `cvquery`, inferring filters from
  natural-language intent when the cue is clear

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
database.

```text
Use $doc-assistant in store mode on notes.md.
```

```text
Use $doc-assistant to store chapter1.md.
```

```text
Use $doc-assistant to store derived notes for chapter1.md.
```

### Search
Use `search` when the user wants to retrieve from the internal workspace
database.

```text
Use $doc-assistant in search mode for: how do embeddings help search?
```

```text
Use $doc-assistant in search mode for my chapter1 notes about regularization.
```

```text
Use $doc-assistant in search mode for the chapter1 source on vector search.
```

The skill should infer filters only when the intent is clear. For example,
`chapter1 notes about regularization` should map naturally to the stable
`doc="chapter1-notes"` id plus a `label` filter for `Regularization`.

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
