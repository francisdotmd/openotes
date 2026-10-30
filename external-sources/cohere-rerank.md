---
title: Rerank — Cohere documentation
description: Rerank models sort text inputs by semantic relevance to a specified query.
type: source
source_url: https://docs.cohere.com/v2/docs/rerank
media_type: text/html
date_fetched: 2026-10-30
author: Cohere
preservation: text-extracted
tags:
  - source
  - immutable
  - layer-ingest
  - text
---
Rerank models sort text inputs by semantic relevance to a specified query. They are often used to sort search results  returned from an existing search solution. Learn more about using Rerank in the best practices guide.

Latest ModelDescriptionModalityContext LengthEndpoints
rerank-v5.0-proOur highest-quality reranker. A multilingual model that allows for re-ranking English and non-english documents and semi-structured data (JSON). This is better suited for complex, reasoning-heavy queries and long or domain-specific documents than its fast variant. Supports Context Focus and calibrated relevance scores.Text32kRerank
rerank-v5.0-fastA light version of rerank-v5.0-pro, this is a multilingual model that allows for re-ranking English and non-english documents and semi-structured data (JSON). This model is better suited for low latency and high throughput use-cases than its pro variant. Supports Context Focus and calibrated relevance scores.Text32kRerank
rerank-v4.0-proA multilingual model that allows for re-ranking English and non-english documents and semi-structured data (JSON). This is better suited for state-of-the-art quality and complex use-cases than its fast variant.Text32kRerank
rerank-v4.0-fastA light version of rerank-v4.0-pro, this is a multilingual model that allows for re-ranking English and non-english documents and semi-structured data (JSON). This model is better suited for low latency and high throughput use-cases than its pro variant. Supports Context Focus and calibrated relevance scores.Text32kRerank
rerank-v3.5A model for documents and semi-structured data (JSON). Performs well in English and non-English languages; supports the same languages as embed-multilingual-v3.0.Text4kRerank
rerank-english-v3.0A model that allows for re-ranking English Language documents and semi-structured data (JSON).Text4kRerank
rerank-multilingual-v3.0A model for documents and semi-structure data (JSON) that are not in English. Supports the same languages as embed-multilingual-v3.0.Text4kRerank

For each document included in a request, Rerank combines the tokens from the query with the tokens from the document and the combined total counts toward the context limit for a single document. If the combined number of tokens from the query and a given document exceeds the model’s context length for a single document, the document will automatically get chunked and processed in multiple inferences. See our best practice guide for more info about formatting documents for the Rerank endpoint.

Rerank 5
Rerank 5 is our latest reranking model family, available in two tiers. Both support Context Focus and calibrated
relevance scores, and share the same endpoint and request format.
rerank-v5.0-pro: Highest ranking quality. Best for complex or reasoning-heavy queries and long or
domain-specific documents.
rerank-v5.0-fast: Best for low-latency, high-throughput workloads such as high-volume search and agent loops.
Model details
Context length: 32,768 tokens (query and document combined)
Number of parameters: ~2B (Pro), ~500M (Fast)
Model architecture: Proprietary
Input: Text. Pass semi-structured data (JSON, tables, records) as YAML strings
Output: Documents ranked by relevance, each with a calibrated score between 0 and 1
Languages: 100+, with a focus on English, Chinese, Korean, Hindi, Japanese, Spanish, German, French, Arabic,
Russian, and Portuguese
Endpoint: Rerank
Best for
Improving search relevance on top of existing keyword, vector, or hybrid search, with no re-indexing
Selecting context for RAG and agentic retrieval, and reducing the tokens sent to the LLM
Setting relevance thresholds, for example to trigger another search or return “I don’t know”
Matching and triage, such as tickets to known issues or regulations to internal policies
Context Focus
Context Focus returns the most relevant passages within each document in the same Rerank call, so you can send only
those passages to your LLM or highlight them in your interface. It’s off by default. To enable it, add
relevant_spans to a Rerank 5 request:
"relevant_spans": {"mode": "enabled", "threshold": 0.3, "max_spans": 10}
Each returned span has start and end character offsets into the original document and a score. A higher
threshold removes more text, and max_spans caps the number of spans per document. If no passage meets the
threshold, relevant_spans is omitted for that result. Context Focus is available through the Cohere API.
Known limitations
Rerank 5 accepts text only.
Rerank 5 doesn’t support instructions that change how relevance is judged.
Context Focus works best for targeted queries. Broad requests such as summarization need the full document, and
tables, forms, and code split less cleanly than prose.
Availability
Cohere API, subject to rate limits
Model Vault (pricing)
Microsoft Foundry (Pro, Fast)
Amazon SageMaker (Pro, Fast)
North
Private deployments in your own VPC or on-premises: contact our team
Getting started
Reranking quickstart
Rerank overview and best practices
Rerank API reference
Integrations, including LangChain,
LlamaIndex, and Elasticsearch
Rerank 5 release announcement