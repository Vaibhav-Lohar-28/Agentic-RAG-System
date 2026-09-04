# Curated YouTube learning path for the Agentic RAG System

**Companion to:** [`PROJECT_STUDY_PLAN.md`](PROJECT_STUDY_PLAN.md)
**Repository:** `Vaibhav-Lohar-28/Agentic-RAG-System`
**Curated and public-page checked:** 2026-09-04

This is a deliberately curated set of supplementary videos, not a claim that every repository-specific behavior has a dedicated tutorial. The project code, deployment manifests, tests, notebooks, and the official documentation listed in [`PROJECT_STUDY_PLAN.md`](PROJECT_STUDY_PLAN.md) remain authoritative. Video APIs, commands, model names, UI screens, and library behavior can age; re-check them against the repository's pinned or minimum versions before copying anything.

## How to use this guide

1. Read the corresponding section of [`PROJECT_STUDY_PLAN.md`](PROJECT_STUDY_PLAN.md) and open the repository paths listed below.
2. Watch the short or focused video first. Use a long course only for a gap that the short video exposes.
3. Reproduce the idea in this repository without copying the video's project structure, credentials, endpoints, model IDs, or deployment commands.
4. Run the relevant tests or write a small test before making a change. For infrastructure videos, prefer `docker compose config`, manifest review, and a disposable environment over a production account.
5. Record the mismatch when the video and this checkout differ. The mismatch is often the lesson: this repository has separate API and Airflow dependency environments, provider-specific paths, and documented drift.

### Labels

- **Official / primary:** the channel is the technology project, vendor, maintainer, or an official conference/event channel. This identifies provenance, not a guarantee that every example matches this repository.
- **Community / practitioner:** an independent explanation or implementation selected for clarity or useful coverage.
- **Conceptual:** prioritize the mental model; do not expect runnable project code.
- **Provider-specific:** tied to AWS, OpenAI, Jina, a particular vector store, or another vendor/library. Translate the idea to the project's provider abstraction.
- **Old-version:** the recording or its commands predate the versions in this checkout, or the ecosystem has materially changed. Use the explanation, then consult current docs.
- **Promotional:** the page contains a course, product, affiliate, sponsor, service, or channel call-to-action. This is a disclosure, not an automatic quality judgment.
- **Announcement:** the video announces a longer course rather than teaching that course. Follow the linked course and current docs if you want the complete treatment.
- **Hands-on:** includes a code or configuration walkthrough, still subject to the version/provider warnings above.

## The shortest useful route through the plan

This is the recommended order for the project's 14-session schedule. The links are repeated in the detailed sections with the exact scope and caveats.

