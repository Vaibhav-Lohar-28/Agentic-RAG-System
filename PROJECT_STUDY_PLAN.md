# arXiv Paper Curator — Project-Specific Learning and Research Plan

**Repository reviewed:** `Vaibhav-Lohar-28/Agentic-RAG-System`
**Review baseline:** commit `94daa1a758aa1a2694d6abb036cbe0961efa53d6`
**Purpose:** learn the system from executable behavior first, then use the notebooks, phase guides, diagrams, and external documentation to understand the design choices and to test where the documentation has drifted.
**Supplementary videos:** [`YOUTUBE_LEARNING_PATH.md`](YOUTUBE_LEARNING_PATH.md) maps curated YouTube explanations to the study plan; videos are learning aids, not a source of truth for this repository.

This is a study plan, not a claim that every historical phase document describes the current runtime. The source of truth is the code and deployment configuration described in the next section.

---

## 1. How to study this repository

### Source-of-truth order

When two files disagree, inspect them in this order:

1. **Executable Python and shell:** `src/`, `airflow/`, `scripts/`, `entrypoint.sh`.
2. **Deployment manifests and workflows:** `compose.yml`, `Dockerfile`, `airflow/Dockerfile`, `deployment/`, `.github/workflows/`.
3. **Schemas and tests:** `src/schemas/`, `tests/`, `data/golden_dataset.json`.
4. **Phase notebooks and READMEs:** `notebooks/phase1` through `notebooks/phase7`.
5. **Architecture/workflow prose and diagrams:** `workflows/`, `docs/`, `README.md`, `step-by-step.md`, `AWS Integration.md`, `static/`.

The phase material is useful for concepts and intended evolution, but several documents predate the Bedrock, supervisor, A2A, MCP, and current LangGraph implementation.

### Complete study index

Use these files as a fixed checklist so a phase folder is not accidentally skipped:

- **Notebook ladder:** `notebooks/phase1/phase1_setup.ipynb`, `notebooks/phase2/phase2_arxiv_integration.ipynb`, `notebooks/phase3/phase3_opensearch.ipynb`, `notebooks/phase4/phase4_hybrid_search.ipynb`, `notebooks/phase5/phase5_complete_rag_system.ipynb`, `notebooks/phase6/phase6_cache_testing.ipynb`, and `notebooks/phase7/phase7_agentic_rag.ipynb`; read each phase's `README.md` first.
- **Phase workflows:** `workflows/overview.md`, `workflows/phase1_infrastructure.md`, `workflows/phase2_data_ingestion.md`, `workflows/phase3_keyword_search.md`, `workflows/phase4_hybrid_search.md`, `workflows/phase5_rag_pipeline.md`, `workflows/phase6_monitoring_caching.md`, and `workflows/phase7_agentic_rag.md`.
- **Kubernetes teaching workflows:** `workflows/kubernetes/README.md`, `01-eks-cluster-architecture.md`, `02-pod-layout.md`, `03-services-and-ingress.md`, `04-hpa-scaling.md`, `05-cicd-pipeline.md`, `06-monitoring.md`, and `07-request-flow.md`.
- **Diagram source files:** `static/phase1_infra_setup.md`, `phase2_data_ingestion_flow.md`, `phase3_opensearch_flow.md`, `phase4_hybrid_opensearch.md`, `phase5_complete_rag.md`, `phase6_monitoring_and_caching.md`, `phase7_telegram_and_agentic_ai.md`, `static/agentic_rag_architecture.md`, and `static/langgraph-mermaid.md`; the adjacent PNG/GIF files are rendered visualizations, not a more authoritative source.
- **Non-Python support:** `requirements-airflow.txt`, `airflow/requirements-airflow.txt`, `opensearch_dashboards/opensearch_dashboards.yml`, `bedrock-policy.json`, `enhancements.txt`, `locustfile.py`, `.pre-commit-config.yaml`, and the two secret-generation scripts (`secrets.sh`, `scripts/secrets.sh`).
- **Verification inventory:** `tests/conftest.py`, `tests/api/conftest.py`, `tests/unit/services/agents/conftest.py`, the unit/API/integration/evaluation files listed in section 16, and `data/golden_dataset.json`.

### Current runtime shape

The repository is best understood as four connected systems:

```text
1. API runtime
   FastAPI -> dependency/app state -> OpenSearch/Postgres/Jina/LLM/Redis
              -> standard RAG, streaming RAG, agentic RAG, supervisor, A2A, MCP

2. Ingestion runtime
   Airflow DAG -> arXiv metadata -> PDF download/cache -> Docling -> Postgres
              -> TextChunker -> Jina embeddings -> OpenSearch chunk index

3. Delivery/runtime infrastructure
   Docker Compose locally; EKS/Kubernetes, ECR, IRSA, HPA, ELBs and Grafana Cloud in deployment material

4. Quality/feedback loop
   pytest mocks and contract tests -> golden structure gate -> Langfuse traces/scores/datasets
```

### Environment limitation from this review

The repository declares Python `>=3.12,<3.13`. In this environment, `uv`, project dependencies, Docker, Ruff, mypy, shellcheck, and Kubernetes tooling were unavailable, so runtime tests were not executed. Python 3.11 `compileall` rejects the backslash-containing f-string expression in `src/services/indexing/text_chunker.py`; recheck under Python 3.12 before treating that syntax as a defect. A later environment validation should be part of the plan, not skipped.

---

## 2. First session: orient yourself in the repository

### Read first

- `README.md` — historical seven-phase narrative and quick start.
- `step-by-step.md` — local setup sequence.
- `pyproject.toml` — Python version, runtime/dev dependencies, Ruff, pytest and mypy settings.
- `uv.lock` — exact resolved dependency graph.
- `.env.example` and `.env.test` — configuration names and test defaults.
- `Makefile` — intended developer commands, while checking each target against the current code.
- `Dockerfile`, `compose.yml` — actual local container graph.

### Learn

- Python package layout and import side effects.
- `pyproject.toml` versus a lockfile and why CI uses `uv sync --frozen`.
- Runtime versus development dependencies.
- Async Python basics: coroutine, `await`, async generator, `asyncio.run`, `asyncio.to_thread`.
- Why a function declared `async` can still block the event loop when it calls a synchronous client.
- Container build context, multi-stage builds, bind mounts, named volumes, networks, health checks, and `depends_on`.

### Commands

```bash
python3 --version
uv --version
uv sync --frozen --dev
uv run pytest -x --tb=short -v
uv run ruff check --no-fix src/ tests/
uv run ruff format --check src/ tests/
uv run mypy src/

docker compose config
docker compose up --build -d
docker compose ps
docker compose logs -f
curl http://localhost:8000/api/v1/health
```

### Exercise

Draw the four Compose services and mark which dependencies are local versus managed cloud services:

- API: `8000`
- OpenSearch: `9200`, performance/plugin port `9600`
- OpenSearch Dashboards: `5601`
- Airflow: `8080`
- PostgreSQL, Redis, Langfuse, OpenAI/Jina: normally external in the documented cloud setup

Note that there is **no Ollama service in `compose.yml`** and no Gradio service; Gradio is a separate process started with `python gradio_launcher.py`.

---

## 3. Configuration and application bootstrap

### Read in this order

1. `src/config.py`
2. `src/dependencies.py`
3. `src/main.py`
4. All factories under `src/services/*/factory.py`
5. `src/routers/ping.py`
6. `.env.example`, `.env.test`, `compose.yml`, and the API/Kubernetes secret templates

### Concepts to learn

- Pydantic v2 `BaseSettings` and `pydantic-settings`.
- `env_nested_delimiter="__"`, prefixes such as `REDIS__` and `BEDROCK__`, case-insensitive environment names, `SecretStr`, validators, and frozen settings.
- Dependency injection with `Depends`, `Annotated`, request state, and generator dependencies.
- FastAPI lifespan as an async context manager: code before `yield` is startup; code after `yield` is cleanup.
- Singleton/caching strategies: `lru_cache`, `cached_property`, application state, and why multiple factories can still create multiple settings objects.
- Uvicorn workers and process-local state.
- Health/readiness/liveness semantics.

### Actual startup order to trace

Follow `src/main.py:41-194` and make a numbered sequence:

