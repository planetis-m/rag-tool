# Search Mode

Use this mode when the user wants retrieval from the document database.

## Command Shape

Run search with:

```bash
cvquery [--doc=DOC] [--kind=source|derived] [--page=N] [--label=TEXT] QUERY DB.sqlite
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
- `page` is exact integer match
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
- `page` only from explicit page references that clearly map to the stored numbering scheme
- `label` only when the user explicitly filters by topic or scoped subject, using stable topic phrases such as `Regularization`, `Vector Search`, or `Chain Rule`

Do not infer filters when the cue is ambiguous.

- Do not treat generic page references as `page` unless the stored material uses page metadata.
- Do not invent a `doc` filter from a loose description.
- Do not invent a new base name during search.
- Do not convert topic wording alone into a filter unless the user explicitly asks to constrain the search.
- If confidence is low, leave the text as a plain semantic query instead of guessing.

If the user does not explicitly request filtering, pass only the raw semantic query string and no filter flags.

## Search Execution

- Always use the internal database.
- Pass one raw `QUERY` positional argument to `cvquery`.
- Pass query filters through `cvquery` flags only when constrained retrieval is needed.
- Run `cvquery` against that database path.
- Use only the filters the user actually needs.

## Outputting Results

- Evaluate the retrieved chunks against the user's specific query.
- Filter out any irrelevant or overly broad chunks that do not directly answer the query.
- Rerank the remaining chunks so the most directly relevant ones appear first.
- Output the exact text of these filtered and reranked chunks verbatim.
- Do not summarize, synthesize, rephrase, or interpret the retrieved chunks.
- Output the exact text of the retrieved chunks verbatim, without raw metadata.
- Do not add any introductory framing, commentary, or transitional phrases (e.g., do not say "Here are the results...", "The material says...", or "Based on the text...").
- Do not offer unsolicited follow-up searches or list alternative queries.
- If no results match, simply say so directly.
