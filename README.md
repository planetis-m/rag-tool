# RAG Tool Skill

`rag-tool` is an installable Agent Skill for preparing semantically coherent
`chunkvec` ingest and query workflows.

## Features

It provides two modes:
- `store`: chunk text, assign stable topic labels, and store it with `cvstore`
- `search`: search the stored database with `cvquery`, using filters only when
  the user explicitly asks to narrow scope

This skill is designed for source material that needs to be stored and searched
through `chunkvec` while preserving document ids, source/derived kind,
optional page metadata, reusable topic labels, and provenance paths when available.

`doc` names the document identity, such as `chapter1`. Use `kind` for `source`
versus `derived`. Subtypes such as notes, quiz, or flashcards are not
first-class exact query filters in the current CLI.

## Requirements

- **`cvstore` and `cvquery`**: Required to ingest and query `chunkvec` data.
- **DeepInfra API Key**: Required by `cvstore` and `cvquery`.
  - Set it via `DEEPINFRA_API_KEY` (recommended).
  - Or provide it via `config.json` next to the real `cvstore` and `cvquery`
    executables.

## Installation

### Using Codex
Install the published `rag-tool` skill with `$skill-installer`, using the repo
and path where this renamed skill is hosted.

### Manual Install
Copy or clone this skill directory into your agent's scanned skills path as
`~/.agents/skills/rag-tool`.

## Modes

### Store
Use `store` when the user wants to add material to the search database.

```text
Use $rag-tool in store mode on notes.md.
```

```text
Use $rag-tool to store chapter1.md.
```

```text
Use $rag-tool to store derived notes for chapter1.md.
```

### Search
Use `search` when the user wants to retrieve from the stored material.

```text
Use $rag-tool in search mode for: how do embeddings help search?
```

```text
Use $rag-tool in search mode and search only in my chapter1 notes for regularization.
```

```text
Use $rag-tool in search mode and search within the chapter1 source for vector search.
```