1. Create the FastMCP HTTP sub-application.
2. Enter the composed MCP lifespan.
3. Load settings and assign `app.state.settings`.
4. Configure Logfire and FastAPI instrumentation.
5. Create/start PostgreSQL.
6. Create OpenSearch, check health, create the hybrid index and RRF search pipeline, and inspect document count.
7. Create arXiv client, Docling parser, Jina embeddings client.
8. Select `OpenAILLMClient` or `BedrockLLMClient` from `PROVIDER`.
9. Always construct `BedrockGuardrailsService`; it is effectively disabled when no guardrail ID is configured.
10. Create Langfuse tracer and Redis cache client.
11. Build one shared `AgenticRAGService`.
12. Build `SupervisorAgent` around that shared service.
13. Wire the MCP global context.
14. Acquire `/tmp/telegram_bot.lock`; only the worker that acquires it may start Telegram polling.
15. Yield to serve requests.
16. On shutdown, stop Telegram, release the lock, and dispose PostgreSQL.

### Important configuration findings to study rather than assume

- `src/config.py:get_settings()` is not cached, while `src/dependencies.py:get_settings()` has its own `lru_cache`. Factories may therefore load independent settings instances.
- Active provider choices are OpenAI and Bedrock. `src/services/ollama/` is legacy and not selected by `src/main.py`.
- `src/routers/ping.py` creates and checks **OpenAI regardless of `PROVIDER`**. A Bedrock deployment can therefore report degraded health or fail a readiness probe because the wrong provider is checked.
- `src/middlewares.py` contains helper logging functions but is not registered as FastAPI middleware.
- `src/database.py` is an older global database helper; the active API path uses `src/db/` and `app.state`.
- Startup creates database tables with `Base.metadata.create_all`; there is an Alembic dependency but no active migration workflow was found.

### Environment-variable matrix

`Settings` uses top-level names exactly as shown below and nested settings with the configured prefixes. Blank secret values are valid defaults in code, but the corresponding service may fail later. `extra="ignore"` means a typo can be silently ignored.

| Python setting | Environment variable | Code default / purpose |
|---|---|---|
| `app_version`, `debug`, `environment`, `service_name` | `APP_VERSION`, `DEBUG`, `ENVIRONMENT`, `SERVICE_NAME` | `0.1.0`, `true`, `development`, `rag-api`; environment is limited to development/staging/production |
| `postgres_database_url`, `postgres_echo_sql`, `postgres_pool_size`, `postgres_max_overflow` | `POSTGRES_DATABASE_URL`, `POSTGRES_ECHO_SQL`, `POSTGRES_POOL_SIZE`, `POSTGRES_MAX_OVERFLOW` | local PostgreSQL URL, `false`, `5`, `0` |
| `openai_api_key`, `openai_model`, `openai_timeout` | `OPENAI_API_KEY`, `OPENAI_MODEL`, `OPENAI_TIMEOUT` | blank, `gpt-4o-mini`, `300` |
| `provider`, `jina_api_key` | `PROVIDER`, `JINA_API_KEY` | `openai`, blank; provider code selects OpenAI or Bedrock |
| arXiv URL/cache/query | `ARXIV__BASE_URL`, `ARXIV__PDF_CACHE_DIR`, `ARXIV__SEARCH_CATEGORY`, `ARXIV__MAX_RESULTS` | export API, `./data/arxiv_pdfs`, `cs.AI`, `15` |
| arXiv timing/retries | `ARXIV__RATE_LIMIT_DELAY`, `ARXIV__TIMEOUT_SECONDS`, `ARXIV__DOWNLOAD_MAX_RETRIES`, `ARXIV__DOWNLOAD_RETRY_DELAY_BASE` | `3.0`, `60`, `3`, `5.0` |
| arXiv concurrency | `ARXIV__MAX_CONCURRENT_DOWNLOADS`, `ARXIV__MAX_CONCURRENT_PARSING` | `5`, `1` |
| parser | `PDF_PARSER__MAX_PAGES`, `PDF_PARSER__MAX_FILE_SIZE_MB`, `PDF_PARSER__DO_OCR`, `PDF_PARSER__DO_TABLE_STRUCTURE` | `30`, `20`, `false`, `true` |
| chunking | `CHUNKING__CHUNK_SIZE`, `CHUNKING__OVERLAP_SIZE`, `CHUNKING__MIN_CHUNK_SIZE`, `CHUNKING__SECTION_BASED` | `600`, `100`, `100`, `true` |
| OpenSearch | `OPENSEARCH__HOST`, `OPENSEARCH__INDEX_NAME`, `OPENSEARCH__CHUNK_INDEX_SUFFIX`, `OPENSEARCH__MAX_TEXT_SIZE` | `http://localhost:9200`, `arxiv-papers`, `chunks`, `1000000` |
| vector/hybrid search | `OPENSEARCH__VECTOR_DIMENSION`, `OPENSEARCH__VECTOR_SPACE_TYPE`, `OPENSEARCH__RRF_PIPELINE_NAME`, `OPENSEARCH__HYBRID_SEARCH_SIZE_MULTIPLIER` | `1024`, `cosinesimil`, `hybrid-rrf-pipeline`, `2` |
| Redis | `REDIS__URL`, `REDIS__TTL_HOURS` | `redis://localhost:6379`, `6` hours |
| Langfuse | `LANGFUSE__PUBLIC_KEY`, `LANGFUSE__SECRET_KEY`, `LANGFUSE__HOST`, `LANGFUSE__ENABLED`, `LANGFUSE__FLUSH_AT`, `LANGFUSE__FLUSH_INTERVAL`, `LANGFUSE__MAX_RETRIES`, `LANGFUSE__TIMEOUT`, `LANGFUSE__DEBUG` | cloud US host; enabled, flush 15/1 second, retries 3, timeout 30, debug false |
| Bedrock | `BEDROCK__AWS_ACCESS_KEY_ID`, `BEDROCK__AWS_SECRET_ACCESS_KEY`, `BEDROCK__AWS_REGION`, `BEDROCK__MODEL_ID` | blank credentials, `us-east-1`, `meta.llama3-1-70b-instruct-v1:0` |
| guardrails | `BEDROCK__GUARDRAIL_ID`, `BEDROCK__GUARDRAIL_VERSION` | blank/disabled, `DRAFT` |
| Telegram/MCP | `TELEGRAM__BOT_TOKEN`, `TELEGRAM__ENABLED`, `MCP__ENABLED`, `MCP__PATH` | blank/false, true, `/mcp` |
| Logfire | `LOGFIRE__ENABLED`, `LOGFIRE__TOKEN`, `LOGFIRE__SERVICE_NAME`, `LOGFIRE__ENVIRONMENT`, `LOGFIRE__SEND_TO_LOGFIRE` | true, blank, `arxiv-rag`, `development`, `if-token-present` |

The `.env.example` is not identical to code defaults: it sets `ARXIV__TIMEOUT_SECONDS=30`, while `src/config.py` defaults to 60. Kubernetes/CD overrides additional values, including `ARXIV__MAX_RESULTS=2`, timeout 120, rate delay 5.0, parser max pages 60, and provider `bedrock`. Airflow variables such as `AIRFLOW__CORE__EXECUTOR`, `AIRFLOW__DATABASE__SQL_ALCHEMY_CONN`, `AIRFLOW__WEBSERVER__SECRET_KEY`, `AIRFLOW__API__AUTH_BACKENDS`, `AIRFLOW__CORE__DAG_IGNORE_FILE_SYNTAX`, `AIRFLOW__WEBSERVER__EXPOSE_CONFIG`, `AIRFLOW__HOME`, `AIRFLOW__CORE__LOAD_EXAMPLES`, and `PYTHONWARNINGS` configure Airflow itself, not `src.config.Settings`. Deployment scripts additionally consume `CLUSTER_NAME`, `AWS_REGION`, `AWS_ACCOUNT_ID`, `K8S_NAMESPACE`, `GRAFANA_ENABLED`, `GRAFANA_CLOUD_TOKEN`, and GitHub secret values.

### Exercises

- Instantiate `Settings` with `.env.test`, then override one nested variable from the shell and verify precedence.
- Use `app.state` versus a `Depends` function in a small FastAPI test.
- Run the application with four workers and explain why the Telegram file lock is needed.
- Replace the health check in a design diagram with a provider-neutral check and list which clients need explicit shutdown.

---

## 4. API contracts and request flow

### Read

- `src/routers/ping.py`
- `src/routers/hybrid_search.py`
- `src/routers/ask.py`
- `src/routers/agentic_ask.py`
- `src/routers/supervisor_ask.py`
- `src/routers/a2a.py`
- `src/schemas/api/health.py`
- `src/schemas/api/search.py`
- `src/schemas/api/ask.py`
- `src/dependencies.py`
- `tests/api/routers/`

### Active HTTP surface

