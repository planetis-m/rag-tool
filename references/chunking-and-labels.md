# Chunking And Labels

Use this reference only when the source structure is not enough to decide chunk
boundaries, label reuse, or explicit search filters.

## Chunking Reminders

- Keep each chunk on one dominant topic.
- Keep enough nearby context that the chunk still answers a question when
  retrieved alone.
- Reuse the same label across adjacent chunks that stay on the same topic.
- Change the label only when the dominant topic shifts.

## Example: one topic over multiple chunks

```text
<chunk page=12 label="Regularization">
Regularization reduces overfitting by limiting how closely a model can match
training noise. Common methods include penalties on weights and structured
randomness during training.

<chunk page=13 label="Regularization">
Dropout is a regularization method that disables random activations during
training so the network cannot depend too heavily on a single pathway.
```

Keep the same label because both chunks stay in the same retrieval group.

## Example: topic shift requires a new label

```text
<chunk page=14 label="Vector Search">
Nearest-neighbor search compares a query vector with stored vectors and returns
the closest matches by distance.

<chunk page=15 label="Embedding Models">
Embedding models map text into vectors so semantically similar passages stay
close in vector space.
```

Change the label when the dominant topic changes.

## Example: explicit filtered search

```text
Use $rag-tool in search mode and search only in my chapter1 notes for regularization.
```

Expected behavior:

- pass `--doc=chapter1 --kind=derived --label=Regularization` to `cvquery`
- pass the raw query string as the `QUERY` positional argument

## Example: ambiguous cue stays semantic

```text
Use $rag-tool in search mode for what page 12 says about dropout.
```

Expected behavior:

- do not assume `--page=12` unless the stored numbering is explicitly page-based
- keep the request as the raw semantic query string if that mapping is unknown
- do not invent a `doc` filter if the document identity is unknown