| Study stage | Watch | Then study in this checkout |
|---|---|---|
| 1. Python and HTTP foundations | [Learn Python](https://www.youtube.com/watch?v=rfscVS0vtbw), then the [FastAPI crash course](https://www.youtube.com/watch?v=rvFsGRvj9jo) | `pyproject.toml`, `src/main.py`, `src/routers/`, `src/schemas/api/` |
| 2. Async and persistence | [FastAPI sync vs async](https://www.youtube.com/watch?v=2JPDt-Jp6fM), [Pydantic V2](https://www.youtube.com/watch?v=ok8bF8M7gjk), and [SQLAlchemy 2.0](https://www.youtube.com/watch?v=Uym2DHnUEno) | `src/config.py`, `src/database.py`, `src/db/`, `src/models/`, `src/repositories/`, `src/services/*/client.py` |
| 3. Tests and contracts | [Testing a FastAPI API](https://www.youtube.com/watch?v=SO7m7nod0ts) | `tests/conftest.py`, `tests/api/`, `tests/unit/`, `pyproject.toml` |
| 4. Containers and orchestration | [Docker full course](https://www.youtube.com/watch?v=3c-iBn73dDE), [Airflow for beginners](https://www.youtube.com/watch?v=xUKIL7zsjos) | `Dockerfile`, `compose.yml`, `airflow/Dockerfile`, `airflow/dags/` |
| 5. Ingestion | [Docling PDF parsing](https://www.youtube.com/watch?v=26thuRsxiUc) | `src/services/arxiv/`, `src/services/pdf_parser/`, `src/services/metadata_fetcher.py`, `notebooks/phase2/` |
| 6. Retrieval | [BM25](https://www.youtube.com/watch?v=TW9vHU1GpU4), [OpenSearch vectors](https://www.youtube.com/watch?v=oX0HMAztP8E), [RRF](https://www.youtube.com/watch?v=px4YBYrz0NU) | `src/services/indexing/`, `src/services/embeddings/`, `src/services/opensearch/`, `notebooks/phase3/`–`phase4/` |
| 7. RAG and evaluation | [RAG overview/course](https://www.youtube.com/watch?v=ShEOoJLSLbI), [RAG evaluation concepts](https://www.youtube.com/watch?v=qI2qQfOG0Js) | `src/routers/ask.py`, `src/services/agents/`, `data/golden_dataset.json`, `tests/eval/`, `notebooks/phase5/` |
| 8. Caching and observability | [Redis caching strategies](https://www.youtube.com/watch?v=8A6s9d0jnWI), [Langfuse tracing](https://www.youtube.com/watch?v=pTneXS_m1rk), [OpenTelemetry propagation](https://www.youtube.com/watch?v=azyVG0T1aVc) | `src/services/cache/`, `src/services/langfuse/`, `src/services/logfire/`, `src/middlewares.py`, `docs/grafana_integration.md` |
| 9. Agentic RAG | [LangGraph Essentials announcement](https://www.youtube.com/watch?v=B_BCeWhyD5Q), then the [hands-on LangGraph RAG agent](https://www.youtube.com/watch?v=60XDTWhklLA) | `src/services/agents/`, `src/routers/agentic_ask.py`, `notebooks/phase7/`, `static/langgraph-mermaid.md` |
| 10. Bedrock and safety | [Bedrock Converse and inference APIs](https://www.youtube.com/watch?v=PxFWzYBbG0U), [Bedrock Guardrails](https://www.youtube.com/watch?v=gp6bGpid62E) | `src/services/bedrock_llm/`, `src/services/bedrock_guardrails/`, `scripts/create_bedrock_guardrail.py`, `bedrock-policy.json` |
| 11. Protocols and interfaces | [MCP workshop](https://www.youtube.com/watch?v=kQmXtrmQ5Zg), [A2A course announcement](https://www.youtube.com/watch?v=4gYm0Rp7VHc), [Telegram](https://www.youtube.com/watch?v=kj_5oa72aR4), [Gradio streaming](https://www.youtube.com/watch?v=RB8OZtqdeFQ) | `src/mcp_server/`, `src/routers/a2a.py`, `src/services/a2a/`, `src/services/telegram/`, `src/gradio_app.py`, `gradio_launcher.py` |
| 12. Kubernetes, AWS and CI/CD | [Kubernetes course](https://www.youtube.com/watch?v=X48VuDVv0do), [IRSA](https://www.youtube.com/watch?v=Sj_apaQFIbI), [GitHub Actions](https://www.youtube.com/watch?v=R8_veQiYBjI) | `deployment/`, `deployment/eks/`, `deployment/k8s/`, `.github/workflows/`, `docs/updated_kubernetes.md` |

The order is a learning dependency, not an execution order for the application. Do not start with the agent videos before understanding the ordinary `/ask` retrieval path and its tests.

---

## 1. Python, async Python, FastAPI, Pydantic, SQLAlchemy and pytest

### Python and FastAPI

- [**Learn Python - Full Course for Beginners**](https://www.youtube.com/watch?v=rfscVS0vtbw) — freeCodeCamp.org, 4:26:52. **Community course; foundations; older setup/examples; promotional channel CTA.** Use it for data types, control flow, functions, modules, classes, and inheritance. It is not a guide to this project's Python version or dependency management.
- [**FastAPI Full Crash Course - Python's Fastest Web Framework**](https://www.youtube.com/watch?v=rvFsGRvj9jo) — NeuralNine, 58:29. **Community; hands-on; conceptual plus basic FastAPI/Pydantic; provider-neutral.** Useful for routes, path/query parameters, HTTP methods, async-versus-sync intuition, automatic docs, validation, and exceptions. Use `src/main.py`, `src/routers/`, and `src/schemas/api/` to see how this repository's route composition differs.
- [**Python API Development - Comprehensive Course for Beginners**](https://www.youtube.com/watch?v=0sOvCWFmrtA) — freeCodeCamp.org, 19:00:27. **Community long course; hands-on; old-version; promotional course-style.** This is the broadest single supplement: FastAPI, Pydantic, PostgreSQL, SQL, SQLAlchemy, Alembic, testing, Docker/Compose, and GitHub Actions. Useful cuts from the published chapters include FastAPI at 36:21, Pydantic at 1:07:29, SQLAlchemy at 4:35:33, testing at 14:14:51, Docker at 13:26:09, and GitHub Actions at 17:34:15. It was published in 2021, so do not copy its package versions, deployment platform, authentication code, or SQLAlchemy/Pydantic syntax blindly.

### Async Python and FastAPI's sync/async boundary

- [**Python FastAPI Tutorial (Part 7): Sync vs Async - Converting Your App to Asynchronous**](https://www.youtube.com/watch?v=2JPDt-Jp6fM) — Corey Schafer, 32:10. **Community; hands-on; relatively current; SQLite/async-driver-specific.** It demonstrates when `async def` helps with I/O, why blocking work still blocks an event loop, and how async SQLAlchemy changes a route. Compare it with the project's synchronous and asynchronous clients under `src/services/`, rather than assuming that every `async` function is non-blocking.
- [**Async Awaits: Mastering Asynchronous Python in FastAPI**](https://www.youtube.com/watch?v=1z8LLSZSWHM) — PyCon DE/PyData Berlin, 30:23. **Conference; conceptual; practitioner experience; no project-specific code.** Watch this when the distinction between concurrency, parallelism, worker processes, event-loop blocking, and I/O-bound LLM calls is unclear. It is especially useful before reading the project's streaming and agent code.

Repository exercises: trace one request through `src/routers/ask.py`, `src/services/opensearch/client.py`, `src/services/bedrock_llm/client.py`, and `src/services/agents/`; identify every synchronous call made from an async function and decide whether it is safe, isolated, or a potential event-loop problem.

### Pydantic and settings

- [**Pydantic (V2) - In-depth Starter Guide**](https://www.youtube.com/watch?v=ok8bF8M7gjk) — MathByte Academy, 1:13:20. **Community; Pydantic v2; hands-on; promotional course CTA.** Covers validation errors, required/nullable fields, aliases, serialization, defaults, serializers, validators, and nested models. Map it to `src/config.py`, `src/schemas/`, `src/models/`, and the configuration tests. The video is from 2023: use the current Pydantic documentation for exact edge cases and settings behavior.

### SQLAlchemy 2.0 and PostgreSQL

- [**TUTORIAL: SQLAlchemy 2.0**](https://www.youtube.com/watch?v=Uym2DHnUEno) — Mike Bayer, hosted by Six Feet Up, 1:44:24. **Conference tutorial; primary maintainer-led; SQLAlchemy 2.0; hands-on.** This is the preferred SQLAlchemy video in this guide because it explains Core, engines, transactions, query construction, metadata, and ORM persistence in the 2.0 style. Read `src/database.py`, `src/db/`, `src/models/paper.py`, `src/repositories/paper.py`, and the separate Airflow requirements before changing database code. The API runtime and Airflow image intentionally have different SQLAlchemy constraints; do not merge their environments.

### pytest and async API tests

- [**Python FastAPI Tutorial (Part 17): Testing the API - Pytest, Fixtures, and Mocking External Services**](https://www.youtube.com/watch?v=SO7m7nod0ts) — Corey Schafer, 1:22:25. **Community; hands-on; current series; test-library/version-sensitive.** Covers `pytest`, `conftest.py`, fixtures, HTTPX's async client, database isolation, and mocking external services. Compare it with `tests/conftest.py`, `tests/api/conftest.py`, `tests/api/routers/`, `tests/unit/`, and `tests/integration/`. The repository's tests and CI are the authority on fixture scope, app lifespan, mocks, and what is actually asserted.

Primary reading after the videos: `pyproject.toml`, [Python asyncio documentation](https://docs.python.org/3/library/asyncio.html), [FastAPI documentation](https://fastapi.tiangolo.com/), [Pydantic documentation](https://docs.pydantic.dev/latest/), [SQLAlchemy 2.0 documentation](https://docs.sqlalchemy.org/20/), and [pytest documentation](https://docs.pytest.org/en/stable/).

---

## 2. Docker, Compose, Airflow, arXiv, PDFs and Docling

### Docker and Compose

- [**Docker Tutorial for Beginners - Full Course in 3 Hours**](https://www.youtube.com/watch?v=3c-iBn73dDE) — TechWorld with Nana, 2:46:15. **Community; hands-on; old-version; promotional links.** Good coverage of images, containers, commands, debugging, networks, Compose, Dockerfiles, registries, deployment, and volumes. Key cuts are Compose at 1:29:49, Dockerfile at 1:42:02, and volumes at 2:27:26. The demo uses a different application and the recording is from 2020; check `Dockerfile`, `compose.yml`, current Compose semantics, health checks, and bind mounts in this repository.

- [**How To Run Airflow with Docker On MacOS**](https://www.youtube.com/watch?v=ouERCRRvkFQ) — CK Data Tech, 34:01. **Community; hands-on; Airflow 2.10.4/Docker Compose-specific; version-sensitive; promotional links.** Useful for seeing the Airflow container, webserver/scheduler startup, DAG mounts, and local Compose workflow. The project uses Airflow 2.10.3 in `airflow/Dockerfile`, so verify every command against that image and the active `compose.yml`.

### Airflow DAGs and execution

- [**Getting Started with Airflow for Beginners**](https://www.youtube.com/watch?v=xUKIL7zsjos) — Data with Marc, 15:59. **Community practitioner; hands-on; Airflow 2-era; version-sensitive; promotional course CTA.** Covers local setup, Docker Compose/Astro alternatives, DAGs, task dependencies, Variables, XComs, TaskFlow, triggering, and the UI. The repository's actual active DAG is `airflow/dags/arxiv_paper_ingestion.py`, with helper modules in `airflow/dags/arxiv_ingestion/`; compare the video's small example with the project's five-stage ingestion chain, external services, retries, and cleanup behavior.

Study the active workflow rather than assuming a video diagram is current: `airflow/dags/arxiv_paper_ingestion.py`, `airflow/dags/arxiv_ingestion/`, `airflow/README.md`, `airflow/Dockerfile`, `airflow/requirements-airflow.txt`, and `compose.yml`. The root and Airflow requirements constrain SQLAlchemy separately from the API runtime.

### arXiv API and PDF ingestion

There is **no selected YouTube video that reliably teaches this repository's exact arXiv behavior**—weekday scheduling, query filters, XML/Atom metadata, rate limiting, PDF caching, database persistence, cleanup, and failure handling. That is intentional. Learn it from:

- `src/services/arxiv/client.py`, `src/services/arxiv/factory.py`, `src/services/metadata_fetcher.py`;
- `src/schemas/arxiv/`, `src/models/paper.py`, `src/repositories/paper.py`;
- `airflow/dags/arxiv_ingestion/fetching.py`, `airflow/dags/arxiv_ingestion/setup.py`, and `airflow/dags/arxiv_paper_ingestion.py`;
- `notebooks/phase2/phase2_arxiv_integration.ipynb` and `notebooks/phase2/README.md`; and
- the [official arXiv API user manual](https://info.arxiv.org/help/api/user-manual.html).

Treat generic “download papers” videos and scraper examples as particularly risky: arXiv rate limits, response formats, robots guidance, versioned IDs, and licensing/usage constraints are operational concerns, not merely a Python HTTP exercise.

### Docling and complex PDFs

- [**PDF Parsing with Scanned Images, Tables, Text with Docling, Claude, GPT and Llama**](https://www.youtube.com/watch?v=26thuRsxiUc) — Rajesh Srivastava, 27:58. **Community; hands-on comparison; Docling/LLM provider-specific; old-version; promotional channel CTA.** Useful for understanding why text-only PDF extraction loses tables, layout, scans, and reading order, and why parser quality affects downstream chunking and retrieval. It compares several tools and proprietary/vision models; do not treat those comparisons or model names as project requirements.

Map the video to `src/services/pdf_parser/docling.py`, `src/services/pdf_parser/parser.py`, `src/services/pdf_parser/factory.py`, `src/schemas/pdf_parser/`, `notebooks/phase2/`, and the PDF parser tests. Verify the current Docling API and the parser's page limits, timeout, fallback, and error behavior before changing ingestion.

Primary reading: [Apache Airflow DAG concepts](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html), [Airflow tasks](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html), [Airflow Docker Compose documentation](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html), [Docling documentation](https://docling-project.github.io/docling/), and the [arXiv API manual](https://info.arxiv.org/help/api/user-manual.html).

---

## 3. BM25, embeddings, OpenSearch vector/hybrid search, RRF and RAG

### BM25 and lexical retrieval

- [**A no nonsense intro to BM25**](https://www.youtube.com/watch?v=TW9vHU1GpU4) — Abhishek Thakur, 15:45. **Community; conceptual plus small search demo; provider-neutral.** Covers term frequency, inverse document frequency, document-length normalization, `k1`, and `b`. Use it to understand why exact terms, identifiers, acronyms, and rare words can be valuable in paper search.

### Embeddings and OpenSearch vectors

- [**Getting started with OpenSearch as a vector database**](https://www.youtube.com/watch?v=oX0HMAztP8E) — OpenSearch Project, 10:03. **Official / primary; hands-on; OpenSearch-specific; provider/version-sensitive.** Explains vector spaces, embeddings, K-NN, neural/search pipelines, indexing, and hybrid search. It is the most direct visual supplement for `src/services/embeddings/`, `src/services/indexing/`, and `src/services/opensearch/`.

- [**Hybrid Search | Amazon OpenSearch Service**](https://www.youtube.com/watch?v=KzZpIalVOb8) — Amazon OpenSearch Service, 09:26. **Official / primary; OpenSearch/AWS provider-specific; hands-on; version-sensitive.** Demonstrates lexical BM25 versus semantic retrieval, score normalization, and hybrid queries. The video discusses OpenSearch 2.11-era hybrid-query normalization; the repository documents OpenSearch 2.19.5 and uses its own query/pipeline configuration. Use the concept, not the video's JSON or managed-AWS assumptions.

### Reciprocal Rank Fusion

- [**Reciprocal Rank Fusion (RRF) - How to Stop Worrying about Boosting**](https://www.youtube.com/watch?v=px4YBYrz0NU) — OpenSource Connections / Haystack EU, 38:49. **Community conference talk; conceptual; search-engine practitioner; provider-neutral.** Explains why raw-score normalization and hand-tuned boosting are fragile, then walks through rank-based fusion of lexical and dense result lists. Compare it with `src/services/opensearch/query_builder.py`, `src/services/opensearch/index_config_hybrid.py`, `src/services/indexing/hybrid_indexer.py`, `workflows/phase4_hybrid_search.md`, and the OpenSearch score-ranker configuration.

### End-to-end RAG

- [**Learn RAG, LangChain, Vector DBs, with this project - Full Course**](https://www.youtube.com/watch?v=ShEOoJLSLbI) — PropTech Founder, 1:29:20. **Community; hands-on; OpenAI/FAISS/LangChain/LangGraph-specific; old-version; promotional newsletter/services.** A useful project-shaped overview of ingestion, vector similarity, RAG, memory, tools, and a graph flow. It uses a different application, Flask, SQL data, FAISS, OpenAI, and external APIs; use it to build vocabulary, not to select this repository's OpenSearch schema, provider, prompt, or graph topology.

Use the repository's actual RAG path as the lab: `src/routers/ask.py`, `src/routers/hybrid_search.py`, `src/services/opensearch/`, `src/services/indexing/`, `src/services/embeddings/`, `src/services/openai_llm/`, `src/services/bedrock_llm/`, `notebooks/phase3/`–`phase5/`, and `workflows/phase3_keyword_search.md`–`phase5_rag_pipeline.md`.

### A concept-to-code map

| Concept to learn | Start with these repository paths | Video aid and limit |
|---|---|---|
| Sparse/lexical retrieval | `src/services/opensearch/query_builder.py`, `src/services/opensearch/index_config_hybrid.py`, `workflows/phase3_keyword_search.md` | [BM25 intro](https://www.youtube.com/watch?v=TW9vHU1GpU4); conceptual, not an OpenSearch schema prescription |
| Dense embeddings | `src/services/embeddings/`, `src/services/indexing/text_chunker.py` | [OpenSearch vector introduction](https://www.youtube.com/watch?v=oX0HMAztP8E); provider/OpenSearch-specific |
| K-NN/vector index | `src/services/opensearch/index_config_hybrid.py`, `src/services/indexing/hybrid_indexer.py` | [OpenSearch vector introduction](https://www.youtube.com/watch?v=oX0HMAztP8E); verify dimensions and engine in code |
| Hybrid retrieval | `src/services/opensearch/query_builder.py`, `src/routers/hybrid_search.py`, `workflows/phase4_hybrid_search.md` | [Amazon OpenSearch hybrid search](https://www.youtube.com/watch?v=KzZpIalVOb8); OpenSearch 2.11-era/provider-specific explanation |
| RRF/rank fusion | `src/services/opensearch/`, `tests/unit/services/test_opensearch_query_builder.py` | [RRF conference talk](https://www.youtube.com/watch?v=px4YBYrz0NU); conceptual, not this project's exact pipeline JSON |
| Chunking and source context | `src/services/indexing/text_chunker.py`, `src/services/pdf_parser/`, `notebooks/phase5/` | [RAG course](https://www.youtube.com/watch?v=ShEOoJLSLbI); different stack and old APIs |
| Generation and citations | `src/routers/ask.py`, `src/services/openai_llm/`, `src/services/bedrock_llm/`, `src/schemas/api/ask.py` | [RAG course](https://www.youtube.com/watch?v=ShEOoJLSLbI); inspect project prompts and response schemas instead |

Primary reading: [OpenSearch hybrid search](https://docs.opensearch.org/latest/vector-search/ai-search/hybrid-search/index/), [OpenSearch score-ranker/RRF processor](https://docs.opensearch.org/latest/search-plugins/search-pipelines/score-ranker-processor/), [OpenSearch hybrid query](https://docs.opensearch.org/latest/query-dsl/compound/hybrid/), and [Jina embeddings](https://jina.ai/embeddings).

---

## 4. Redis caching, Langfuse/OpenTelemetry observability and RAG evaluation

### Redis caching

- [**Caching Strategies With Redis**](https://www.youtube.com/watch?v=8A6s9d0jnWI) — Software With Shawn, 11:29. **Community; hands-on; conceptual cache-aside/write-through; Redis-specific.** Covers cache hits/misses, TTL, cache-aside, write-through, and invalidation pitfalls. Map it to `src/services/cache/client.py`, `src/services/cache/factory.py`, `src/routers/ask.py`, and the cache-related notebook/tests. Verify serialization, key construction, TTL, error handling, and lifecycle in this checkout rather than copying the example's data model.

### Langfuse and OpenTelemetry

- [**Langfuse Intro - Observability & Tracing Deep Dive**](https://www.youtube.com/watch?v=pTneXS_m1rk) — Langfuse, 11:12. **Official / primary; product demo; OpenTelemetry-based; vendor perspective.** Shows traces, nested spans/generations, user feedback, costs, latency, exports, and integrations. Read `src/services/langfuse/`, `src/services/logfire/`, `src/middlewares.py`, `docs/grafana_integration.md`, and the relevant environment settings alongside it.

- [**OpenTelemetry: How distributed tracing really works with Python, FastAPI and requests**](https://www.youtube.com/watch?v=azyVG0T1aVc) — Adam Gardner, 09:28. **Community; conceptual plus hands-on; FastAPI/Jaeger/requests-specific.** The trace ID, span, `traceparent`, and context-propagation explanation is the useful part. The demo uses `requests`, Jaeger, localhost, and two toy services; the repository uses its own Logfire/Langfuse integrations and deployed endpoints.

- [**RAG Observability and Evaluations with Langfuse**](https://www.youtube.com/watch?v=h5hqelg0_wc) — Langfuse, 05:44. **Official / primary; product demo; conceptual plus experiment workflow; vendor perspective.** Demonstrates tracing retrieved chunks, comparing chunk sizes/overlap, and evaluating relevance and faithfulness. Use it to design experiments, not to assume that a vendor score is ground truth. Compare with `src/services/langfuse/`, `data/golden_dataset.json`, and `tests/eval/test_golden_dataset.py`.

### RAG evaluation

- [**How to evaluate a RAG system: methods and metrics**](https://www.youtube.com/watch?v=qI2qQfOG0Js) — Evidently AI, 07:07. **Official company/educational channel; conceptual; retrieval and generation evaluation.** Covers ground-truth retrieval metrics, human or LLM relevance labels, answer correctness, faithfulness/groundedness, synthetic data, and production stress testing. This is the best short conceptual entry point before redesigning the repository's golden evaluation.

- [**RAGAS: How to Evaluate a RAG Application Like a Pro for Beginners**](https://www.youtube.com/watch?v=5fp6e5nhJRk) — Mervin Praison, 08:37. **Community; hands-on; Ragas/API-version-sensitive; promotional channel CTA.** Useful for the vocabulary of `faithfulness`, `answer correctness`, context, answer, and reference data. It is from 2024 and its imports/metric API may not match current Ragas. Validate against current Ragas documentation and do not add Ragas merely because the video does.

Repository evaluation truth: `data/golden_dataset.json` and `tests/eval/test_golden_dataset.py` currently provide a structural/mocked gate, not a complete measurement of retrieval recall, citation correctness, groundedness, or factuality. The study-plan exercises in section 16 and capstone E are the intended next step.

Primary reading: [Langfuse documentation](https://langfuse.com/docs), [OpenTelemetry documentation](https://opentelemetry.io/docs/), [Ragas documentation](https://docs.ragas.io/), and the [Evidently RAG evaluation guide](https://www.evidentlyai.com/llm-guide/rag-evaluation).

---

## 5. LangGraph, agentic RAG, Bedrock Converse and guardrails

### LangGraph and agentic RAG

- [**LangChain Academy New Course: Introduction to LangGraph**](https://www.youtube.com/watch?v=29XE10U6ooc) — LangChain, 02:37. **Official / primary announcement; announcement; conceptual; links to the free course.** This is not the course itself. It explains the motivation for explicit control, state, memory, human review, tools, and parallelism. Follow the linked [Introduction to LangGraph course](https://academy.langchain.com/courses/intro-to-langgraph/) and current [LangGraph documentation](https://docs.langchain.com/oss/python/langgraph/).

- [**LangChain Academy New Course: LangGraph Essentials**](https://www.youtube.com/watch?v=B_BCeWhyD5Q) — LangChain, 01:37. **Official / primary announcement; announcement; conceptual; links to the free course.** It describes state, execution control, durable/checkpointed runtime, streaming, parallelism, and human intervention. Follow the linked [LangGraph Essentials course](https://academy.langchain.com/) rather than expecting the 97-second video to teach implementation.

- [**LangGraph RAG Agent Tutorial - Basics to Advanced Multi-Agent AI Chatbot**](https://www.youtube.com/watch?v=60XDTWhklLA) — Pradip Nichite / FutureSmartAI, 54:21. **Community; hands-on; LangChain/OpenAI/Chroma-specific; version-sensitive; promotional live app/blog/course links.** Useful for router nodes, tools, memory, retrieval, grading whether context is sufficient, fallback search, and answer generation. Translate its graph to the project's actual topology and state types; do not substitute its vector store, model, web search, or routing policy.

- Optional visual companion: [**Building a RAG agent with LangChain and LangGraph Studio**](https://www.youtube.com/watch?v=WRLevY1M2lc) — Vectrix, 07:57. **Community; conceptual visual walkthrough; LangGraph Studio/reranker-specific; old-version.** It is useful for seeing intent routing, parallel branches, reranking, and citations, but it is not a code review of this repository.

Read and trace, in order: `src/services/agents/state.py`, `context.py`, `models.py`, `prompts.py`, `tools.py`, `agentic_rag.py`, `factory.py`, `src/routers/agentic_ask.py`, `src/routers/supervisor_ask.py`, `notebooks/phase7/phase7_agentic_rag.ipynb`, `static/langgraph-mermaid.md`, and `tests/api/routers/test_agentic_ask.py`. The current graph documented in the study plan is `START → guardrail → retrieve → tool_retrieve → grade_documents → generate_answer → output_guardrail`, with rewrite loops and deterministic retrieval after approval.

### Bedrock Converse and guardrails

- [**Amazon Bedrock Inference APIs and Profiles**](https://www.youtube.com/watch?v=PxFWzYBbG0U) — AWS Events, 57:20. **Official / primary; provider-specific; hands-on; model/API/version-sensitive; AWS event format.** Covers Bedrock's unified Converse API, model IDs, inference profiles, tool/function calls, and switching models. Use it with `src/services/bedrock_llm/`, `src/config.py`, and the current AWS SDK documentation. Never copy a model ID, region, IAM policy, or request shape without checking the project's provider factory and current Bedrock API.

- [**Amazon Bedrock Guardrails**](https://www.youtube.com/watch?v=gp6bGpid62E) — AWS Events, 1:00:19. **Official / primary; provider-specific; current event/demo; vendor perspective; version-sensitive.** Covers harmful-content filters, PII, prompt attacks, configuration, and demonstrations. Compare the video with `src/services/bedrock_guardrails/`, `scripts/create_bedrock_guardrail.py`, `bedrock-policy.json`, and the agentic versus standard-route guardrail paths. The repository's fail-open/error behavior is a code fact, not something to infer from the vendor demo.

Important provider boundary: the project supports OpenAI, Ollama, and Bedrock-related paths, but provider selection is not automatically transparent for every route. Study `src/services/*_llm/`, factories, health behavior, streaming translation, structured output, guardrail application, and model configuration before claiming parity.

Primary reading: [LangGraph basics](https://docs.langchain.com/oss/python/langgraph/), [Amazon Bedrock Converse](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html), [Converse API reference](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html), and [Bedrock guardrails with Converse](https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-use-converse-api.html).

---

## 6. MCP, A2A, Telegram and Gradio

These videos introduce external protocols and interface libraries. They cannot establish the repository's exact endpoint contracts, authentication, lifecycle, or tool permissions.

### MCP

- [**Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic**](https://www.youtube.com/watch?v=kQmXtrmQ5Zg) — AI Engineer, 1:44:11. **Community conference workshop with an Anthropic MCP contributor; conceptual plus hands-on; protocol/provider-ecosystem-specific; version-sensitive.** Covers MCP motivation, client/server architecture, prompts, tools, resources, and agent integration. Read `src/mcp_server/server.py`, `src/mcp_server/tools/`, `src/mcp_server/resources/`, and the MCP SDK requirements. Treat the protocol as evolving and follow the current [MCP documentation](https://modelcontextprotocol.io/).

### A2A

- [**Use A2A to connect agents across different frameworks and teams**](https://www.youtube.com/watch?v=4gYm0Rp7VHc) — DeepLearning.AI, 03:06. **Course announcement; announcement; conceptual; protocol-specific; promotional course enrollment.** It introduces Agent Cards, A2A clients/servers, lifecycle concepts, cross-framework orchestration, and the distinction between A2A (agent-to-agent) and MCP (agent-to-tools/data). Follow the linked [A2A course](https://bit.ly/4anVtk6) and current [A2A documentation](https://a2a-protocol.org/) for the complete implementation.

Read `src/routers/a2a.py`, `src/services/a2a/`, `src/services/a2a/models.py`, and the A2A tests/route schemas. Compare the project's wire contract with the current protocol instead of replacing it with a course sample.

### Telegram

- [**Create a Telegram Bot with Python (Step-by-Step Tutorial 2025)**](https://www.youtube.com/watch?v=kj_5oa72aR4) — AI & Automations, 10:35. **Community; hands-on; Telegram/python-telegram-bot-specific; provider/library-specific; promotional community/tools.** Covers BotFather, token handling, async handlers, command/message filters, and polling. The project pins `python-telegram-bot>=21,<22`; compare the video with `src/services/telegram/bot.py`, `src/services/telegram/factory.py`, `src/services/telegram/`, and the Telegram tests. Never place a bot token in source or a video command transcript.

### Gradio

- [**Streaming on chat interface using Gradio framework**](https://www.youtube.com/watch?v=RB8OZtqdeFQ) — Raj Kapadia, 09:12. **Community; hands-on; old-version; OpenAI streaming/Gradio-specific; promotional community/services.** Useful for generator/yield-based UI streaming and chat-history wiring. It uses the OpenAI Chat Completions API and an older Gradio style; the repository's `src/gradio_app.py` and `gradio_launcher.py` are the source of truth for UI events, backend URLs, and provider behavior.

Primary reading: [MCP documentation](https://modelcontextprotocol.io/), [A2A documentation](https://a2a-protocol.org/), [python-telegram-bot documentation](https://docs.python-telegram-bot.org/), and [Gradio documentation](https://www.gradio.app/docs).

---

## 7. Kubernetes, EKS, HPA, IRSA and GitHub Actions/CI/CD

### Kubernetes fundamentals

- [**Kubernetes Tutorial for Beginners - Full Course in 4 Hours**](https://www.youtube.com/watch?v=X48VuDVv0do) — TechWorld with Nana, 3:36:55. **Community; hands-on; old-version; promotional course links.** Covers Pods, Deployments, Services, Ingress, ConfigMaps/Secrets, namespaces, volumes, StatefulSets, Helm, and `kubectl`. The 2020 recording explicitly includes Helm 2/Tiller-era material; use it for the architecture and resource relationships, not current Helm commands or deprecated APIs.

Map it to `deployment/k8s/api/deployment.yaml`, `deployment/k8s/api/service.yaml`, `deployment/k8s/api/hpa.yaml`, `deployment/k8s/opensearch/`, `deployment/k8s/airflow/`, `deployment/k8s/secrets/`, `deployment/DEPLOYMENT_GUIDE.md`, `docs/updated_kubernetes.md`, and `workflows/kubernetes/`.

### HPA

- [**Kubernetes HPA Horizontal Pod Autoscaler demo**](https://www.youtube.com/watch?v=EefCLPE8K50) — Rafael Benevides / DevX, 07:15. **Community; conceptual demo; old-version (2019); HPA/API-version-sensitive.** Use it to understand the HPA control loop and the relationship between resource metrics and replica count. Verify `apiVersion`, metrics-server behavior, resource requests/limits, stabilization, and target utilization against `deployment/k8s/api/hpa.yaml` and the current [Kubernetes HPA documentation](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/).

### EKS and IRSA

- [**AWS EKS Tutorial: Deploy a Kubernetes Cluster in 10 Minutes**](https://www.youtube.com/watch?v=T46DKrCWSA8) — Cloud Guru, 08:32. **Community/vendor-adjacent; hands-on setup; AWS-specific; promotional/affiliate links; credential and version-sensitive.** It is a quick tool-installation and EKS setup walkthrough, not a production design guide. It demonstrates access-key configuration; do not adopt that pattern for this project. Cluster creation and teardown can incur cost and mutate AWS resources.

- [**IRSA in AWS EKS Explained Deeply**](https://www.youtube.com/watch?v=Sj_apaQFIbI) — The DevOps Bucket, 15:31. **Community; conceptual plus hands-on; AWS/EKS-specific; relatively current; provider-specific.** Explains OIDC, service-account-to-IAM-role trust, STS, temporary credentials, and least privilege. Compare it with `deployment/eks/cluster.yaml`, `deployment/eks/namespace.yaml`, `deployment/k8s/secrets/`, `scripts/secrets.sh`, and the deployment guide. The study plan specifically calls out that static Bedrock credentials can take precedence over the intended IRSA path; verify the effective credential chain inside a disposable pod.

### GitHub Actions and CI/CD

- [**GitHub Actions Tutorial - Basic Concepts and CI/CD Pipeline with Docker**](https://www.youtube.com/watch?v=R8_veQiYBjI) — TechWorld with Nana, 32:30. **Community; hands-on; old-version (2020); promotional links; GitHub Actions/Docker-specific.** Covers events, workflows, runners, secrets, Docker builds, and registry pushes. Compare every step with `.github/workflows/ci.yml` and `.github/workflows/cd.yml`; the repository's CI currently runs Ruff/tests/golden evaluation, while CD and AWS deployment behavior have their own assumptions and gaps.

Primary reading: [Kubernetes concepts](https://kubernetes.io/docs/concepts/), [Kubernetes HPA](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/), [Amazon EKS documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html), [EKS IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html), and [GitHub Actions documentation](https://docs.github.com/en/actions).

---

## What the videos cannot replace

Keep these repository-specific investigations in the study plan rather than searching for a video that appears to answer them:

- **Current configuration and topology:** `.env.example`, `src/config.py`, `compose.yml`, `Dockerfile`, `airflow/Dockerfile`, `deployment/`, and the lockfile.
- **Actual contracts:** `src/schemas/`, `src/routers/`, `src/models/`, `src/repositories/`, and the `ArxivPaper → ParsedPaper → database → TextChunk → OpenSearch hit → SearchHit → response` handoff.
- **Exact ingestion behavior:** arXiv query/rate-limit logic, parser fallbacks, page limits, cleanup, Airflow scheduling, and side effects.
- **Exact retrieval behavior:** chunk formatting, embedding model/dimensions, BM25/K-NN query construction, OpenSearch 2.19.5 assumptions, RRF pipeline behavior, deduplication, and source metadata.
- **Provider parity:** OpenAI/Ollama/Bedrock model factories, streaming event shapes, structured output, health checks, guardrails, and failure handling.
- **Graph reliability:** current LangGraph state, guardrail fail-open paths, rewrite limits, grade semantics, tool routing, and output-source extraction.
- **Evaluation validity:** the current golden test is not a complete factuality or retrieval benchmark; add labeled queries and separate retrieval, groundedness, citation, correctness, and helpfulness measures.
- **Deployment safety:** public LoadBalancers, disabled OpenSearch security assumptions, static credentials versus IRSA, HPA prerequisites, Airflow image/mount behavior, destructive teardown, and real-service credentials.

When a video suggests a command, first ask: does it match this repository's Python `>=3.12,<3.13`, OpenSearch 2.19.5, Airflow 2.10.3 image, current dependency lock, AWS region/IAM design, and test contract? If not, keep the concept and discard the command.

## Suggested video-backed deliverables

Use these as small checkpoints alongside the 14-session schedule:

1. **Runtime map:** draw the FastAPI, service-client, database, cache, and observability boundaries after the Python/async videos.
2. **Contract test:** add or improve one API test after the FastAPI/Pydantic/pytest videos, using the repository's fixtures and response schemas.
3. **Ingestion failure matrix:** run parser fixtures and document arXiv/PDF/Docling failures; do not download a large corpus.
4. **Retrieval experiment:** compare BM25, K-NN, and RRF on a labeled query subset; report recall@k, MRR/nDCG, latency, duplicates, and source correctness.
5. **Trace-and-cache exercise:** capture one cache miss and hit, then inspect nested retrieval/generation spans without leaking secrets or full sensitive prompts.
6. **Graph transition matrix:** test allowed, blocked, irrelevant, rewritten, max-attempt, tool, and guardrail-error paths against the actual state graph.
7. **Provider parity note:** compare one OpenAI and one Bedrock case, including Converse request shape, streaming, model IDs, guardrails, and failure behavior.
8. **Interface comparison:** document which work belongs to HTTP, Telegram, Gradio, MCP, and A2A, and which contracts are project-specific.
9. **Disposable deployment review:** validate manifests and credential flow without running destructive EKS teardown or exposing real secrets.
10. **CI evaluation review:** trace `.github/workflows/ci.yml` from checkout to test result and explain what the golden test does and does not prove.

The goal is not to finish a playlist. The goal is to be able to explain, from this repository's executable behavior, where a query is validated, retrieved, fused, generated, guarded, cached, traced, evaluated, and deployed.