| Method and path | Purpose | Main request/response files |
|---|---|---|
| `GET /api/v1/health` | Database, OpenSearch and currently OpenAI health checks | `ping.py`, `health.py` |
| `POST /api/v1/hybrid-search/` | BM25 or hybrid chunk search | `hybrid_search.py`, `search.py` |
| `POST /api/v1/ask` | Standard cached RAG response | `ask.py`, `ask.py` schema |
| `POST /api/v1/stream` | Streaming standard RAG | `ask.py` |
| `POST /api/v1/ask-agentic` | LangGraph agentic RAG | `agentic_ask.py`, `ask.py` schema |
| `POST /api/v1/feedback` | Langfuse score submission | `agentic_ask.py`, `ask.py` schema |
| `POST /api/v1/ask-supervisor` | Intent classification and RAG/summarizer routing | `supervisor_ask.py` |
| `GET /.well-known/agent.json` | A2A agent card | `a2a.py`, `services/a2a/models.py` |
| `POST /a2a/tasks/send` | Minimal synchronous A2A task | `a2a.py` |
| MCP mounted at configured `MCP__PATH` (default `/mcp`) | MCP tools and resources | `src/mcp_server/` |

### Contract details to memorize

`AskRequest` in `src/schemas/api/ask.py`:

- `query`: 1–1000 characters.
- `top_k`: 1–10, default 3.
- `use_hybrid`: default `true`.
- `model`: optional provider-specific model ID.
- `categories`: optional list.

`HybridSearchRequest` in `src/schemas/api/search.py`:

- `query`: 1–500 characters.
- `size`: 1–100, default 10.
- `from` is the JSON alias for Python `from_`.
- `categories`, `latest_papers`, `use_hybrid`, and `min_score`.

`AskResponse` contains `query`, `answer`, `sources` as URL strings, `chunks_used`, and `search_mode`. `AgenticAskResponse` overrides sources with rich objects and adds reasoning, attempts, rewritten query, trace ID, and guardrail fields.

### Exercises

```bash
curl -s http://localhost:8000/openapi.json | jq '.paths | keys'

curl -X POST http://localhost:8000/api/v1/hybrid-search/ \
  -H 'Content-Type: application/json' \
  -d '{"query":"transformer attention","size":5,"use_hybrid":false}'

curl -X POST http://localhost:8000/api/v1/ask \
  -H 'Content-Type: application/json' \
  -d '{"query":"What are transformer architectures?","top_k":3,"use_hybrid":true}'

curl -X POST http://localhost:8000/api/v1/ask-agentic \
  -H 'Content-Type: application/json' \
  -d '{"query":"What is attention in deep learning?"}'
```

For every request, record validation behavior, status code, response fields, actual number of hits, and whether the returned `search_mode` describes actual work or only the request flag.

---

## 5. PostgreSQL, SQLAlchemy, schemas, and repository behavior

### Read

- `src/db/interfaces/base.py`
- `src/db/interfaces/postgresql.py`
- `src/db/factory.py`
- `src/models/paper.py`
- `src/repositories/paper.py`
- `src/schemas/arxiv/paper.py`
- `src/schemas/pdf_parser/models.py`
- `src/mcp_server/tools/papers.py`
- `src/mcp_server/resources/papers.py`
- `tests/unit/services/test_metadata_fetcher.py`
- `tests/integration/test_services.py`

### Concepts to learn

- SQLAlchemy engine, connection pooling, `pool_pre_ping`, sessions, transactions, `expire_on_commit=False`, and context-manager rollback behavior.
- Declarative models and `Base.metadata.create_all`.
- PostgreSQL UUID, JSON, text/date columns, unique indexes, and denormalized content storage.
- Repository pattern and upsert-by-`arxiv_id`.
- Neon PostgreSQL, SSL mode, DNS/IPv4 versus IPv6, and why `host` and `hostaddr` are handled in `_force_ipv4_connect_arg`.
- Differences between a Pydantic input schema, a SQLAlchemy model, and a serialized API/MCP object.

### Actual stored paper model

`src/models/paper.py` stores core arXiv metadata, raw text, JSON sections/references, parser metadata, processing flags, and timestamps. `PaperRepository.upsert` updates every `exclude_unset` field when the arXiv ID exists and otherwise creates a row.

### Data-contract handoff

| Boundary | Contract | Important shape |
|---|---|---|
| arXiv client -> metadata pipeline | `ArxivPaper` | string `published_date`, author list, categories and PDF URL |
| parser -> metadata pipeline | `PdfContent` | `PaperSection` objects, raw text, figures/tables, string references, parser enum and metadata |
| combined ingestion result | `ParsedPaper` | `arxiv_metadata` plus optional `pdf_content` |
| metadata pipeline -> DB repository | `PaperCreate`/dictionary | `published_date` converted to `datetime`; parsed fields serialized to JSON-compatible dicts |
| DB -> indexer | `paper_data` dictionary | `id`, arXiv metadata, `raw_text` or `full_text`, optional `sections`; indexer creates one or more `TextChunk`s |
| chunker -> indexer | `TextChunk` | text, `ChunkMetadata`, `arxiv_id`, `paper_id` |
| indexer -> OpenSearch | `chunk_data` plus `embedding` | denormalized title/authors/abstract/categories/date, chunk offsets/count, section title and Jina vector |
| OpenSearch -> API | hit dictionary -> `SearchHit` | `_id` becomes `chunk_id`; scores/highlights are added; indexed `chunk_data` does **not** include `pdf_url`, so `SearchHit.pdf_url` can be null |
| standard generation -> API | `AskResponse` | answer plus PDF URL strings and search metadata |
| graph -> API | `AgenticAskResponse` | rich source objects, reasoning, attempts, rewrite, trace and guardrail fields |

The parser's `PdfContent.references` is `List[str]`, while the database-facing `PaperCreate.references` is `Optional[List[Dict[str, Any]]]`; current Docling output is empty, so this mismatch can be hidden until non-empty references are parsed. Also, `bulk_index_chunks` does not supply a deterministic document ID, so indexing the same paper twice without `replace_existing=True` can leave duplicate chunks.

### Risks/exercises

- Verify the import path that registers `Paper` before `Base.metadata.create_all`; current imports reach the model indirectly through MCP modules.
- Compare `PdfContent.references: List[str]` with `PaperCreate.references: Optional[List[Dict[str, Any]]]` and explain the schema drift.
- Decide how a real migration would add a column without relying on startup table creation.
- Run a transaction exercise: insert two papers, force one failure, and verify rollback scope.
- Compare synchronous SQLAlchemy calls from async MCP/router functions with an async SQLAlchemy design.

---

## 6. arXiv ingestion and PDF processing

### Read

- `src/services/arxiv/client.py`
- `src/services/arxiv/factory.py`
- `src/services/metadata_fetcher.py`
- `src/services/pdf_parser/parser.py`
- `src/services/pdf_parser/docling.py`
- `src/services/pdf_parser/factory.py`
- `src/schemas/arxiv/paper.py`
- `src/schemas/pdf_parser/models.py`
- `airflow/dags/arxiv_paper_ingestion.py`
- `airflow/dags/arxiv_ingestion/common.py`
- `airflow/dags/arxiv_ingestion/setup.py`
- `airflow/dags/arxiv_ingestion/fetching.py`
- `airflow/dags/arxiv_ingestion/indexing.py`
- `airflow/dags/arxiv_ingestion/reporting.py`
- `tests/unit/services/test_arxiv_client.py`
- `tests/unit/services/test_pdf_parser.py`
- `tests/unit/services/test_metadata_fetcher.py`
- `notebooks/phase2/phase2_arxiv_integration.ipynb`
- `notebooks/phase2/README.md`
- `workflows/phase2_data_ingestion.md`
- `airflow/README.md`

### Concepts to learn

- arXiv Atom XML namespaces and query syntax: `cat:`, `ti:`, date ranges, sort order and pagination.
- URL encoding and why `safe=":+[]*"` is used.
- Rate limiting, retryable HTTP statuses (429/503), exponential backoff, timeout classes, and partial failure handling.
- PDF cache naming by arXiv ID, streaming downloads, file validation, size/page limits, and parser failure fallback.
- Docling document conversion, text export, section-header detection, OCR/table options, and the difference between sync CPU-heavy parsing and async orchestration.
- Metadata-only versus parsed-content ingestion.
- Airflow XCom as task-to-task metadata rather than a bulk data store.

### Actual Python ingestion path

`MetadataFetcher` fetches metadata, runs bounded concurrent download/parse pipelines, serializes parsed content, and upserts papers into PostgreSQL. Parsing failure is allowed to continue as metadata-only storage. The active Docling parser returns sections, raw text, empty figures/tables/references, parser type, and metadata.

### Actual Airflow path

The DAG in `airflow/dags/arxiv_paper_ingestion.py` is:

```text
setup_environment -> fetch_daily_papers -> index_papers_hybrid
                    -> generate_daily_report -> cleanup_temp_files
```

The DAG is scheduled as `0 6 * * 1-5`, but `fetching.py` deliberately replaces the date-based behavior with:

