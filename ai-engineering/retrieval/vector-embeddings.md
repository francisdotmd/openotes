---
title: Vector embeddings
description: A lecture-style explanation of token embeddings, vector search, and how rerankers refine retrieved candidates.
tags:
  - ai-engineering
  - retrieval
  - embeddings
---
# Vector embeddings: from token IDs to meaning-shaped vectors

An **embedding** is a vector: an ordered list of numbers used to represent a token or other piece of information. The numbers give a model something mathematical to work with. The supplied lecture explains embeddings through words such as *king*, *queen*, and *apple*: after training, related words can have vectors that are closer together than unrelated ones.

## The simple starting point: one-hot encoding

Imagine a tiny vocabulary with four entries. Give each entry an ID, then represent it with a vector containing zeros everywhere except a `1` at that entry’s position. For example, if `cat` has ID 1 and `dog` has ID 4:

```text
cat → [1, 0, 0, 0]
dog → [0, 0, 0, 1]
```

This identifies each word, but the vectors do not express that *cat* and *dog* are related. They are just different labels. A real vocabulary can have tens of thousands of entries, so making a vector with one slot for every entry is also wasteful. These are the two problems the lecture uses to motivate embeddings.

## The embedding matrix: a learned lookup table

Instead of a vocabulary-length one-hot vector, imagine a table with one row per vocabulary item and a shorter vector in each row. The lecture’s example uses 50,000 rows and 300 columns; those are illustrative sizes, not fixed requirements.

```text
                 embedding dimensions
                 1       2       3      ...
ID 0 / word 0   0.12   -0.08    0.31    ...
ID 1 / word 1  -0.24    0.17    0.05    ...
ID 2 / word 2   0.03    0.42   -0.11    ...
...
```

Call the table `E`. If the input token has ID `i`, its embedding is the row `E[i]`. In other words, **looking up an embedding is a table lookup**: find the token ID, then fetch its row. The table starts with small random values during training, so its initial rows do not yet encode useful relationships.

## How training gives the numbers meaning

The model learns the table’s values through training:

1. It sees text and makes a prediction—for example, what token might come next.
2. The prediction is compared with the training target, producing an error (often called a loss).
3. Training updates model parameters, including embedding values, to make future predictions better.
4. Repeating this across many examples shapes the vectors. Words that occur in similar contexts can end up with related representations.

