# Search Mode

Use this mode when the user wants retrieval from the document database.

## Command Shape

Run search with:

```bash
cvquery [--doc=DOC] [--kind=source|derived] [--position=N] [--label=TEXT] QUERY DB.sqlite
```

Use the internal database path from `SKILL.md`.

## Query Input

Always construct one raw semantic query string for `cvquery`.

Example:

```text
Why does dropout reduce overfitting?
```

Query filter behavior is expressed on the `cvquery` command line:

- `doc` is exact match
- `kind` is exact match
- `position` is exact integer match
- `label` is normalized substring match

In `search` mode, infer filters from natural-language intent only when the cue is an explicit request to narrow scope.

Semantic search is the default.

Treat these as explicit filtering requests:

- `only`
- `just`
- `restrict`
- `limit`
- `filter`
- `within`
- `search in`
- `search only in`
- `from the source`
- `from my notes`

Infer these filters:

- `doc` only when the user explicitly requests filtering and the stable document identity is clear, using the same scheme as store mode
- `kind=source` from cues like `source`, `original`, or `transcript`
- `kind=derived` from cues like `derived`, `notes`, `summary`, `quiz`, or `flashcards`
- `position` only from explicit chunk-position references that clearly map to the stored numbering scheme
- `label` only when the user explicitly filters by topic or scoped subject, using stable topic phrases such as `Regularization`, `Vector Search`, or `Chain Rule`

Do not infer filters when the cue is ambiguous.

- Do not treat generic page references as `position` unless the stored material uses pages as positions.
- Do not invent a `doc` filter from a loose description.
- Do not invent a new base name during search.
- Do not convert topic wording alone into a filter unless the user explicitly asks to constrain the search.
- Do not treat `source` provenance paths as query filters; `cvquery` does not support them.
- Do not promise exact notes-vs-quiz-vs-flashcards filtering; that subtype is not a first-class query field.
- If confidence is low, leave the text as a plain semantic query instead of guessing.

If the user does not explicitly request filtering, pass only the raw semantic query string and no filter flags.

## Search Execution

- Always use the internal database.
- Pass one raw `QUERY` positional argument to `cvquery`.
- Pass query filters through `cvquery` flags only when constrained retrieval is needed.
- Run `cvquery` against that database path.
- Use only the filters the user actually needs.

## Interpreting Results

- Do not mechanically echo the top `k` results.
- Use the retrieved chunks to judge which results actually answer the user's question.
- Prefer chunks that directly answer the query over nearby but broader topic matches.
- Do not mention raw metadata; mention only `doc` when needed to identify the document.
- If the retrieved chunks do not support a reliable answer, say so instead of inventing one.