```text
cat:<configured category> AND ti:transformer
max_results=2
sort_by=relevance
```

`target_date` is logged but ignored for the actual query. `verify_hybrid_index` exists but is imported and not placed in the active task dependency chain.

### Exercises

```bash
uv run python scripts/test_connections.py
uv run python scripts/insert_papers_by_id.py 1706.03762 --skip-pdf
uv run python scripts/insert_papers_by_id.py 1706.03762 --replace-existing
```

Use a mocked XML response to test a date query and a custom `ti:` query. Use a small valid PDF, empty file, non-PDF file, oversized file, and too-many-pages file to map the parser exceptions.

### Defects/drift to verify

- `TextChunker` has a separate short-text bug described below.
- `DoclingParser.parse_pdf` docstring says a 20-page limit while configuration defaults to 30 and the Kubernetes secret template sets 60.
- `airflow/README.md` describes Airflow 3.0, 10-paper/date-based ingestion, and OpenSearch placeholders; the Docker image installs Airflow 2.10.3 and the current DAG fully invokes chunking/embedding/indexing.
- The cleanup task searches `/tmp` while the configured arXiv cache defaults to `./data/arxiv_pdfs`; determine whether the cleanup actually removes the cache used by the task.

---

## 7. Chunking, embeddings, indexing, and OpenSearch retrieval

### Read

- `src/services/indexing/text_chunker.py`
- `src/services/indexing/hybrid_indexer.py`
- `src/services/indexing/factory.py`
- `src/schemas/indexing/models.py`
- `src/services/embeddings/jina_client.py`
- `src/services/embeddings/factory.py`
- `src/schemas/embeddings/jina.py`
- `src/services/opensearch/index_config_hybrid.py`
- `src/services/opensearch/query_builder.py`
- `src/services/opensearch/client.py`
- `src/services/opensearch/factory.py`
- `tests/unit/services/test_opensearch_query_builder.py`
- `notebooks/phase3/phase3_opensearch.ipynb`
- `notebooks/phase4/phase4_hybrid_search.ipynb`
- `notebooks/phase3/README.md`
- `notebooks/phase4/README.md`
- `workflows/phase3_keyword_search.md`
- `workflows/phase4_hybrid_search.md`
- `static/phase3_opensearch_flow.md`
- `static/phase4_hybrid_opensearch.md`

### Concepts to learn

- Word-based chunking, overlap, minimum chunk sizes, section-aware splitting, headers, and character offsets.
- Embeddings as fixed-dimensional vectors; retrieval query versus passage embedding tasks.
- Jina `jina-embeddings-v3`, 1024 dimensions, batching, HTTP error handling, and lifecycle of a reusable `httpx.AsyncClient`.
- OpenSearch analyzers, tokenization, stop words, stemming/snowball, BM25 multi-match fields, fuzziness, field boosts, highlighting, filters, and sort order.
- `knn_vector`, HNSW, `ef_construction`, `m`, cosine similarity, approximate nearest neighbor trade-offs.
- Hybrid query execution and rank-based Reciprocal Rank Fusion. The configured formula uses rank constant 60; scores are rank-derived and much smaller than typical BM25 scores.
- `dynamic: strict`, denormalized chunk metadata, `_id` versus source `chunk_id`, bulk indexing and delete-by-query.

### Actual index contract

`src/services/opensearch/index_config_hybrid.py` creates one index named from `{OPENSEARCH__INDEX_NAME}-{OPENSEARCH__CHUNK_INDEX_SUFFIX}`, normally `arxiv-papers-chunks`. It supports `chunk_text`, `title`, `authors`, `abstract`, categories, publication date, section title, chunk offsets/count, and a 1024-dimensional `embedding` field.

BM25 chunk search uses:

```text
chunk_text^3, title^2, abstract^1
```

Hybrid search requests `size * 2` results from both BM25 and KNN, then returns `size` fused results through `hybrid-rrf-pipeline`.

### Important chunker defects

Study and write tests for these before changing them:

1. In `chunk_text`, the short-text path calls `_reconstruct_text(words, text)` even though `_reconstruct_text` accepts only `words`; a paper below `min_chunk_size` will fail instead of returning its single chunk.
2. The Docling synthetic section title is `Content`, and `_is_metadata_section` filters titles containing `content`, so the first useful content can disappear.
3. Section chunks repeat title and abstract headers, increasing index size and prompt duplication.
4. Section-based offsets are generally set to zero; word-based offsets are approximate because whitespace is reconstructed with single spaces.
5. The section strategy uses hard-coded 100/800-word thresholds rather than all configured chunk settings.
6. The `references`/section schema types are not fully consistent across parser, database, and indexing layers.

### OpenSearch risks to investigate

- OpenSearch calls are synchronous inside async FastAPI, MCP, Telegram, and LangGraph tool paths.
- `_create_rrf_pipeline` checks/deletes an **ingest** pipeline but creates a **search** pipeline using `/_search/pipeline`; startup may therefore recreate or report the pipeline incorrectly every time.
- `OPENSEARCH__HYBRID_SEARCH_SIZE_MULTIPLIER` is defined but `_search_hybrid_native` hard-codes `size * 2`.
- Native hybrid search ignores `from_` and `latest` because `_search_hybrid_native` receives neither; pagination/latest behavior is only implemented in BM25 mode.
- The API lets callers set `min_score`; with RRF rank scores around `1/(60+rank)`, a tutorial value such as `0.5` can remove every result.
- The client disables SSL/certificate verification unconditionally; this is appropriate only for the security-disabled local setup, not a hardened production cluster.

### Exercises

```bash
curl http://localhost:9200/_cluster/health
curl http://localhost:9200/_cat/indices?v
curl http://localhost:9200/arxiv-papers-chunks/_mapping
curl -X POST http://localhost:8000/api/v1/hybrid-search/ \
  -H 'Content-Type: application/json' \
  -d '{"query":"semantic retrieval","size":5,"use_hybrid":true,"min_score":0}'
```

Create a table comparing BM25, pure KNN, and RRF results for exact-keyword, paraphrased, and category-filtered queries. Record recall, latency, score range, and duplicate papers/chunks.

---

## 8. Standard RAG and streaming RAG

### Read

- `src/routers/ask.py`
- `src/services/ollama/prompts.py`
- `src/services/ollama/prompts/rag_system.txt`
- `src/services/openai_llm/client.py`
- `src/services/bedrock_llm/client.py`
- `src/services/llm_client_protocol.py`
- `src/schemas/ollama.py`
- `tests/api/routers/test_ask.py`
- `notebooks/phase5/phase5_complete_rag_system.ipynb`
- `notebooks/phase5/README.md`
- `workflows/phase5_rag_pipeline.md`
- `notebooks/phase6/phase6_cache_testing.ipynb`
- `workflows/phase6_monitoring_caching.md`

### Standard `/ask` execution

Trace `src/routers/ask.py:79-195`:

1. Read `X-User-Id` and form a session ID.
2. Enter `RAGTracer.trace_request`.
3. Try exact Redis cache.
4. If hybrid is requested, call Jina `embed_query`; on failure continue with BM25.
5. Call synchronous `OpenSearchClient.search_unified`.
6. Reduce each hit to `arxiv_id` and chunk text; derive PDF source URLs.
7. Build a prompt using the shared builder in the legacy `ollama` package.
8. Call the selected direct provider's `generate_rag_answer`.
9. Return answer, sources, chunk count and mode; score/save telemetry; cache the response.

Learn the RAG boundary carefully: retrieval supplies evidence; the prompt instructs the model to answer only from evidence; the provider adapter supplies generation; response assembly is not the same thing as factual evaluation.

### Streaming `/stream` execution

The route returns an async generator that emits lines shaped like:

```text
data: {"sources": [...], "chunks_used": 3, "search_mode": "hybrid"}\n\n
data: {"chunk": "partial text"}\n\n
data: {"answer": "full answer", "done": true}\n\n
```

Learn SSE framing, async generators, back-pressure, cancellation, client disconnects, and the difference between `text/plain` and `text/event-stream`.

### Provider study

**OpenAI:** `OpenAILLMClient` uses the direct async Chat Completions path for standard RAG and `ChatOpenAI` for LangGraph nodes. It builds source URLs from chunk IDs and labels confidence `high` without an independent confidence model.

**Bedrock:** `BedrockLLMClient` uses boto3 `converse`/`converse_stream` through `asyncio.to_thread` for direct RAG and `ChatBedrock` for LangGraph nodes. It infers a provider from model IDs/ARNs and translates usage fields.

