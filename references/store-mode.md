# Store Mode

Use this mode when the user wants to add or refresh content in the document database.

## Contents

- [Command Shape](#command-shape)
- [Chunking](#chunking)
- [Topic Labels](#topic-labels)
- [Metadata](#metadata)
- [Ingest Input](#ingest-input)
- [Store Execution](#store-execution)

## Command Shape

Run ingest with:

```bash
cvstore --doc=DOC --kind=source|derived [--source=RELATIVEPATH] INPUT.txt DB.sqlite
```

Use the internal database path from `SKILL.md`.

## Chunking

The chunking policy must align with the embedding model used by `cvstore`, which defaults to `Qwen/Qwen3-Embedding-0.6B`.

- Chunk by topic boundary, not by page shape.
- Each chunk should cover one semantic unit or one tightly related concept cluster.
- Preserve enough nearby context that the chunk still makes sense when retrieved alone.
- Split when the dominant topic, argument, procedure, or example changes.
- Prefer sentence-boundary splits.
- Keep short lead-in context with lists, definitions, formulas, or procedure steps so the embedding captures what the item is about.
- Do not combine unrelated topics just to make a chunk longer.
- Do not emit empty chunks.

Use moderately conservative chunk sizes:

- preferred: about `100-280` words
- soft upper bound: about `350` words
- hard ceiling: about `450` words unless a clean split is impossible
- shorter chunks are acceptable for atomic content such as a definition, theorem, quiz item, or short procedure

## Topic Labels

Use `label` when it improves retrieval grouping. It is optional, not required.

- Use short human-readable topic phrases, preferably `2-5` words.
- Use stable Title Case noun phrases such as `Vector Search`, `Regularization`, or `Chain Rule`.
- Reuse the exact same label across chunks that belong to the same retrieval group.
- Change the label only when the dominant topic actually changes.
- Avoid labels that are too generic, such as `Notes`, `Concept`, or `Summary`.
- Avoid labels that are too narrow, such as sentence-level paraphrases.
- Avoid label drift across similar content. Do not alternate between near-synonyms for the same topic.
- Keep labels descriptive but not overly specific.
- Omit `label` when a chunk is still clear and useful without it.

If the source needs more examples for chunking or label reuse, read [chunking-and-labels.md](chunking-and-labels.md).

## Metadata

Split ingest metadata into two levels.

Global ingest metadata, passed once on `cvstore`:

- `doc`: stable logical document id for the document identity
- `kind`: one of `source`, `derived`
- `source`: optional provenance path passed via `--source`

Per-chunk metadata, written inside each `<chunk ...>` marker:

- `pos`: integer source locator within that `doc`, such as a page or slide number
- `label`: optional short topic string

Also provide `cvstore --source=RELATIVEPATH` on ingest so stored rows keep a useful provenance identity when that provenance is known.

Apply these rules:

- Derive `doc` from the stable document identity, such as `chapter1`.
- Use `--kind=source` for original material or faithful transcription.
- Use `--kind=derived` for generated or rewritten material, including notes, lectures, quizzes, flashcards, essays, and summaries.
- `pos` should follow the source's own numbering when available, such as page or slide numbers.
- Multiple chunks may share the same `pos`. `chunkvec` keeps chunk order separately through `ordinal`.
- If the source has no native numbering at all, use sequential positions only as a fallback.
- `label` is optional.
- For quizzes, keep the question and its answer in the same chunk so retrieval returns a complete item without extra linking metadata.
- One ingest file must represent exactly one logical artifact, so `doc` and `kind` stay fixed for the whole run.
- Multiple artifact variants may share the same `doc` when they come from the same document identity.
- If the user wants multiple artifact types, produce separate ingest files and separate `cvstore` runs.

## Ingest Input

Write the material as repeated `<chunk ...>` markers plus non-empty chunk bodies.

Example:

```text
<chunk pos=12 label="Regularization">
Dropout disables random activations during training.

<chunk pos=12>
Regularization reduces overfitting by constraining model behavior.
```

Rules:

- Do not emit legacy `<page ...>` markers.
- Keep only the supported chunk-level metadata fields. Do not invent extra attributes.
- Do not write `doc` or `kind` inside chunk markers.
- Do not write legacy `position=` inside chunk markers.
- Trim decorative boilerplate that hurts retrieval quality.
- Keep chunk bodies non-empty and semantically coherent.
- Prefer chunks that stand on their own during retrieval.
- `label` is optional, not required.

## Store Execution

- Write the ingest input to an agent-managed temporary file when needed.
- Always use the internal database.
- Derive stable `doc` ids and pass them via `cvstore --doc=...`.
- Pass `--kind=source` or `--kind=derived` on `cvstore`.
- Pass `--source` when a meaningful provenance path is known.
- Run `cvstore` against that database path.
- Never mix multiple `doc` or `kind` values in one ingest file.

When providing `--source`, choose it with these rules:

- if there is a real source path or stable logical artifact path, use that as `--source`
- do not use an internal temporary file path as `--source`
