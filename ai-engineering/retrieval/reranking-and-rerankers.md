---
title: Reranking and rerankers
description: "A detailed guide to reranking in RAG: how rerankers score retrieved candidates, complement embeddings, and trade off precision against latency."
tags:
  - ai-engineering
  - retrieval
  - reranking
---
# Reranking and rerankers

A RAG system can have a capable language model and still answer poorly if retrieval gives it the wrong evidence. A **reranker** is a second-stage judge: it takes results already found by a retriever, scores how well each result matches the user’s question, and reorders them. **The retriever finds possibilities; the reranker decides which retrieved evidence deserves the top positions.**

## Why use two stages?

The stages solve different problems:

| Stage | Main goal | What it does | Main limitation |
| --- | --- | --- | --- |
| First-stage retrieval | Speed and recall | Quickly finds a broad candidate set with vector search, keyword search, or both. | It may return related passages that do not directly answer the question. |
| Reranking | Precision | Judges each candidate against the full question and sorts the candidates again. | It can only rank passages the first stage returned, and it adds latency and compute. |

The first stage is the net; reranking is the sorting table. A net that is too narrow misses useful documents. A weak sorting table lets loosely related results rise to the top.

## How a reranker works

1. **Receive a query and candidate passages.** The retriever may use vector search, keyword search, or a hybrid of the two. It returns a manageable candidate batch rather than asking the reranker to search the whole collection.
2. **Score each query–passage match.** The reranker considers the full question together with each candidate and estimates how relevant that passage is to that question.
3. **Sort by the new scores.** The result is a reordered candidate list. The reranker does not write the final answer.
4. **Select context for generation.** The application sends only the highest-ranked passages that fit its context budget to the language model.

A common implementation uses a cross-encoder-style model that evaluates the query and one passage together. That lets it judge how the parts of the question relate to the passage, rather than relying only on the earlier vector-search score. Exact model internals vary.

```text
Question → first-stage search → candidate passages
                              → reranker scores question + each passage
                              → sorted passages → selected context → answer
```

The transcript mentions retrieving “20 or 50” candidates as an example, then sending the top few to generation. Those numbers illustrate the two-stage pattern; they are not recommended settings for every system.

## Bi-encoder vs. cross-encoder

Here, **query** means the user’s question. A **passage** is a chunk of text from a source document that retrieval returns as a candidate. The reranker judges how well each passage answers the query.

The key difference is **whether the query and passage are encoded separately or evaluated together**.

| | Bi-encoder | Cross-encoder |
| --- | --- | --- |
| What it receives | Query and passage separately | Query and passage together as a pair |
| Output | A vector for each text; compare vectors to find likely matches | A relevance score for that specific query–passage pair |
| Best role | First-stage retrieval across a large collection | Reranking a smaller candidate set |
| Trade-off | Efficient: passage vectors can be prepared ahead of a query | More computation: score each candidate for the current query |

A bi-encoder turns the query into one vector and each passage into its own vector. Vector search compares the query vector with the stored passage vectors and returns nearby candidates. This is why it works well as the broad first search: passage vectors can be indexed and reused for many queries.

A cross-encoder instead evaluates a pair such as `(question, passage)` together and returns a score for that pair. Because the score depends on both inputs, the system runs it on the candidates for the current question, then sorts those candidates by score. It is useful for making a more focused judgment, but it adds latency and compute, so it is normally applied after retrieval has narrowed the collection.

### Using them together

```text
All passages
  → bi-encoder: embed/index passages; embed query; retrieve top-K
  → cross-encoder: score (query, passage) for each candidate; reorder
  → select top passages for the language model
```

For example, a bi-encoder may find several passages about checkout timeouts, cache settings, and deployment history. The cross-encoder can then move the specific postmortem about the cache change above general timeout guidance. The first stage finds plausible evidence; the second stage improves the ordering among what was found. Neither stage guarantees the passage is true, and the cross-encoder cannot recover a passage missing from the first-stage candidates.

## How it works with embeddings

When the first-stage retriever uses embeddings, it represents the question and passages as vectors and uses vector search to find candidates. The reranker then compares the **original question text** with those candidate passages and improves their order. Embeddings help answer, “Which passages might be related?” Reranking asks, “Which of these best addresses the whole question?”

They are complementary, not alternatives: reranking can follow embedding/vector search, keyword search, or a hybrid search. But if the useful passage never enters the candidate set, the reranker cannot recover it. Improve retrieval when the evidence is missing; try reranking when the evidence was found but buried.

## Worked example: admin access after an SSO migration

**Question:** “How do I reset admin access after single sign-on migration?”

Initial retrieval may return passages about password resets, admin roles, SSO setup, and migration notes because each overlaps with some of the query’s terms. The reranker compares each passage with the complete question and can move the migration troubleshooting note above the more generally related pages.

```text
Initial candidates: password reset | admin roles | SSO setup | migration troubleshooting

After reranking:   migration troubleshooting | admin roles | SSO setup | password reset
```

This does not mean the reranker discovered a new document. It selected the strongest match from the candidates already retrieved.

## More examples

- **Checkout timeouts:** For “Why did the checkout service start timing out after the cache change?”, retrieval may find generic timeout guidance, cache configuration, deployment history, and a postmortem. Reranking can move the relevant postmortem and rollout note above generic documentation.
- **Reversed wire transfer:** For “Why was my wire transfer reversed after compliance review?”, retrieval may find wire limits, reversal definitions, KYC checks, and a specific compliance exception policy. Reranking can promote that exception policy over generic transfer-failure explanations.

These examples illustrate why better ordering can help the generator receive more relevant evidence. It does not guarantee that the evidence is true, authoritative, or sufficient, and it cannot guarantee a correct answer.

## Why ranking matters in RAG

The generator receives a limited set of context. If the best passage is somewhere in the first-stage top 20 but does not make it into the top few passages sent to the model, the model may reason from related but weaker evidence. The answer can sound polished while missing the point. Reranking aims to increase the odds that the useful evidence reaches the prompt.

That is why checking only whether retrieval found the right chunk is not enough. Also check whether it rose high enough to be included in the final context. The transcript contrasts a relevant chunk somewhere in the top 20 with one in the top three; those are examples of the gap, not universal thresholds.

## Costs and limits

Reranking examines candidates more carefully than the fast first-stage search, so it adds latency and compute. It is usually applied to a narrowed candidate set, not the entire corpus. Increasing the candidate batch may give the reranker more chances to find a strong passage, but it also gives it more candidates to score.

Remember the boundaries:

- **Missing from candidates:** retrieval did not find the evidence; improve recall or search.
- **Present but ranked too low:** reranking may help improve its position.
- **Well-ranked but still wrong answer:** inspect source quality, context assembly, and generation; ranking alone cannot verify truth.

## When to consider a reranker

Try reranking when manual review shows that the retriever often finds relevant material but the final answer still misses the point because that material is buried. The transcript highlights nuanced queries, long-tail wording, and collections with many near-duplicate or loosely related passages as promising cases. If the relevant document is not retrieved at all, fix the retrieval stage first.

## A simple explanation

Imagine a librarian pulls ten books that might answer your question. A second helper checks your exact question against each book description and puts the best match first. The first helper is retrieval; the second helper is the reranker. The second helper can sort only the books already pulled.