**Ollama:** `src/services/ollama/client.py` and `src/services/ollama/factory.py` are historical code. They refer to missing `Settings.ollama_host`/`ollama_timeout`, import undeclared `langchain_ollama`, and are not active in `main.py` or `pyproject.toml`.

### RAG risks

- The standard route labels `search_mode` from `request.use_hybrid`, even when embedding generation failed and actual search was BM25.
- The stream route declares `media_type="text/plain"`, not `text/event-stream`.
- Cached streaming splits the cached answer on whitespace, so its chunk boundaries do not match provider streaming.
- Standard `/ask` and `/stream` do not run the Bedrock input/output guardrail nodes; those guardrails are on the agentic graph path.
- The direct prompt builder is located under `services/ollama` even though OpenAI and Bedrock are the active providers; treat it as shared legacy naming, not evidence that Ollama is active.
- `src/gradio_app.py` hard-codes `http://localhost:8000/api/v1`, which is correct only when the browser-facing Gradio process and API are reachable on the same machine. It is not correct for a remote preview or EKS deployment.

### Exercises

- Capture a normal response and a cache-hit response, comparing sources, mode, chunk count, and trace behavior.
- Force Jina to fail in a unit test and assert both actual search method and response metadata.
- Write an SSE parser that accepts only `text/event-stream`, then document what must change in the server/client contract.
- Compare OpenAI and Bedrock direct generation output/usage shape using mocked clients.

---

## 9. Redis caching and observability

### Read

- `src/services/cache/client.py`
- `src/services/cache/factory.py`
- `src/services/langfuse/client.py`
- `src/services/langfuse/tracer.py`
- `src/services/langfuse/factory.py`
- `src/services/logfire/factory.py`
- `src/config.py` Langfuse/Redis/Logfire settings
- `workflows/phase6_monitoring_caching.md`
- `notebooks/phase6/phase6_cache_testing.ipynb`

### Concepts

- Exact-match cache keys, canonicalization, SHA-256, collision/truncation trade-offs, TTL, cache-aside pattern, cache hit/miss metrics, invalidation after re-indexing.
- Redis TCP/TLS URLs (`redis://` and Upstash `rediss://`), connection timeout and retry configuration.
- OpenTelemetry instrumentation versus semantic LLM tracing.
- Langfuse traces, spans, generations, callback handlers, scores, feedback and datasets.
- Logfire as infrastructure/Pydantic/FastAPI/HTTPX/SQLAlchemy/Redis telemetry.

### Actual cache key

`CacheClient._generate_cache_key` includes `query`, `model`, `top_k`, `use_hybrid`, and sorted `categories`, serializes with sorted JSON keys, hashes with SHA-256, and retains 16 hex characters. It is exact-match caching, not semantic similarity caching.

Redis operations are synchronous `get`/`set` calls inside async methods. Measure whether high Redis latency blocks the event loop.

### Actual tracing behavior

- `RAGTracer` wraps standard requests and creates embedding/search/prompt/generation spans.
- Agent nodes create Logfire spans and optional Langfuse observations.
- Langfuse feedback is submitted by `POST /api/v1/feedback` using a returned trace ID.
- Standard RAG auto-scores `0.9` whenever chunks exist and `0.2` when none exist; this is a heuristic, not answer correctness.
- Dataset insertion saves query/answer pairs but does not supply a human-verified expected answer.

### Exercises

- Change only whitespace, category order, model omission, and `top_k`; determine which requests share a cache entry.
- Design a cache key that includes the embedding model/version and prompt version.
- Query Langfuse for latency, token use, and user scores, then compare to Logfire HTTP/DB timings.
- Create a quality rubric that separately scores retrieval recall, citation correctness, groundedness, and answer completeness.

---

## 10. LangGraph agentic RAG

### Read

- `src/services/agents/agentic_rag.py`
- `src/services/agents/config.py`
- `src/services/agents/context.py`
- `src/services/agents/state.py`
- `src/services/agents/models.py`
- `src/services/agents/prompts.py`
- `src/services/agents/tools.py`
- Every file in `src/services/agents/nodes/`
- `src/services/agents/factory.py`
- `tests/unit/services/agents/`
- `tests/api/routers/test_agentic_ask.py`
- `notebooks/phase7/phase7_agentic_rag.ipynb`
- `notebooks/phase7/README.md`
- `workflows/phase7_agentic_rag.md`
- `static/agentic_rag_architecture.md`
- `static/langgraph-mermaid.md`

### Actual graph topology

The compiled graph in `agentic_rag.py` is:

```text
START
  -> guardrail
       -> out_of_scope -> END
       -> retrieve
            -> tool_retrieve -> grade_documents
                                      -> generate_answer
                                           -> output_guardrail -> END
                                      -> rewrite_query -> retrieve
            -> END when retrieve emits no tool call (max-attempt fallback)
```

The runtime dependency object is `Context`; the mutable graph data is `AgentState`. `Runtime[Context]` is how nodes receive clients without globals.

### Node-by-node study checklist

1. **`guardrail_node.py`**
   - Reads the latest human message.
   - Calls Bedrock `ApplyGuardrail` when a service is present.
   - Maps allowed to score 100 and blocked to 0 for backward-compatible `GuardrailScoring`.
   - Passes a sanitized output when available.
   - Fails open on missing service or exceptions.
   - Routes using `runtime.context.guardrail_threshold`.

2. **`retrieve_node.py`**
   - Stores `original_query` once.
   - Uses `sanitized_query` or latest human message.
   - Increments attempts and creates an explicit `retrieve_papers` tool call.
   - Emits a fallback AI message after the maximum is reached.

3. **`tools.py` / `ToolNode`**
   - Always calls Jina `embed_query`, even if `use_hybrid=False`.
   - Calls synchronous OpenSearch from an async tool.
   - Converts hits to `langchain_core.documents.Document` and serializes metadata.

4. **`grade_documents_node.py`**
   - Grades the complete concatenated tool context with one LLM call.
   - Uses a text heuristic to find a `binary_score=no` marker.
   - Treats ambiguous output as relevant and fails open on LLM errors.
   - Produces one `GradingResult` for `retrieved_docs`, not one result per chunk.
   - Extracts sources only when the aggregate grade is relevant.

5. **`rewrite_query_node.py`**
   - Calls provider-specific LangChain model with structured `QueryRewriteOutput`.
   - Uses temperature 0.3.
   - Falls back to appending `research paper arxiv machine learning` when structured output fails.
   - Appends a new `HumanMessage`, so latest-query helpers now see the rewritten text.

6. **`generate_answer_node.py`**
   - Uses `GENERATE_ANSWER_PROMPT` and the latest tool context.
   - Calls `get_langchain_model` with runtime model/temperature.
   - Fails to a user-facing error message if generation fails.

7. **`output_guardrail_node.py`**
   - Sends answer, retrieved tool context, and query to Bedrock output grounding/content checks.
   - Replaces a blocked answer with `GROUNDING_FAIL_MESSAGE`.
   - Fails open on guardrail exceptions.
   - Splits serialized tool context on blank lines, which is not necessarily a clean list of source chunks.

8. **`nodes/utils.py`**
   - Extracts source metadata by regex-parsing serialized `ToolMessage` content.
   - Treats separately matched ID/title/source/score/author arrays as aligned; formatting changes can break that assumption.

### Major documentation correction

The phase 7 README and workflow diagrams describe an LLM deciding whether to call retrieval (`generate_query_or_respond`) and list an old `nodes.py`. The current graph has no such node and no active use of `SYSTEM_MESSAGE`, `DECISION_PROMPT`, `DIRECT_RESPONSE_PROMPT`, or `GUARDRAIL_PROMPT`. The guardrail makes the scope decision; in-scope requests deterministically create a retrieval tool call.

### Configuration and API drift

- Production factory config defaults `GraphConfig.guardrail_threshold` to 40; the agent unit fixture explicitly uses 60. `Context` also defaults to 60, but factory-built production context receives the graph's 40. Determine the intended threshold and update tests/docs accordingly.
- `POST /api/v1/ask-agentic` passes only `query` and `model` to `AgenticRAGService.ask`. Request `top_k`, `use_hybrid`, and `categories` do not reach the graph's tool, which was built at startup with factory defaults.
- The response fills `chunks_used` and `search_mode` from the request rather than actual graph work.
- A caller can pass an OpenAI model name while `PROVIDER=bedrock`; the Bedrock adapter may then receive an invalid model ID.
- Retrieval attempts are bounded by a fallback in `retrieve_node`, but grade/rewrite behavior and final routing must be tested at the boundary (`retrieval_attempts == max_retrieval_attempts`).

### Exercises

