---
name: rag-tool
description: Manages text and markdown documents for semantic storage and retrieval. Use this skill to chunk, label, store, refresh, and search document collections within a reusable vector store, optionally utilizing metadata filters.
---

# RAG Tool

Follow this workflow exactly to prepare document content for storage and retrieval.

Do not add verification steps unless the user explicitly asks for them.

## Internal Paths

Use this fixed internal workspace-local database:

- `./.rag-tool/docs.db`

This path is fixed. Do not vary it during normal skill use.

## Resolve Input

Always preprocess the text yourself before running `cvstore` or `cvquery`.

- Read plain text or markdown directly.
- For PDFs or other binary documents, use a text transcription, not the binary file itself.
- Do not use custom scripts or programmatic pipelines to generate cleaned input text.
- Do not invent content that is not present in the source.
- Rewrite only when needed to remove decorative boilerplate, repeated navigation text, or formatting noise that would hurt retrieval quality.
- Preserve the source meaning. Do not add new facts.

## Tool Readiness

- Check with `command -v cvstore` and `command -v cvquery`.
- If either command is missing, read [references/chunkvec-install.md](references/chunkvec-install.md) and attempt installation.
- Retry the missing check after installation. If the required command is still missing, stop and report.
- Request unrestricted network or escalated execution directly in the tool call for `cvstore` and `cvquery`, since both require network access.
- Do not inspect shell profiles, environment files, or arbitrary filesystem files to discover API keys.
- If `cvstore` or `cvquery` reports an auth or config failure, report the error and ask the user to configure `DEEPINFRA_API_KEY` or `config.json`, then retry.

## Mode Selection

Map user intent into one of these two modes:

- `store`: the user wants to add or refresh content in the database
- `search`: the user wants retrieval from the database

Use `store` for requests such as:

- store this file
- ingest these notes
- add this document to the workspace database

Use `search` for requests such as:

- search my notes
- find what the source says about the new policy
- query the stored material for project guidelines

After choosing the mode, load only the relevant mode reference:

- `store` -> [references/store-mode.md](references/store-mode.md)
- `search` -> [references/search-mode.md](references/search-mode.md)

Do not load both mode files unless the request truly spans both workflows.

## Stable Doc IDs

`doc` ids must be deterministic across store and search runs.

Derive a lowercase kebab-case `doc` from one stable source identity:

- prefer the source file stem, such as `chapter1.md` -> `chapter1`
- otherwise use an explicit logical title supplied by the user
- do not invent a fresh one-off name when stable retrieval later may matter

Keep `doc` focused on the document identity.

Examples:

- source text for `chapter1.md` -> `chapter1`
- derived notes for `chapter1.md` -> `chapter1`
- quiz for `chapter1.md` -> `chapter1`

If the user provides pasted text with no stable file or title and later
doc-specific retrieval may matter, ask for a short logical name before storing.

## Shared References

- For chunking and label examples during ingest, read [references/chunking-and-labels.md](references/chunking-and-labels.md).
