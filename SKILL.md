---
name: doc-assistant
description: Prepare, chunk, label, store, and query source and derived documents for `chunkvec` using concrete document ids plus the `doc`/`kind`/`position`/`label` metadata contract. Use when a task involves turning text or markdown into semantically coherent chunked ingest input or filtered query input for `cvstore` and `cvquery`.
---

# Doc Assistant

Follow this workflow exactly to prepare document content for `chunkvec`.

## Internal Paths

Use this fixed internal workspace-local database:

- `./.doc-assistant/chunkvec.sqlite`

This path never changes during normal skill use and should not be presented as
a user-facing choice.

For agent-managed temporary input files:

- place ingest and query text files under `./.doc-assistant/`
- do not require the user to name those paths explicitly

## Resolve Input

This skill is text-first.

- If the source is plain text or markdown, read it directly.
- If the source is a PDF or another binary document, work from a text or markdown transcription rather than the binary file itself.
- Do not invent content that is not present in the provided source.

## Process chunkvec Input

Always prepare the text yourself before running `cvstore` or `cvquery`.

- Do not rely on upstream headings, page breaks, or paragraph boundaries as chunk boundaries by default.
- Rewrite only when needed to remove decorative boilerplate, repeated navigation text, or formatting noise that would hurt retrieval quality.
- Preserve the source meaning. Do not add new facts.

### Installation

- Check with `command -v cvstore`.
- Check with `command -v cvquery`.
- If either command is missing, read [references/chunkvec-install.md](references/chunkvec-install.md) and attempt installation.
- Retry both `command -v` checks after installation. If either command is still missing, stop and report.

### Execution

- Request unrestricted network or escalated execution directly in the tool call when installation or `chunkvec` execution requires it.
- Do not inspect shell profiles, environment files, or arbitrary filesystem files to discover API keys.
- If `cvstore` or `cvquery` reports an auth or config failure, report the error and ask the user to configure `DEEPINFRA_API_KEY` or `config.json`, then retry.

### Usage

- `cvstore INPUT.txt DB.sqlite`
- `cvquery QUERY.txt DB.sqlite`

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
- find what the source says about dropout
- query the stored material for regularization

## Stable Doc IDs

`doc` ids must be deterministic across store and search runs.

Derive a lowercase kebab-case `base` from one stable source identity:

- prefer the source file stem, such as `chapter1.md` -> `chapter1`
- otherwise use an explicit logical title supplied by the user
- do not invent a fresh one-off base when stable retrieval later may matter

Then apply exactly one typed suffix:

- source, original, transcript -> `-source`
- notes, summary, study-notes, lecture, eli5 -> `-notes`
- quiz -> `-quiz`
- flashcards -> `-flashcards`
- essay -> `-essay`

Examples:

- `chapter1-source`
- `chapter1-notes`
- `chapter1-quiz`

If the user provides pasted text with no stable file or title and later
doc-specific retrieval may matter, ask for a short logical name before storing.

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

Use conservative chunk sizes:

- preferred: about `80-220` words
- soft upper bound: about `300` words
- hard ceiling: about `400` words unless a clean split is impossible
- shorter chunks are acceptable for atomic content such as a definition, theorem, quiz item, or short procedure

## Topic Labels

Every skill-generated chunk should have a non-empty `label`.

- Use short human-readable topic phrases, preferably `2-5` words.
- Use stable Title Case noun phrases such as `Vector Search`, `Regularization`, or `Chain Rule`.
- Reuse the exact same label across chunks that belong to the same retrieval group.
- Change the label only when the dominant topic actually changes.
- Avoid labels that are too generic, such as `Notes`, `Concept`, or `Summary`.
- Avoid labels that are too narrow, such as sentence-level paraphrases.
- Avoid label drift across similar content. Do not alternate between near-synonyms for the same topic.
- Keep labels descriptive but not overly specific.

If the source needs more examples for chunking or label reuse, read [references/chunking-and-labels.md](references/chunking-and-labels.md).

## Choose Metadata

Every stored chunk must carry exactly these fields:

- `doc`: stable logical document id for one stored artifact
- `kind`: one of `source`, `derived`
- `position`: integer locator within that `doc`
- `label`: short topic string

Apply these rules:

- Use the stable typed-suffix scheme from `Stable Doc IDs`.
- Use a separate `doc` id for each stored artifact type derived from the same base.
- Use `kind=source` for original material or faithful transcription.
- Use `kind=derived` for generated or rewritten material, including notes, lectures, quizzes, flashcards, essays, and summaries.
- `position` is required even when the source has no native numbering. In that case, assign sequential positions.
- Multiple chunks may share the same `position`. `chunkvec` keeps chunk order separately through `ordinal`.
- `label` is required for skill-generated output even though `chunkvec` accepts unlabeled chunks.
- For quizzes, keep the question and its answer in the same chunk so retrieval returns a complete item without extra linking metadata.

## Produce Ingest Input

Write the material as repeated `<chunk ...>` markers plus non-empty chunk bodies.

Example:

```text
<chunk doc="ml-unit-3-source" kind=source position=12 label="Regularization">
Dropout disables random activations during training.

<chunk doc="ml-unit-3-notes" kind=derived position=12 label="Regularization">
Regularization reduces overfitting by constraining model behavior.
```

Rules:

- Do not emit legacy `<page ...>` markers.
- Keep only the 4 supported metadata fields. Do not invent extra attributes.
- Trim decorative boilerplate that hurts retrieval quality.
- Keep chunk bodies non-empty and semantically coherent.
- Prefer chunks that stand on their own during retrieval.
- Ensure every generated chunk includes a `label`.

In `store` mode:

- write the ingest input to an agent-managed file under `./.doc-assistant/`
- always use the internal database at `./.doc-assistant/chunkvec.sqlite`
- derive `doc` ids with the stable typed-suffix scheme before writing chunks
- run `cvstore` against that database path

Run ingest with:

```bash
cvstore INPUT.txt DB.sqlite
```

## Produce Query Input

When constrained retrieval is needed, write a query file with an optional leading `<search ...>` header, then a blank line, then the semantic query text.

Example:

```text
<search doc="ml-unit-3-notes" kind=derived position=12 label="regularization">

Why does dropout reduce overfitting?
```

Query filter behavior:

- `doc` is exact match
- `kind` is exact match
- `position` is exact integer match
- `label` is normalized substring match

In `search` mode, infer filters from natural-language intent only when the cue
is clear.

Infer these filters:

- `doc` only when the stable base artifact and artifact type are both clear, using the same typed-suffix scheme as store mode
- `kind=source` from cues like `source`, `original`, or `transcript`
- `kind=derived` from cues like `derived`, `notes`, `summary`, `quiz`, or `flashcards`
- `position` only from explicit chunk-position references that clearly map to the stored numbering scheme
- `label` from stable topic phrases such as `Regularization`, `Vector Search`, or `Chain Rule`

Do not infer filters when the cue is ambiguous.

- Do not treat generic page references as `position` unless the stored material uses pages as positions.
- Do not invent a `doc` filter from a loose description.
- Do not invent a new base name during search.
- If confidence is low, leave the text as a plain semantic query instead of guessing.

If no high-confidence filters are present, write only the semantic query text.

In `search` mode:

- write the query input to an agent-managed file under `./.doc-assistant/`
- always use the internal database at `./.doc-assistant/chunkvec.sqlite`
- run `cvquery` against that database path

Use only the filters the user actually needs.

Run search with:

```bash
cvquery QUERY.txt DB.sqlite
```