- Use mocked node outputs to draw a state snapshot after every graph node.
- Test: empty query, out-of-scope query, allowed query with relevant docs, irrelevant docs then rewrite, embedding failure, grading failure, generation failure, output guardrail block, output guardrail API failure.
- Assert `ToolMessage` content shape rather than relying on regex parsing.
- Compare the actual graph Mermaid output with `static/langgraph-mermaid.md` and `workflows/phase7_agentic_rag.md`.
- Add a request-scoped graph configuration design on paper that would correctly propagate `top_k`, hybrid mode, categories and provider/model.

---

## 11. Bedrock LLM and Guardrails

### Read

- `AWS Integration.md`
- `src/services/bedrock_llm/client.py`
- `src/services/bedrock_llm/factory.py`
- `src/services/bedrock_guardrails/service.py`
- `src/services/bedrock_guardrails/factory.py`
- `scripts/create_bedrock_guardrail.py`
- `src/exceptions.py`
- `.env.example`
- `deployment/k8s/secrets/secret-template.yaml`
- `.github/workflows/cd.yml`

### Concepts

- AWS credential provider chain, explicit access keys versus IRSA, regions, model IDs versus inference profile ARNs.
- boto3 sync clients and wrapping blocking calls with `asyncio.to_thread`.
- Bedrock `converse`, `converse_stream`, `ApplyGuardrail`, guardrail versioning, input versus output source, content qualifiers, PII anonymization, topic/content policies, contextual grounding, and failure handling.
- Provider abstraction: a Python protocol can standardize methods without guaranteeing every provider accepts the same model ID, structured output mode, tool schema, or safety feature.

### Actual guardrail flow

- `check_input` sends raw query content as `INPUT`.
- `check_output` sends source documents as `grounding_source`, query as `query`, and answer as `guard_content` with source `OUTPUT`.
- `INTERVENED` is allowed only when the assessment is interpreted as PII anonymization without a hard block.
- Missing `BEDROCK__GUARDRAIL_ID` and API errors are fail-open.
- The guardrail result is converted into legacy score/reason fields in `AgentState`.

### Safety/operations exercises

- Create a test guardrail with topic denial, PII anonymization, content filters, and grounding thresholds using `scripts/create_bedrock_guardrail.py`.
- Mock `ApplyGuardrail` responses for `NONE`, topic block, PII anonymization, content block, and malformed assessments.
- Verify the resulting agent response and whether the query/answer is replaced, sanitized, or passed.
- Audit whether standard `/ask`, `/stream`, A2A, MCP and Telegram paths receive the same safety checks; they do not all use the same path.

---

## 12. Supervisor agent, A2A, MCP, Telegram, and Gradio

### Supervisor

Read:

- `src/services/agents/supervisor_agent.py`
- `src/services/agents/summarizer_agent.py`
- `src/routers/supervisor_ask.py`
- `src/main.py` supervisor construction

Learn intent classification, delegation, reuse of a shared service context, and why a router that reads `request.app.state.supervisor_agent` is different from a typed FastAPI dependency.

Actual behavior:

- One direct LLM call classifies the query as `summarize` if the output contains that word; everything else becomes `rag_lookup`.
- Summarization does BM25-only top-five search and a second direct LLM call.
- Summary results do not include source objects.
- No dedicated supervisor tests were found; add route and classifier tests.

### A2A

Read:

- `src/routers/a2a.py`
- `src/services/a2a/models.py`
- `workflows/kubernetes/07-request-flow.md` for the external request context

Learn agent cards, task/message/part/artifact models, synchronous task completion, and protocol version/feature negotiation. Current implementation advertises no streaming, push notifications or state-transition history and returns a completed task immediately.

Risk: `a2a.py` constructs a new `Settings()` and always creates `OpenAILLMClient`; it ignores the global `PROVIDER` and shared LLM/guardrail/trace services. It also performs embedding and synchronous OpenSearch directly in an async route.

### MCP

Read:

- `src/mcp_server/server.py`
- `src/mcp_server/tools/search.py`
- `src/mcp_server/tools/ask.py`
- `src/mcp_server/tools/feedback.py`
- `src/mcp_server/tools/papers.py`
- `src/mcp_server/resources/papers.py`
- `src/main.py` MCP lifespan/mount wiring
- `.env.example` MCP settings

Learn MCP server/tool/resource registration, stateless HTTP transport, global context wiring, JSON-serializable tool contracts, resource URIs, and the security implications of exposing a tool server.

Active capabilities:

- Tool `search_papers`.
- Tool `ask_question` using the shared agentic service.
- Tool `submit_feedback`.
- Tool `get_index_stats`.
- Tool `get_paper_details`.
- Tool `list_recent_papers`.
- Resource `papers://{arxiv_id}`.
- Resource `index://stats`.

Risk: `MCPContext.llm_client` is typed as `OpenAILLMClient`, but Bedrock can be injected. Several tools use synchronous DB/OpenSearch operations inside async functions.

### Telegram

Read:

- `src/services/telegram/bot.py`
- `src/services/telegram/factory.py`
- `tests/unit/services/test_telegram.py`
- `src/main.py` lock/start/stop path
- `notebooks/phase7/README.md`
- `workflows/phase7_agentic_rag.md`

Learn polling lifecycle, command/message handlers, Markdown escaping, Telegram message limits, per-user identity, and process locks.

Actual handlers are `/start`, `/help`, `/search`, and ordinary text questions. The phase README claims `/settings`, `/status`, `/clear`, model selection, session preferences, and other modules that do not exist in the current `src/services/telegram/` directory.

### Gradio

Read:

- `src/gradio_app.py`
- `gradio_launcher.py`
- `README.md`
- `workflows/phase5_rag_pipeline.md`

Learn an async HTTP client feeding a Gradio generator, UI inputs, streaming Markdown updates, and deployment URL configuration. The client assumes localhost and expects text lines containing `data:` JSON; it is not configured through an environment variable or reverse-proxy-relative URL.

---

## 13. Docker and local operations

### Read

- `Dockerfile`
- `compose.yml`
- `airflow/Dockerfile`
- `entrypoint.sh`
- `airflow/entrypoint.sh`
- `airflow/README.md`
- `docs/infra_start.md`
- `docs/tear_down.md`
- `scripts/infra_start.sh`
- `scripts/tear_down.sh`
- `scripts/secrets.sh`
- root `secrets.sh`

### Concepts

- Multi-stage image construction and copying a prebuilt virtual environment.
- CPU-only Torch index and why the project tries to avoid CUDA layers.
- Root build context for `airflow/Dockerfile` and why `COPY src` works in CI but not with an arbitrary subdirectory context.
- Uvicorn workers versus one Airflow scheduler/webserver process.
- Bind mounts for source/DAG development versus baked code in Kubernetes.
- Health checks and startup order.
- Secret injection and why `.env.example` is not a usable credential file.

### Commands

```bash
docker compose up --build -d
docker compose ps
docker compose logs api
docker compose logs airflow
docker compose exec api python -c 'import sys; print(sys.version)'
docker compose exec opensearch curl -s http://localhost:9200/_cluster/health
docker compose down
docker compose down -v
```

Do not run destructive teardown commands against a real AWS account until you understand `scripts/tear_down.sh`, its cluster/ECR/load-balancer/CloudFormation behavior, and which managed cloud services it does **not** remove.

### Script runbook

- `uv run python scripts/test_connections.py` checks OpenAI, PostgreSQL, Redis, Langfuse and Jina connectivity; it requires real credentials and should not be treated as a unit test.
- `uv run python scripts/insert_papers_by_id.py 1706.03762` is the focused paper-ingestion/indexing path; inspect flags such as `--skip-pdf`, `--replace-existing`, and the actual service dependencies before using it.
- `uv run python scripts/create_bedrock_guardrail.py` creates AWS resources and should be run only with an intentional region/account and least-privilege policy.
- `./scripts/infra_start.sh [cluster-name] [region] [account-id]` creates ECR/EKS/IAM/storage and deploys the stack. It needs `aws`, `eksctl`, `kubectl`, Docker and optionally Helm.
- `./scripts/tear_down.sh` is destructive. Read every deletion branch, confirm the cluster/account/region, and export any required variables before running it.
- `python gradio_launcher.py` starts the separate Gradio process, normally on port 7861; it expects the API at the hard-coded localhost URL in `src/gradio_app.py`.
- `uv run locust -f locustfile.py` is the likely load-test entry point; inspect `locustfile.py` and target host before generating traffic. The repository also has `scripts/load_test.py` and `scripts/ramp_load_test.py`.

---

## 14. Kubernetes, EKS, IRSA, HPA, and monitoring

### Read in dependency order