The lecture’s *king*/*queen* example is an intuition for this process: the model is not given a hand-written rule saying those words are related. Instead, training patterns shape their vectors. The transcript simplifies the optimization into “tiny adjustments” to a word’s vector; that is a teaching model for the idea, not a step-by-step account of every training update.

## What “nearby vectors” means

A vector has coordinates in a many-dimensional space. A model or search system can compare vectors mathematically. If training has made *king* and *queen* representations similar, a vector comparison can place them nearer to one another than either is to *apple*—the example from the lecture.

“Near” is about learned numerical similarity, not a dictionary definition or a guarantee that two words can be substituted in every sentence. The training objective and the data shape what relationships the vectors capture.

## A simple analogy

Think of the embedding table as a giant card catalog. A token ID is the card number; looking up the ID retrieves that card’s coordinates. At first, the coordinates are random. Training repeatedly adjusts them based on examples, so items used in related ways can end up near each other on the map.

One-hot encoding is like giving every card a unique number with no map at all. An embedding adds learned coordinates that let a model compare items by their positions.

## How embeddings fit into retrieval

In embedding-based retrieval, the first search stage uses vector representations to find candidate passages that may relate to a question. A reranker can then compare the full question with those candidates and reorder them. So embeddings help retrieve possibilities; reranking helps choose which retrieved passages best match the question. The reranker cannot recover a useful passage that retrieval did not return.

```text
Question and passages → embedding/vector search → candidate passages
                                              → reranker → best-ranked passages
```

The supplied lecture focuses on a word/token lookup-table explanation. It is a useful foundation, but it does not cover every detail of how a model creates a representation for a whole sentence or passage.

## The whole process

The token lookup table is the foundation in the lecture. In retrieval, the system uses an embedding model to represent whole passages and questions as vectors, so it can search for passages related to a question. Exact model details vary, but the stages fit together like this.

### 1. Prepare and index passages

Before a user asks anything, the system splits source documents into passages or chunks. An embedding model turns each passage into a vector. The system stores each vector with the original passage text and useful metadata, such as the document title or source, so it can return the actual text later.

```text
source documents → chunks → passage vectors + text + metadata → search index
```

This is the passage-level version of the lecture’s token lookup idea: text is represented numerically so the search system can compare it.

### 2. Embed the question and retrieve candidates

When a question arrives, the same or a compatible embedding model turns it into a query vector. Vector search compares that vector with the stored passage vectors using the index’s configured similarity or distance measure, then returns a top-K candidate set. A system may combine vector search with keyword search as well. The candidates are likely matches, not yet a carefully judged final answer set. The reranking transcript describes this first stage as a fast, broad search for recall.

```text
query text → query vector → compare with passage vectors → top-K candidates
```

The vector similarity score expresses how close the representations are under the chosen model and search metric. It is not a probability that the passage is correct, and related wording or concepts can bring in a passage that does not directly answer the question.

### 3. Rerank the candidate passages

The reranker receives the original question and the text of each candidate passage. It scores how relevant each question–passage pair is, then sorts the candidates again. For a common cross-encoder-style reranker, the question and one passage are evaluated together; this lets the scorer judge the match between the full question and that passage rather than relying only on their earlier vector-similarity score. Exact internals vary by reranker. The transcript describes this as the second, more precision-focused judge.

```text
(question, candidate 1) → relevance score
(question, candidate 2) → relevance score
(question, candidate 3) → relevance score
                              ↓
                       sort by score
```

The reranker only sees the candidates from step 2. It cannot bring back a useful passage that the first-stage search did not return.

### 4. Select context and generate

The application selects the highest-ranked passages that fit its context budget and sends them, with the question, to the language model. The model then writes an answer using that selected evidence. Reranking improves which passages are presented; it does not itself write the answer or prove that the passages are true.

### Worked example

Question: **“How do I restore admin access after an SSO migration?”**

The initial vector search might return all of these because each is related to some part of the question:

| Candidate passage | Initial vector-search position | Reranker’s likely decision |
| --- | ---: | --- |
| General password-reset instructions | 1 | Move down: about passwords, but not the migration problem. |
| SSO setup guide | 2 | Keep as supporting context, but not the direct answer. |
| Admin-role reference | 3 | Keep nearby: relevant to admin permissions. |
| Migration troubleshooting: restoring admin access | 4 | Move to the top: it addresses the combined question. |

The ranks are illustrative, not model output. After reranking, the application might send the migration troubleshooting note and admin-role reference to the language model, rather than letting the generic password-reset guide dominate. This mirrors the example in the supplied transcript.

### Where to look when results are bad

- **The useful passage never appears in the candidate set:** investigate retrieval—such as the source content, chunking, embedding model, or search configuration. Reranking cannot fix a recall miss.
- **The useful passage appears but is buried:** reranking may improve its position before context is selected.
- **The right passage reaches the model but the answer is still wrong:** inspect context assembly, source quality, and generation; ranking alone does not guarantee correctness.

This is why the two stages work together: embeddings/vector search find possibilities, and reranking chooses which of those retrieved passages deserve the limited space in the prompt.

## Key takeaways

- A vector is a list of numbers; an embedding is a learned vector representation.
- One-hot vectors identify items but do not encode their relationships.
- An embedding matrix stores a shorter vector for each vocabulary ID; retrieving one is a lookup.
- Training adjusts the matrix so its vectors become useful for the model’s task.
- In retrieval, embeddings help find candidates, while reranking can improve their order.
