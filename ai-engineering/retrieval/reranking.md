---
title: Reranking and rerankers
description: How rerankers refine search results, how cross-encoder rerankers score query-document pairs, and when to use them in retrieval-augmented generation.
tags:
  - ai-engineering
  - retrieval
  - reranking
---

# Reranking and rerankers

A **reranker** takes a query and a set of already-retrieved candidates, scores how relevant each candidate is to that query, and sorts the candidates by those scores. It is a second-stage ranking step: a search system first finds a manageable candidate set, then the reranker helps decide which candidates deserve the top positions. [Sentence Transformers describes cross-encoder rerankers as scoring input pairs and reranking top-k results](../../external-sources/sentence-transformers-cross-encoder-usage.md); [Cohere describes reranking results from an existing search system](../../external-sources/cohere-rerank.md).

For RAG, the usual sequence is:

```text
Question
  → first-stage search (keyword, vector/embedding, or hybrid)
  → candidate passages
  → reranker scores question–passage pairs
  → keep the best passages
  → send those passages to the language model
```

Reranking complements [embeddings](./embeddings.md): retrieval gets plausible candidates from a large collection; reranking refines their order. A reranker cannot promote a relevant passage that the first-stage search never returned.

## Explain it like you’re 10

Imagine you ask a librarian, “Which book explains how bees make honey?” The librarian quickly pulls ten books that might help. A second helper reads your question and checks each book’s description more carefully. The helper puts the book that actually explains honey-making at the top, ahead of books that merely mention bees.

The first helper is search. The second helper is the reranker. It sorts the books it was given; it cannot choose a book that was never pulled from the shelves.

## High-level example

Suppose someone asks: **“How do I reset my password?”** A first-stage search might return these passages, perhaps because they contain related words:

1. “To reset your password, choose **Forgot password?** on the sign-in page. We’ll email you a reset link.”
2. “If you cannot sign in, check whether your account uses single sign-on.”
3. “Password requirements: use at least twelve characters.”
4. “Change your email address in account settings.”

A reranker compares the question with each passage and moves the direct reset instructions to the top. It may keep the sign-in and password-requirement passages next, while pushing the email-address passage down. The language model then receives the best few passages, rather than every initial result. Reranking search results and selecting context for RAG are documented uses of rerankers. [Cohere’s Rerank documentation](../../external-sources/cohere-rerank.md)

## Deep dive: what happens to each candidate

A common reranker design is a **cross-encoder**. In this design, the model receives a pair—typically `(query, document)`—and produces a relevance score for that pair. Sentence Transformers documents pair scoring and ranking passages; Cohere describes combining the query tokens with each document’s tokens before scoring. [Sentence Transformers](../../external-sources/sentence-transformers-cross-encoder-usage.md) · [Cohere](../../external-sources/cohere-rerank.md)

Conceptually, the operation looks like this:

```text
score(query, passage_1) → 0.91
score(query, passage_2) → 0.27
score(query, passage_3) → 0.73

sort by score descending → passage_1, passage_3, passage_2
```

Those numbers are illustrative, not scores from a real model. The important output is the **ordering**: the first passage is judged more relevant to this query than the others.

### Why it is often a second stage

A cross-encoder evaluates a query-document pair for each candidate. Sentence Transformers notes that this often gives better ranking performance than a bi-encoder, but is slower because it computes for each pair rather than each text. That makes it practical to use a fast search step to narrow a large collection, then spend more computation on reranking just those candidates. [Sentence Transformers’ comparison](../../external-sources/sentence-transformers-cross-encoder-usage.md)

The candidate count is therefore a quality/latency trade-off:

- **Too few candidates:** the right passage may be missed before reranking begins.
- **More candidates:** the reranker has more chances to find a useful passage, but it must score more pairs.
- **Too many or too-long candidates:** ranking can add latency and may hit model input limits. Cohere, for example, counts query and document tokens together against its per-document context limit and describes chunking documents that exceed it. [Cohere’s input limits](../../external-sources/cohere-rerank.md)

There is no universal candidate count or cut-off. Choose values by testing on your own queries, corpus, quality needs, and latency budget.

### Scores are not always probabilities