1. `deployment/eks/cluster.yaml`
2. `deployment/eks/namespace.yaml`
3. `deployment/k8s/opensearch/service.yaml`
4. `deployment/k8s/opensearch/statefulset.yaml`
5. `deployment/k8s/opensearch-dashboards/deployment.yaml` and `service.yaml`
6. `deployment/k8s/api/deployment.yaml`, `service.yaml`, `hpa.yaml`
7. `deployment/k8s/airflow/deployment.yaml`, `service.yaml`
8. `deployment/k8s/secrets/secret-template.yaml`
9. `deployment/DEPLOYMENT_GUIDE.md`
10. `docs/updated_kubernetes.md`
11. `docs/infra_start.md`
12. `docs/grafana_integration.md`
13. `deployment/grafana/values.yaml`
14. `workflows/kubernetes/01-eks-cluster-architecture.md` through `07-request-flow.md`
15. `workflows/kubernetes/README.md`

### Concepts to learn

- EKS control plane, node groups, VPC/private networking, ECR, EBS CSI and persistent volumes.
- Kubernetes namespace, labels/selectors, Deployment/ReplicaSet, StatefulSet, Service types, headless service DNS, init containers, probes, resources, anti-affinity, rolling updates, ConfigMaps, Secrets, service accounts and RBAC.
- IRSA/OIDC and AWS credential precedence.
- HPA control loops, resource requests, Metrics Server, pending pods, cluster autoscaling, scale-up/down stabilization.
- ELB/LoadBalancer services, TLS/authentication, network exposure, and reverse-proxy behavior.
- Grafana Alloy/Kubernetes Monitoring, Prometheus remote write, Loki, kube-state-metrics, node exporter, OpenCost and LogQL.

### Actual topology

- `m5.xlarge`, 2–4 managed nodes in `deployment/eks/cluster.yaml`.
- OpenSearch is a one-replica StatefulSet with a PVC and security disabled.
- API starts at 2 replicas, requests 6 Gi memory/500m CPU, and HPA scales 2–6 based on CPU 70% and memory 80%.
- Airflow is one replica using LocalExecutor, with 2 Gi request/5 Gi limit.
- API, Airflow, and Dashboards use `LoadBalancer` Services; OpenSearch is internal `ClusterIP` plus a headless Service.
- API and Airflow use `envFrom` with `rag-app-secrets`; API references `rag-api-sa`.
- `deployment/eks/cluster.yaml` enables OIDC, but the deployment path also provisions explicit Bedrock access keys in the secret. Explicit credentials can prevent the AWS SDK from using IRSA, so the claimed keyless pod path is not the effective path when keys are present.

### Deployment discrepancies/risks to investigate

- The workflow creates an `airflow-dags` ConfigMap, but the current Airflow Deployment does not mount it; DAGs are baked into the Airflow image. The CD comments say both that it is mounted and that ConfigMap recursion is not used.
- The API readiness probe calls `/api/v1/health`, which checks OpenAI even for Bedrock deployments.
- Public LoadBalancers expose API/Airflow/Dashboards without an evident TLS/authentication layer in these manifests.
- OpenSearch security is disabled and the client disables TLS verification.
- The documents say a cluster autoscaler adds nodes, but no active cluster-autoscaler installation is part of the core manifests/scripts reviewed. HPA may request pods that remain Pending when node capacity is exhausted.
- The HPA and Deployment resource math must be tested against real allocatable node capacity; the workflow diagrams already describe a 6 Gi scheduling ceiling.
- The secret template permits static AWS credentials and has placeholders; do not apply it with real values committed to Git.

### Commands to practice safely

```bash
kubectl apply --dry-run=client -f deployment/eks/namespace.yaml -o yaml
kubectl diff -f deployment/k8s/
kubectl get pods -n production -o wide
kubectl describe pod <pod> -n production
kubectl logs deployment/rag-api -n production --all-containers --tail=200
kubectl get svc,hpa,statefulset -n production
kubectl top pods -n production
kubectl get events -n production --sort-by=.lastTimestamp
kubectl rollout status deployment/rag-api -n production
kubectl rollout history deployment/rag-api -n production
```

For an actual cluster, follow `scripts/infra_start.sh` and `deployment/DEPLOYMENT_GUIDE.md` only after validating costs, regions, credentials, and teardown steps.

---

## 15. CI/CD and GitHub Actions

### Read

- `.github/workflows/ci.yml`
- `.github/workflows/cd.yml`
- `.pre-commit-config.yaml`
- `pyproject.toml`
- `deployment/k8s/*`
- `workflows/kubernetes/05-cicd-pipeline.md`

### Learn

- GitHub Actions triggers, path filters, matrix jobs, job dependencies, outputs, caches and concurrency groups.
- Reproducible dependency installation with `uv sync --frozen --dev`.
- Ruff import linting/format checks, mypy configuration, pytest collection and coverage.
- ECR image tags, Docker Buildx, GitHub Actions cache, Kubernetes rollout and immutable image tags.
- Secret handling and the difference between GitHub Secrets, Kubernetes Secrets, IRSA and application configuration.

### Actual CI

CI runs on PRs to `main`, `aws`, and `agentops`, ignoring `deployment/**`, top-level Markdown, and notebooks. It installs Python 3.12 and frozen dependencies, runs Ruff, runs informational mypy, runs pytest, creates coverage, and runs the golden evaluation gate.

`pyproject.toml` sets `ignore_errors=true` for mypy and the workflow uses `continue-on-error: true`; type checking is therefore non-blocking.

The golden test in `tests/eval/test_golden_dataset.py` uses five cases from `data/golden_dataset.json`, mocks the whole agent service, and verifies non-empty answer/source structure. It is a pipeline-shape test, not factual or retrieval-quality evaluation.

### Actual CD

CD runs on pushes to `deployment` and `agentops`, builds API/Airflow images, pushes SHA and `latest` tags to ECR, configures kubectl, creates the namespace/secret, deploys OpenSearch, creates the unused Airflow ConfigMap, substitutes the SHA image into manifests with `sed`, applies services/workloads/HPA, waits for API/Airflow rollouts, and prints external URLs.

Compare this with workflow prose that describes only a `deployment` branch or diagrams that say `agentops`. Branch names and the actual trigger list must be treated as separate facts.

### Exercises

- Run the CI commands locally after installing Python 3.12 and `uv`.
- Deliberately introduce a type error and explain why CI remains green.
- Validate all Kubernetes YAML with `kubectl apply --dry-run=client` or a schema validator.
- Replace mutable `latest`-only reasoning with immutable SHA-tag reasoning and verify the manifest actually changes per deploy.
- Inspect whether a docs-only change triggers the desired CI/CD behavior.

---

## 16. Testing and evaluation map

### Test files and what they teach

| Area | Files | What is actually covered |
|---|---|---|
| Configuration | `tests/unit/test_config.py` | Defaults and settings initialization |
| Search schemas | `tests/unit/schemas/test_search.py` | Pydantic validation, aliases, hit/response shape |
| Query DSL | `tests/unit/services/test_opensearch_query_builder.py` | Multi-match fields, filters, sort, highlighting |
| arXiv | `tests/unit/services/test_arxiv_client.py` | XML parsing, URL/query construction, timeout/status errors, cache path/rate delay |
| PDF | `tests/unit/services/test_pdf_parser.py` | Validation and mocked Docling success/failure |
| Metadata | `tests/unit/services/test_metadata_fetcher.py` | Pipeline defaults and empty/concurrency behavior |
| Agent models/nodes/tools | `tests/unit/services/agents/` | Models, node behavior, service initialization and graph invocation with mocks |
| API routes | `tests/api/routers/` | HTTP validation/response structure; most external work is mocked or status-tolerant |
| Telegram | `tests/unit/services/test_telegram.py` | Factory/settings and bot wiring |
| Integration | `tests/integration/test_services.py` | Real arXiv/OpenSearch connectivity if run; not isolated unit tests |
| Golden evaluation | `tests/eval/test_golden_dataset.py` | Mocked response structure, not answer truth |

### Known test gaps

- No dedicated tests for supervisor, summarizer, A2A, MCP, Bedrock client, Bedrock guardrails, Redis cache, Langfuse, Logfire, deployment manifests, or Airflow task dependencies.
- `tests/unit/services/agents/test_agentic_rag.py` explicitly sets threshold 60, while factory production defaults result in 40.
- `tests/api/routers/test_ask.py` permits 500/503 in several tests, which can hide service wiring failures.
- The golden test patches `AgenticRAGService.ask` and then calls a `MagicMock` service; it does not execute the graph, retrieve documents, or evaluate factuality.
- No test asserts that agent API request `top_k`, `use_hybrid`, or categories reach retrieval.
- No test covers actual SSE content type, cache-hit whitespace splitting, disconnect cancellation, or Bedrock stream events.
- No test asserts cache invalidation after indexing or response differences after prompt/model changes.

### Recommended test progression

