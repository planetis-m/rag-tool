# Chunking And Labels

Use these examples when the source structure is not enough to decide chunk
boundaries or labels.

## Stable doc ids

Use one deterministic base name plus one typed suffix.

- `chapter1.md` as source -> `chapter1-source`
- derived notes from `chapter1.md` -> `chapter1-notes`
- quiz from `chapter1.md` -> `chapter1-quiz`

Use those ids on the `cvstore` command line, not inside each chunk marker.

## Chunking principles

- Keep each chunk on one dominant topic.
- Keep enough surrounding context that the chunk still answers a question when
  retrieved alone.
- Reuse the same label across adjacent chunks that stay on the same topic.
- Change the label only when the dominant topic shifts.

## Example: one topic over multiple chunks

```text
<chunk pos=12 label="Regularization">
Regularization reduces overfitting by limiting how closely a model can match
training noise. Common methods include penalties on weights and structured
randomness during training.

<chunk pos=13 label="Regularization">
Dropout is a regularization method that disables random activations during
training so the network cannot depend too heavily on a single pathway.
```

Both chunks stay under the same retrieval topic, so the label stays
`Regularization`.

## Example: topic shift requires a new label

```text
<chunk pos=14 label="Vector Search">
Nearest-neighbor search compares a query vector with stored vectors and returns
the closest matches by distance.

<chunk pos=15 label="Embedding Models">
Embedding models map text into vectors so semantically similar passages stay
close in vector space.
```

The labels change because the dominant topic changes.

## Example: quiz chunk keeps question and answer together

```text
<chunk pos=5 label="Chain Rule">
Question: Why is the chain rule central to backpropagation?
Answer: It decomposes the effect of each parameter through nested functions, so
gradients can be propagated layer by layer.
```

## Example: filtered search command

```text
cvquery --doc=ml-unit-3-source --kind=source --label=regularization "How does dropout reduce overfitting?" ./.doc-assistant/chunkvec.sqlite
```

Label matching is a normalized substring match, so stable reusable labels and
stable typed-suffix doc ids help keep filtered retrieval predictable.

## Example: store mode prompt

```text
Use $doc-assistant in store mode on chapter1.md.
```

Expected behavior:

- chunk and label the source
- write ingest input under `./.doc-assistant/`
- store into the internal workspace database
- run `cvstore --doc=chapter1-source --kind=source ...`

## Example: store derived notes with stable naming

```text
Use $doc-assistant to store derived notes for chapter1.md.
```

Expected behavior:

- keep the same base name `chapter1`
- run `cvstore --doc=chapter1-notes --kind=derived ...`

## Example: sparse ingest markers

```text
<chunk pos=1 label="Regularization">
Regularization reduces overfitting by limiting how closely a model can match
training noise.

<chunk pos=2>
Dropout disables random activations during training.
```

Expected behavior:

- `doc` and `kind` come from `cvstore --doc=... --kind=...`
- `position` stays in each chunk marker
- `label` may be omitted

## Example: plain semantic search

```text
Use $doc-assistant in search mode for: how do embeddings help search?
```

Expected query string:

```text
How do embeddings help search?
```

## Example: explicit filtered search

```text
Use $doc-assistant in search mode and search only in my chapter1 notes for regularization.
```

Expected behavior:

- pass `--doc=chapter1-notes --kind=derived --label=Regularization` to `cvquery`
- pass the raw query string as the `QUERY` positional argument

## Example: explicit source-scoped search

```text
Use $doc-assistant in search mode and search within the chapter1 source for vector search.
```

Expected behavior:

- pass `--doc=chapter1-source --kind=source --label=Vector Search` to `cvquery`
- pass the raw query string as the `QUERY` positional argument

## Example: topic mention without explicit filtering stays semantic

```text
Use $doc-assistant in search mode for my chapter1 notes about regularization.
```

Expected behavior:

- do not pass filter flags
- keep the request as the raw semantic query string because the user did not explicitly ask to filter

## Example: ambiguous cue should stay semantic

```text
Use $doc-assistant in search mode for what page 12 says about dropout.
```

Expected behavior:

- do not assume `position=12` unless the stored numbering is explicitly page-based
- keep the request as the raw semantic query string if that mapping is unknown
- do not invent a `doc` filter if the base artifact is unknown