A reranker’s score is a model signal for sorting, not automatically a calibrated probability that a passage is correct. Score scales vary: the Sentence Transformers documentation notes that some MS Marco models return logits, and applying a sigmoid changes the score scale without changing the ranking. Cohere documents calibrated 0–1 relevance scores for its Rerank 5 models specifically. Do not assume another model’s score has that interpretation, or reuse a threshold across models without checking its documentation and behavior. [Sentence Transformers](../../external-sources/sentence-transformers-cross-encoder-usage.md) · [Cohere](../../external-sources/cohere-rerank.md)

## A worked retrieval example

**Question:** “Can I return unworn shoes after 30 days?”

The first-stage search searches a larger policy collection and returns a small candidate list. A reranker then scores each candidate against the exact question. The following scores are made up to illustrate the process; they are not output from a particular model.

| Candidate returned by search | Illustrative reranker score | Why it might rank here |
| --- | ---: | --- |
| “Unworn footwear can be returned within 30 days of delivery.” | 0.94 | Directly addresses item condition and the time period. |
| “Most products may be returned within 60 days; some categories have exceptions.” | 0.76 | Relevant to returns, but less specific about shoes. |
| “Shoes are available in standard and wide sizes.” | 0.08 | Shares the product topic, but does not answer the return question. |
| “Orders usually ship within three business days.” | 0.02 | Unrelated to the question. |

The RAG application can pass the top policy passages to the language model. The reranker has improved their order; it has **not** established which policy is authoritative, resolved a conflict between policy versions, or proved that the policy is true. The application still needs to retrieve the right source and preserve useful metadata such as document version and authority.

## Conceptual implementation

This pseudocode shows the boundary between retrieval and reranking. The exact API varies by library or provider:

```python
candidates = search(query, limit=candidate_count)
ranked = rerank(query, [item.text for item in candidates])
context = ranked[:passage_count]
answer = language_model.answer(query, context)
```

In a real system, keep each result’s original document ID, source, and metadata attached while sorting, so the final answer can cite or link back to the right material. Consider a relevance threshold only after evaluating what scores mean for the selected model. Some rerank APIs also offer features such as relevance thresholds or returning relevant spans, but those are provider-specific rather than requirements of reranking itself. [Cohere’s documented use cases and Context Focus behavior](../../external-sources/cohere-rerank.md)

## What a reranker does—and does not do

**It can:**

- Reorder candidates from keyword, vector, or hybrid search.
- Promote passages that better match the full meaning of a query, even when their wording differs.
- Help select a smaller context set for RAG.

These are documented reranking use cases; exact performance depends on the model, query, corpus, and candidate set. [Cohere](../../external-sources/cohere-rerank.md) · [Sentence Transformers](../../external-sources/sentence-transformers-cross-encoder-usage.md)

**It does not automatically:**

- Search the entire corpus or find candidates omitted by the first-stage search.
- Verify that a passage is true, current, or from an authoritative source.
- Guarantee that the top-scored result is sufficient to answer the question.
- Replace access controls, deduplication, source selection, or answer citations.

Treat its score as a relevance aid. The surrounding retrieval system remains responsible for candidate coverage, source quality, and how results are used.

## Practical evaluation checklist

Test with representative questions and judge whether the results actually answer them. Compare the original search order with the reranked order, including cases where the useful passage is initially lower down. Also measure added latency and the number and length of passages sent onward. This helps determine whether reranking improves the final experience enough to justify the extra scoring step.

Useful questions to ask during evaluation:

- Was the best answer-containing passage in the initial candidate set?
- Did reranking move it above merely related passages?
- Did the final selected context retain enough detail to answer correctly?
- Did the system behave sensibly when no candidate was relevant?
- Is the result still fast and affordable for the expected workload?

## When to use one

Reranking is worth trying when the initial search often returns relevant material but places it too low, or when a RAG application needs to choose a smaller set of passages for its context. It is less useful when first-stage retrieval already returns a tiny, consistently well-ordered set—or when its latency is unacceptable. Benchmark against the current system rather than assuming a reranker will help.

## Sources

- [Cohere Rerank documentation](../../external-sources/cohere-rerank.md)
- [Sentence Transformers: Cross-Encoder usage](../../external-sources/sentence-transformers-cross-encoder-usage.md)
- Related: [Embeddings](./embeddings.md)
