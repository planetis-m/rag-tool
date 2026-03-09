# Chunking And Labels

Use these examples when the source structure is not enough to decide chunk
boundaries or labels.

## Chunking principles

- Keep each chunk on one dominant topic.
- Keep enough surrounding context that the chunk still answers a question when
  retrieved alone.
- Reuse the same label across adjacent chunks that stay on the same topic.
- Change the label only when the dominant topic shifts.

## Example: one topic over multiple chunks

```text
<chunk doc="ml-unit-3" kind=source position=12 label="Regularization">
Regularization reduces overfitting by limiting how closely a model can match
training noise. Common methods include penalties on weights and structured
randomness during training.

<chunk doc="ml-unit-3" kind=source position=13 label="Regularization">
Dropout is a regularization method that disables random activations during
training so the network cannot depend too heavily on a single pathway.
```

Both chunks stay under the same retrieval topic, so the label stays
`Regularization`.

## Example: topic shift requires a new label

```text
<chunk doc="ml-unit-3" kind=source position=14 label="Vector Search">
Nearest-neighbor search compares a query vector with stored vectors and returns
the closest matches by distance.

<chunk doc="ml-unit-3" kind=source position=15 label="Embedding Models">
Embedding models map text into vectors so semantically similar passages stay
close in vector space.
```

The labels change because the dominant topic changes.

## Example: quiz chunk keeps question and answer together

```text
<chunk doc="ml-unit-3-quiz" kind=derived position=5 label="Chain Rule">
Question: Why is the chain rule central to backpropagation?
Answer: It decomposes the effect of each parameter through nested functions, so
gradients can be propagated layer by layer.
```

## Example: filtered query

```text
<search doc="ml-unit-3" kind=source label="regularization">

How does dropout reduce overfitting?
```

Label matching is a normalized substring match, so stable reusable labels help
group related chunks and make filtered retrieval predictable.
