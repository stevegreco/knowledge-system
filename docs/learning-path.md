# Learning path

The project is split into issues, worked through in order. Each issue teaches one concept and is small enough to finish in one or two sittings. Each one has:

- **Concept**: what you should understand when it's done
- **Build**: the smallest thing that makes the concept concrete
- **Done when**: checks you can run or show
- **Check your understanding**: questions to answer in your own words before moving on

Each issue maps to one GitHub issue. Phase gates come from the [README roadmap](../README.md#roadmap): don't start a phase until the previous gate is met.

| # | Issue | Phase |
| --- | --- | --- |
| 01 | Settle the open questions and pick a corpus | 0: Foundations |
| 02 | Python project setup | 0: Foundations |
| 03 | Postgres + pgvector in Docker | 0: Foundations |
| 04 | Embeddings by hand | 0: Foundations |
| 05 | Write the golden question set | 0: Foundations |
| 06 | Schema: documents, chunks, ingest_runs | v1: Baseline RAG |
| 07 | Discover and hash documents | v1: Baseline RAG |
| 08 | Parse documents to clean text | v1: Baseline RAG |
| 09 | Structure-aware chunking | v1: Baseline RAG |
| 10 | Embed and upsert chunks | v1: Baseline RAG |
| 11 | Idempotent re-runs, pruning, failure isolation | v1: Baseline RAG |
| 12 | Vector search in SQL | v1: Baseline RAG |
| 13 | Measure retrieval: hit rate @5 | v1: Baseline RAG |
| 14 | Generate a cited answer | v1: Baseline RAG |
| 15 | Query API | v1: Baseline RAG |
| 16 | Measure answers and record the v1 baseline | v1: Baseline RAG (gate) |
| 17 | Failure analysis of v1 misses | v2: Retrieval quality |
| 18 | Keyword search with Postgres full-text | v2: Retrieval quality |
| 19 | Hybrid retrieval with reciprocal rank fusion | v2: Retrieval quality |
| 20 | Reranking and the v2 report | v2: Retrieval quality (gate) |
| 21 | Access groups and test users | v3: Enterprise controls |
| 22 | Permission-filtered retrieval | v3: Enterprise controls |
| 23 | Query logging and tracing a bad answer | v3: Enterprise controls (gate) |
| 24 | Stretch experiments | Optional |

---

## Phase 0: Foundations

Nothing gets ingested yet. These issues set up the tools and the yardstick.

### 01 · Settle the open questions and pick a corpus

**Concept:** A RAG system is only as interesting as its corpus. Format mix, size, and access boundaries decide what you'll get to learn.

**Build:**
- Shortlist two or three candidate corpora and compare them on formats, size (a few hundred docs), and natural access boundaries
- Answer the remaining open questions in the README: time budget, LLM and embedding provider, cost ceiling, UI vs CLI, LLM judge or not
- Record each decision and its reason in the README

**Done when:** every README open question is checked off with a one-line answer, and the corpus can be downloaded with a documented command.

**Check your understanding:**
- Why does a corpus with only one format teach less than a mixed one?
- What part of your corpus could act as "restricted" content in v3?

### 02 · Python project setup

**Concept:** A reproducible environment: pinned dependencies, a package layout, and a way to run code and tests.

**Build:**
- Choose a package manager (for example `uv` or `pip` + `venv`) and create `pyproject.toml`
- Create a package directory and an empty test that passes
- Add a `.env.example` listing the settings you'll need (database URL, API keys)

**Done when:** a fresh clone can install and run the empty test with two documented commands.

**Check your understanding:**
- What does a lock file protect you from?
- Why do secrets go in `.env` and not in code or git?

### 03 · Postgres + pgvector in Docker

**Concept:** pgvector adds a `vector` column type and distance operators to Postgres, so vectors live next to normal data.

**Build:**
- `docker-compose.yml` with a Postgres image that includes pgvector, and a named volume
- Enable the extension and, in `psql`, create a scratch table with a 3-dimension vector column. Insert a few rows and query them by distance (`<->`, `<=>`)

**Done when:** `docker compose up` starts the database, data survives a restart, and you can show a nearest-neighbor query on toy vectors.

**Check your understanding:**
- What is the difference between L2 distance and cosine distance? When do they rank results the same?
- What happens to the data if you remove the volume?

### 04 · Embeddings by hand

**Concept:** An embedding model maps text to a fixed-length vector, and similar meanings land near each other. This is the core idea behind retrieval.

**Build:** a small script or notebook that:
- Loads a local sentence-transformers model and prints the vector length
- Embeds 10 to 15 sentences, including paraphrases, unrelated sentences, and keyword-overlap-but-different-meaning pairs
- Computes cosine similarity yourself (with numpy) and prints a ranked list for a few queries

**Done when:** you can point at a pair where embeddings beat keyword matching, and a pair where they fail.

**Check your understanding:**
- Why must every vector in a column come from the same model?
- What does the model's maximum input length mean for chunk size later?

### 05 · Write the golden question set

**Concept:** Evaluation comes before tuning. A fixed question set turns "seems better" into a number.

**Build:**
- Read through the corpus and write 30 to 50 questions in a data file (for example `eval/golden.jsonl`)
- Each entry has the question, the expected answer, and the source document(s)
- Include a mix: single-fact lookups, questions that need two documents, questions phrased differently from the source, and 5 or so with no answer in the corpus

**Done when:** the file exists, loads with a few lines of Python, and has no-answer questions in it.

**Check your understanding:**
- Why write these before building retrieval, rather than after?
- Which of your questions do you expect pure vector search to get wrong, and why?

---

## Phase v1: Baseline RAG

The simplest complete pipeline: ingest, retrieve, answer, measure. Resist improving anything until issue 16 has recorded a baseline.

### 06 · Schema: documents, chunks, ingest_runs

**Concept:** The data model is the contract between ingestion and querying. Vector dimension is fixed by the embedding model.

**Build:** a SQL migration (a plain `.sql` file is fine) creating the three tables from the README data model, applied by one command.

**Done when:** the tables exist after a fresh `docker compose up` plus the migration command, and `chunks.embedding` has the dimension your issue 04 model produces.

**Check your understanding:**
- Why store `parsed_text` on documents instead of re-parsing?
- Why is `embedding_model` stored per chunk rather than once globally?

### 07 · Discover and hash documents

**Concept:** Incremental ingestion. A content hash tells you what changed without comparing full text.

**Build:** an ingest CLI command that walks the corpus, computes SHA-256 per file, inserts or updates `documents` rows, skips unchanged ones, and records counts in `ingest_runs`. No parsing yet.

**Done when:** the first run reports N new docs; a second run reports N skipped; editing one file reports 1 updated.

**Check your understanding:**
- Why hash the content rather than trust the last-modified time?
- What would break if two documents had the same `source_uri`?

### 08 · Parse documents to clean text

**Concept:** Garbage in, garbage out. Parsing quality is the single biggest lever on retrieval quality.

**Build:** a parser per format (Markdown, HTML, PDF) that produces clean text and keeps heading structure. Store the result in `documents.parsed_text`. Save a few before/after samples to look at.

**Done when:** you've eyeballed at least one parsed document per format and fixed the worst noise (navigation menus, headers, footers, broken lines).

**Check your understanding:**
- What noise did each format introduce, and how did you remove it?
- Why keep headings instead of flattening to plain paragraphs?

### 09 · Structure-aware chunking

**Concept:** Chunks are the unit of retrieval. Their size and boundaries decide what the model can find and quote.

**Build:** a chunker that splits on headings first, then by size (about 300 to 800 tokens, 10 to 15% overlap), and records each chunk's section path. Prepend the section path to the text that will be embedded. Write a few unit tests with small inputs.

**Done when:** tests pass, and you can print the chunks for one real document and explain every boundary.

**Check your understanding:**
- What goes wrong with chunks that are too small? Too large?
- What does the overlap protect against?
- Why prepend the section path before embedding?

### 10 · Embed and upsert chunks

**Concept:** Batched embedding and transactional writes keep the index consistent with the source.

**Build:** wire chunking and embedding into the ingest command. For each changed document, in one transaction: delete its old chunks, insert the new ones with vectors and `embedding_model`, and update the hash.

**Done when:** the full corpus ingests, the chunk count looks sane, and killing the process mid-run leaves no document with half its chunks.

**Check your understanding:**
- What would a reader see if the delete and insert weren't in one transaction?
- Why embed in batches instead of one chunk at a time?

### 11 · Idempotent re-runs, pruning, failure isolation

**Concept:** Operating a pipeline. A re-run should be safe, cheap, and honest about what failed.

**Build:**
- Prune documents (and their chunks) that no longer exist in the source
- Catch failures per document, record them in `ingest_runs.errors`, and keep going

**Done when:** deleting a source file removes its chunks on the next run; a deliberately broken file is logged as failed while every other doc still ingests.

**Check your understanding:**
- What does "idempotent" mean for this command, concretely?
- Why is one failed document not allowed to stop the run?

### 12 · Vector search in SQL

**Concept:** Retrieval is a nearest-neighbor query. An approximate index (HNSW) trades a little recall for a lot of speed.

**Build:**
- A function that embeds a question and returns the top-k chunks with their distance, document, and section path
- Add an HNSW index and compare `EXPLAIN ANALYZE` before and after

**Done when:** you can run five golden questions and read the retrieved chunks, and you can explain the query plan change.

**Check your understanding:**
- What does "approximate" mean in approximate nearest neighbor?
- Why must the question use the same embedding model as the chunks?

### 13 · Measure retrieval: hit rate @5

**Concept:** Measure retrieval on its own, before an LLM gets involved, so you know which half of the system to blame.

**Build:** an eval script that runs every golden question through retrieval and reports hit rate @5 (did a correct source document appear in the top 5?), plus a list of misses.

**Done when:** the script prints a single number and a miss list, and the result is saved to a file you can compare against later.

**Check your understanding:**
- Why judge retrieval separately from answers?
- How do no-answer questions fit into hit rate?

### 14 · Generate a cited answer

**Concept:** The prompt is the contract with the LLM. It should answer only from the provided chunks, cite them, and refuse when they don't contain the answer.

**Build:** a function that takes a question and retrieved chunks, builds a prompt with numbered sources, calls the LLM, and parses the citations from the response.

**Done when:** for a few golden questions you get grounded answers with valid citation numbers, and a no-answer question gets a refusal.

**Check your understanding:**
- What in your prompt pushes the model to refuse instead of guessing?
- How do you detect a citation that points to a source you never sent?

### 15 · Query API

**Concept:** Wrapping the pipeline in a typed HTTP interface with clear request and response shapes.

**Build:** a FastAPI app with `POST /query` that returns the answer and its cited sources (document title, URI, section path, chunk text). Add one test using the test client.

**Done when:** `curl` returns a cited answer and the test passes.

**Check your understanding:**
- What belongs in the API layer, and what belongs in the retrieval and generation functions?
- What should the API return when retrieval finds nothing?

### 16 · Measure answers and record the v1 baseline

**Concept:** End-to-end evaluation: answer correctness, citation accuracy, and correct refusals.

**Build:** extend the eval script to call the API, then grade answers by hand (pass/fail) and check citation accuracy. Save the results as the v1 baseline.

**Done when (v1 gate):** the full corpus ingests, re-runs skip unchanged docs, and the baseline scores are committed.

**Check your understanding:**
- When an answer is wrong, how do you tell a retrieval failure from a generation failure?
- Which metric surprised you most?

---

## Phase v2: Retrieval quality

Every change in this phase is judged against the v1 baseline.

### 17 · Failure analysis of v1 misses

**Concept:** Debugging bad answers. Group failures by cause before choosing a fix.

**Build:** for every v1 miss, write down the cause: bad parse, bad chunk boundary, vocabulary mismatch, exact term or ID lost by embeddings, or a generation error. Count each category.

**Done when:** you have a table of misses by cause and a prediction of which v2 change will help which category.

**Check your understanding:**
- Which failure category do you expect keyword search to fix?
- Which failures will no retrieval change fix?

### 18 · Keyword search with Postgres full-text

**Concept:** Lexical search (BM25-style ranking) finds exact terms, names, and IDs that embeddings blur.

**Build:** a `tsvector` column with a GIN index on chunks, and a keyword retrieval function. Run hit rate @5 for keyword-only search.

**Done when:** you have a keyword-only score next to the vector-only score, plus examples where each one wins.

**Check your understanding:**
- What do stemming and stop words do to a query?
- Why might keyword search beat vectors on some of your questions?

### 19 · Hybrid retrieval with reciprocal rank fusion

**Concept:** Combine rankings instead of raw scores, because vector distances and text-rank scores aren't on the same scale.

**Build:** run both searches, merge them with RRF (score = sum of 1 / (k + rank)), and evaluate.

**Done when:** hybrid hit rate @5 is recorded next to vector-only and keyword-only.

**Check your understanding:**
- Why fuse ranks rather than normalize scores?
- What does the `k` constant in RRF change?

### 20 · Reranking and the v2 report

**Concept:** A cross-encoder reads the question and each chunk together. It is more accurate than embeddings but too slow to run on the whole corpus, so it reorders a short candidate list.

**Build:** retrieve a wider candidate set (for example 30) with hybrid search, rerank with a cross-encoder, and keep the top 5. Write a side-by-side report of all four retrieval setups.

**Done when (v2 gate):** hybrid plus rerank beats the v1 baseline on hit rate @5, and the report shows latency cost next to the quality gain.

**Check your understanding:**
- Why not use the cross-encoder for first-stage retrieval?
- Was the quality gain worth the added latency for your corpus?

---

## Phase v3: Enterprise controls

### 21 · Access groups and test users

**Concept:** Permissions belong on the data at ingestion time, not bolted on at answer time.

**Build:** assign access groups to documents based on the corpus boundary from issue 01, copy them into each chunk's metadata, and define three test users at different permission levels. Add golden questions whose answers are restricted.

**Done when:** every chunk has access groups and the test users are defined in one place.

**Check your understanding:**
- Why copy groups onto chunks instead of joining to documents at query time?
- What happens to permissions when a document's access changes?

### 22 · Permission-filtered retrieval

**Concept:** Filter before ranking, not after. Post-filtering can return too few results or leak restricted content through the answer.

**Build:** pass the user into `/query`, filter chunks by access group inside the SQL for both vector and keyword search, and add a leakage check to the eval.

**Done when:** zero restricted chunks are returned to unauthorized test users across the whole eval set.

**Check your understanding:**
- What goes wrong if you retrieve the top 5 and then remove restricted chunks?
- How does filtering interact with the HNSW index?

### 23 · Query logging and tracing a bad answer

**Concept:** Observability: every answer should be traceable to the chunks, user, latency, and cost that produced it.

**Build:** a `query_log` table, written on every request, with question, user, retrieved chunk IDs, latency, token cost, and answer. Then pick one bad answer and trace it end to end using only the log.

**Done when (v3 gate):** every query is in the log, and you've written a short trace of one bad answer.

**Check your understanding:**
- Which field mattered most when tracing the bad answer?
- What would you need to add before logging real users' questions?

---

## 24 · Stretch experiments (optional)

Pick any, one at a time, each measured against the latest eval:

- LLM-as-judge for answer grading, checked against your manual grades
- Swap the local embedding model for a hosted one and compare
- Run ingestion as a scheduled job
- Swap pgvector for Qdrant or Weaviate and compare
- A minimal web page over the API