1. Run unit tests only.
2. Fix/characterize import and dependency failures.
3. Add focused regression tests for every risk in this plan.
4. Run mocked API tests with lifespan enabled.
5. Run a local OpenSearch/PostgreSQL integration environment.
6. Run a small end-to-end ingestion of one known paper.
7. Evaluate retrieval with a labeled set, then answer quality with citations/grounding labels.
8. Run load tests only after defining safe concurrency and cloud cost limits.

Commands:

```bash
uv run pytest tests/unit -q
uv run pytest tests/api -q
uv run pytest tests/eval -q
uv run pytest --cov=src --cov-report=term-missing
uv run python scripts/load_test.py
uv run python scripts/ramp_load_test.py
```

`locustfile.py` is the other performance-test entry point; inspect its host, request shape and concurrency before running it against a cloud endpoint.

---

## 17. Documentation drift register

Use this as a checklist when consulting prose:

| Document claim | Executable reality to verify |
|---|---|
| Root quick start clones `sourangshupal/Agentic-RAG-project` and checks out `develop` | This checkout is `Vaibhav-Lohar-28/Agentic-RAG-System`; the active session branch is different. Treat the command as historical. |
| README/step guide says four local containers and cloud OpenAI stack | Compose does have four services, but active code also supports Bedrock and has no Ollama/Gradio Compose service. |
| Phase 2 says date-based ingestion and 15 papers | Active Airflow fetching uses title keyword `transformer` and max 2; `target_date` is ignored. |
| `airflow/README.md` says Airflow 3.0/10 default and OpenSearch placeholders | `airflow/Dockerfile` installs 2.10.3; current DAG uses hybrid indexing and secret/deployment overrides set max 2. |
| Phase 7 says LLM decides whether to retrieve and refers to `nodes.py` | Current graph has a guardrail route and deterministic `retrieve_node`; actual nodes are separate files under `src/services/agents/nodes/`. |
| Phase 7 advertises `/settings`, `/status`, `/clear` and rich Telegram preferences | Current Telegram code implements only `/start`, `/help`, `/search`, and text questions. |
| AWS Integration says provider switch is transparent for all routes | A2A always uses OpenAI; health checks OpenAI; standard RAG does not use agent guardrails; model IDs remain provider-specific. |
| Workflow phase 5 calls streaming output SSE | Server emits SSE-like `data:` frames but declares `text/plain`. |
| Workflow phase 7 says output guardrail is in graph | This is current and must be studied; older diagrams that omit it are incomplete. |
| K8s/CD says Airflow DAG ConfigMap is mounted | CD creates it, but current Airflow Deployment uses the baked-in DAGs and has no ConfigMap volume mount. |
| K8s docs describe IRSA keyless Bedrock access | Secret/workflow also inject explicit Bedrock keys; explicit keys can take precedence. |
| README references `docs/kubernetes.md` and several phase-7 planning docs | Those paths are absent in this checkout; use `deployment/DEPLOYMENT_GUIDE.md`, `docs/updated_kubernetes.md`, and executable source. |
| Some old docs reference `deployment/k8s/namespace.yaml` or `src/services/opensearch/mappings.py` | Actual files are `deployment/eks/namespace.yaml` and `src/services/opensearch/index_config_hybrid.py`. |
| Some documentation references `scripts/insert_test_paper.py` | It is absent; use `scripts/insert_papers_by_id.py`. |
| `Makefile:health` curls `/health` and an Ollama endpoint at `:11434` | Active API health is `/api/v1/health`; Compose has no Ollama service, so treat that target as stale until corrected. |
| Root `requirements-airflow.txt` and `airflow/requirements-airflow.txt` constrain SQLAlchemy `<2.0` while the main project requires `sqlalchemy>=2.0.0` | These requirements are consumed by the separate Airflow image; do not mix them into the API environment without resolving the conflict. |
| Phase/Compose documentation says OpenSearch is 2.19.5 and maps a local index name | The Python default forms `arxiv-papers-chunks` from base name plus suffix; old examples that set `OPENSEARCH__INDEX_NAME=arxiv-papers-chunks` would form `arxiv-papers-chunks-chunks`. |

---

## 18. Capstone research projects

Choose these only after the execution traces are understood.

### A. Retrieval quality study

- Build a labeled query set across exact keywords, paraphrases, abbreviations, categories and recent papers.
- Compare BM25, KNN, RRF, different `size` multipliers and HNSW parameters.
- Report recall@k, MRR/nDCG, duplicate-paper rate, latency, embedding cost and RRF score distributions.
- Include the effect of section headers and repeated title/abstract content.

### B. Agent reliability study

- Create a state-transition test matrix for allowed/out-of-scope, relevant/irrelevant, rewrite, max attempts and guardrail errors.
- Measure extra LLM calls, retrieval attempts, latency, failure mode and answer groundedness.
- Compare aggregate grading with per-document grading.
- Replace regex source extraction in a prototype and compare robustness.

### C. Provider parity study

- Run the same standard, streaming and agentic cases with OpenAI and Bedrock.
- Compare model selection, structured output, tool behavior, stream event translation, usage accounting, guardrails and failures.
- Explicitly test invalid cross-provider model IDs.

### D. Production hardening study

- Remove static AWS keys in favor of IRSA and verify credential source from inside a pod.
- Define provider-neutral health/readiness checks.
- Add TLS/authentication for external services and restrict public LoadBalancers.
- Make every HTTP/Redis/OpenSearch/trace client lifecycle explicit.
- Validate HPA plus node autoscaling under bounded Locust load.

### E. Evaluation/feedback study

- Turn `data/golden_dataset.json` into a real labeled evaluation set with reference answers and expected citations.
- Separate retrieval, groundedness, citation correctness, helpfulness and refusal scores.
- Compare automatic Langfuse heuristic scores with human labels.
- Use trace IDs to connect user feedback to model/provider/prompt/retrieval versions.

---

## 19. External canonical references

Use the repository files for project-specific behavior, then consult the official technology documentation for the underlying concepts:

- FastAPI lifespan and startup/shutdown: <https://fastapi.tiangolo.com/advanced/events/>
- FastAPI testing of lifespan: <https://fastapi.tiangolo.com/advanced/testing-events/>
- LangGraph tools, `ToolNode`, conditional routing: <https://docs.langchain.com/langgraph-platform/langgraph-basics/2-add-tools>
- OpenSearch hybrid search: <https://docs.opensearch.org/latest/vector-search/ai-search/hybrid-search/index/>
- OpenSearch score-ranker/RRF processor: <https://docs.opensearch.org/latest/search-plugins/search-pipelines/score-ranker-processor/>
- OpenSearch hybrid query: <https://docs.opensearch.org/latest/query-dsl/compound/hybrid/>
- Apache Airflow DAGs: <https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html>
- Apache Airflow tasks: <https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html>
- Amazon Bedrock `ApplyGuardrail`: <https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-use-independent-api.html>
- Amazon Bedrock Converse with guardrails: <https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-use-converse-api.html>
- Kubernetes HPA: <https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/>
- Kubernetes workload/autoscaling concepts: <https://kubernetes.io/docs/concepts/workloads/autoscaling/>
- Pydantic settings: <https://docs.pydantic.dev/latest/concepts/pydantic_settings/>
- Redis Python client: <https://redis.readthedocs.io/en/stable/>
- Jina embeddings: <https://jina.ai/embeddings>
- Docling: <https://docling-project.github.io/docling/>
- Langfuse: <https://langfuse.com/docs>
- AWS EKS IRSA: <https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html>

---

## 20. Short-term study schedule

| Session | Deliverable |
|---|---|
| 1 | Repository map, dependency/runtime diagram, working Python/uv environment |
| 2 | Settings matrix and startup sequence diagram |
| 3 | OpenAPI route/contract table and validation tests |
| 4 | Postgres model/repository transaction exercise |
| 5 | arXiv XML + PDF parser failure matrix |
| 6 | Chunker unit tests and documented defect list |
| 7 | OpenSearch BM25/KNN/RRF comparison |
| 8 | Standard and streaming RAG trace with cache miss/hit |
| 9 | Provider parity and Bedrock guardrail mock tests |
| 10 | LangGraph state snapshots for every branch |
| 11 | MCP/A2A/Telegram/Gradio interface comparison |
| 12 | Airflow DAG/XCom and local ingestion run |
| 13 | Docker and Kubernetes manifest validation |
| 14 | CI/CD review, golden-evaluation redesign, capstone proposal |

At the end, you should be able to answer—not merely repeat documentation—where a query is validated, whether it is embedded, which OpenSearch mode ran, which model/provider generated the answer, how source metadata was assembled, where guardrails applied, what was cached/traced, and which deployment component controls the process.
