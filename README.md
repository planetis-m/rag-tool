# Doc Assistant Skill

`doc-assistant` is an installable Agent Skill for preparing semantically coherent
`chunkvec` ingest and query files.

## Features

It provides one workflow:
- chunks text into retrieval-friendly semantic units
- assigns stable topic labels to every chunk
- prepares marked-up ingest input for `cvstore`
- prepares optional filtered query input for `cvquery`
- runs the `chunkvec` tools when asked

This skill is designed for source material that needs to be stored and searched
through `chunkvec` while preserving document ids, source/derived kind,
positions, and reusable topic labels.

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

## Usage Examples

Invoke the skill explicitly using `$doc-assistant` in your prompts:

```text
Use $doc-assistant to turn these markdown notes into chunkvec ingest input with stable topic labels.
```

```text
Use $doc-assistant on chapter1.md, store it with cvstore, and write a filtered cvquery file for regularization.
```
