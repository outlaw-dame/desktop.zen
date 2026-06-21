# Titan local-first semantic architecture

Titan's local-first AI/search layer should use SQLite, FTS5, and sqlite-vec as the initial architecture. This document records the intended direction and the specific ideas Titan should emulate from ObjectBox, Weaviate, and Meilisearch while improving on them for a browser-local, privacy-gated environment.

## Decision

Use a local embedded architecture:

```text
Titan profile directory
└── titan-semantic.sqlite
    ├── SQLite metadata tables
    ├── FTS5 lexical indexes
    ├── sqlite-vec vector tables
    ├── privacy/deletion state
    ├── embedding/model metadata
    ├── migration metadata
    └── diagnostics tables
```

This is an architecture direction, not a final implementation commitment. Before implementation, add an Architecture Decision Record comparing SQLite + sqlite-vec + FTS5 against PGlite, ObjectBox, native Firefox storage-only approaches, and external local services.

## Why SQLite + sqlite-vec + FTS5

Titan is a browser fork, not a cloud vector database. The semantic layer must be local-first, profile-scoped, migration-safe, deletion-aware, and able to run without network dependencies.

`sqlite-vec` is a SQLite vector extension written in pure C, with no dependencies, that runs anywhere SQLite runs and supports float, int8, and binary vectors in `vec0` virtual tables. This fits Titan's cross-platform browser constraints better than treating Postgres-only pgvector as if it were a SQLite extension.

FTS5 provides the lexical side of the hybrid index. sqlite-vec provides the semantic/vector side. SQLite metadata tables hold browser context, privacy state, chunk metadata, model identity, and lifecycle state.

## Goals

- Local by default.
- No silent remote indexing, embedding, sync, or retrieval.
- Queryable without uploading browsing data.
- Browser-aware retrieval over pages, notes, collections, tabs, workspaces, and approved history-derived artifacts.
- Transactional updates across content, FTS rows, vector rows, metadata, and deletion state.
- Explainable result scoring for Titan Assistant and user-facing search.
- Cross-platform data model that can later be shared conceptually with a Swift/WebKit companion.
- Rebase-friendly implementation that avoids deep Gecko modifications until benchmarks prove they are necessary.

## Non-goals for the first implementation

- Do not embed a full server database.
- Do not expose raw SQL to web content.
- Do not expose unrestricted vector search APIs to arbitrary websites.
- Do not index private browsing content.
- Do not index sensitive sites unless explicitly allowed.
- Do not modify Gecko storage internals during the prototype phase.
- Do not make cloud embedding providers mandatory.

## What Titan emulates from ObjectBox

ObjectBox's on-device vector search is useful as a design reference because it treats vector search as an embedded database feature rather than a loose in-memory ANN library.

Titan should emulate:

1. **On-device vector search**
   ObjectBox positions vector search as local on-device approximate nearest neighbor search for high-dimensional vectors. Titan should follow that local-first principle for browser memory, notes, semantic history, and AI context.

2. **Vectors attached to the data model**
   ObjectBox can store vectors alone or as part of a broader object model. Titan should never store embeddings as detached opaque blobs. Every vector row must connect to source, chunk, URL, workspace, model, privacy, and deletion metadata.

3. **HNSW-style parameter discipline**
   ObjectBox documents vector index dimensions, distance type, neighbors per node, indexing search count, repair behavior, and vector cache hints. Titan should similarly document vector dimension, distance metric, index/search parameters, memory cache policy, and deletion/repair behavior for sqlite-vec or any future vector extension.

4. **Transactions around bulk inserts**
   ObjectBox recommends wrapping bulk inserts in transactions. Titan should use transactions for indexing batches and for any operation that updates metadata + FTS + vector tables together.

5. **Embeddings are caller-owned**
   ObjectBox stores/searches vectors; the app is responsible for producing embeddings. Titan should preserve this boundary: embedding providers are separate from the semantic index.

6. **Disk-backed, update-friendly storage**
   ObjectBox highlights no full initial load, delta persistence, ACID transactions, and unified data storage. Titan should aim for the same operational properties: open the profile database and search without loading the full semantic index into memory.

## Titan improvements over ObjectBox

ObjectBox is a general embedded database. Titan can be more specific because it is a browser.

Titan should improve by adding:

- hard privacy gates before retrieval;
- browser-context filters for workspace, container/profile, tab state, collection membership, domain, recency, and deletion state;
- hybrid FTS + vector retrieval as the default, not vector-only search;
- per-site and per-workspace AI exclusions;
- deletion propagation to embeddings, FTS rows, summaries, and derived AI artifacts;
- explainable search diagnostics for every result used by Titan Assistant;
- future WebKit data-model portability without binding the companion app to Gecko internals.

## What Titan emulates from Weaviate

Weaviate is the strongest reference for hybrid retrieval behavior.

Titan should emulate:

1. **Hybrid-first retrieval**
   Weaviate hybrid search combines keyword/BM25-style retrieval with vector retrieval. Titan should combine FTS5 lexical search and sqlite-vec semantic search by default.

2. **Alpha-style balance control**
   Weaviate exposes `alpha`: `0` is pure keyword and `1` is pure vector. Titan should support the same internal concept, probably exposed as both numeric `semanticRatio` and user-facing modes such as Exact, Balanced, and Conceptual.

3. **Relative score fusion**
   Weaviate supports `relativeScoreFusion`, which preserves more score information than rank-only fusion. Titan should implement relative score fusion as the default when FTS and vector score normalization is reliable.

4. **Ranked/RRF fallback**
   Titan should also support rank-based fusion or reciprocal rank fusion when score scales are unreliable across datasets, models, or index versions.

5. **Field/property weighting**
   Weaviate lets keyword search emphasize specific properties. Titan should weight browser fields differently: title, URL/domain, headings, selected text, user notes, bookmark title, body chunks, and collection labels should not contribute equally.

6. **Explicit query vectors**
   Weaviate hybrid search can accept a supplied query vector. Titan should support user-provided or subsystem-provided query vectors internally for future multimodal features, imported embeddings, and advanced local models.

7. **Vector thresholds**
   Weaviate supports a maximum vector distance threshold. Titan should use semantic distance thresholds to prevent weak semantic matches from being included merely because lexical results were sparse.

8. **Explainable scores**
   Weaviate can return score/explain-score metadata. Titan should require diagnostics that show lexical score, vector distance/similarity, normalized scores, fusion method, alpha/semanticRatio, boosts, filters, and final score.

## Titan improvements over Weaviate

Weaviate is a vector database/server product. Titan can improve by being local and browser-aware.

Titan should improve by adding:

- local-only operation inside the user profile;
- no server dependency for baseline retrieval;
- privacy filters that execute before result fusion and before AI context assembly;
- private-browsing exclusion by default;
- browser deletion hooks so history clearing removes derived semantic artifacts;
- tab/workspace/session-aware ranking;
- result citations tied to local browser artifacts;
- no raw GraphQL/SQL/vector API exposure to web pages;
- strict AI context preview and approval before remote model use.

## What Titan emulates from Meilisearch

Meilisearch is the strongest reference for developer ergonomics and embedding lifecycle discipline.

Titan should emulate:

1. **Simple semanticRatio control**
   Meilisearch exposes `semanticRatio` to tune the balance between keyword and semantic results. Titan should adopt the concept and map it to readable product modes.

2. **Embedding models are not LLMs**
   Meilisearch correctly distinguishes embedding models from LLMs. Titan should preserve the same distinction: local embedding generation is search infrastructure, while LLM use is an optional assistant layer.

3. **Document templates**
   Meilisearch lets developers define which fields are embedded via a document template. Titan should require source-specific embedding templates so it does not blindly embed entire pages.

4. **Embedding cache discipline**
   Meilisearch stores embeddings and regenerates them only when relevant document content changes. Titan should key embeddings by content hash, model id, dimensions, template version, chunking version, locale, and privacy state.

5. **Controlled reindexing**
   Meilisearch warns that changing model, provider, template, dimensions, or pooling can force reindexing. Titan should have explicit reindex jobs and user-visible storage/performance controls for these changes.

6. **Small model preference**
   Meilisearch notes that in hybrid search, full-text already handles exact matches, so smaller embedding models are often sufficient. Titan should benchmark small local embeddings first, especially 384-dimensional models, before considering larger dimensions.

7. **User-provided embeddings**
   Meilisearch supports user-provided embeddings. Titan should allow internal user-provided/imported vectors later for images, PDFs, audio, external notes, and plugin-like local sources.

## Titan improvements over Meilisearch

Meilisearch optimizes a search server/product API. Titan can improve for browser-local AI by adding:

- per-source embedding templates for browser artifacts;
- local-only embedding providers as the default;
- remote embedding providers only after explicit consent;
- no API-key storage requirement for baseline semantic search;
- deletion propagation from browser actions;
- per-site and per-workspace embedding exclusion;
- hybrid retrieval diagnostics built for AI citation and context assembly, not only search tuning.

## Initial data model

The first implementation should use a separate Titan-owned profile database, not Firefox history/bookmark tables directly.

Suggested tables:

```text
titan_sources
  id
  source_type              -- page, note, bookmark, collection, tab_session, pdf, reader_view, selection
  canonical_url
  origin
  title
  workspace_id
  container_id
  profile_scope
  private_browsing_state
  created_at
  updated_at
  deleted_at
  privacy_state
  content_hash

titan_chunks
  id
  source_id
  chunk_index
  chunk_kind               -- title, heading, body, note, selection, summary, caption
  text
  text_hash
  language
  token_estimate
  created_at
  deleted_at

titan_chunk_fts
  FTS5 virtual table over chunk text and weighted searchable fields

titan_embedding_models
  id
  provider_type            -- local, remote, user_provided
  provider_name
  model_name
  dimensions
  distance_metric
  tokenizer_or_pooling
  license
  local_only_capable
  created_at

titan_embedding_templates
  id
  source_type
  template_name
  template_version
  template_body
  max_bytes
  created_at

titan_embeddings
  id
  chunk_id
  model_id
  template_id
  embedding_hash
  vector
  dimensions
  created_at
  stale_at
  deleted_at

titan_privacy_rules
  id
  scope_type               -- global, site, origin, workspace, source_type
  scope_value
  rule                     -- allow, exclude, warn, ephemeral
  created_at
  updated_at

titan_index_jobs
  id
  job_type                 -- index, reindex, delete, compact, verify
  status
  reason
  started_at
  finished_at
  error

titan_search_diagnostics
  id
  query_hash
  timestamp
  fusion_method
  semantic_ratio
  filters_applied
  result_count
  elapsed_ms
```

## Retrieval pipeline

Titan retrieval should be hybrid-first:

```text
1. Receive query from Titan UI or Titan Assistant.
2. Build a retrieval plan.
3. Apply hard privacy filters before search.
4. Run FTS5 lexical retrieval.
5. Run sqlite-vec semantic retrieval.
6. Normalize lexical and semantic scores.
7. Fuse results with relative score fusion by default.
8. Fall back to RRF/rank fusion when score scales are unreliable.
9. Apply browser-context boosts.
10. Apply semantic distance thresholds.
11. Return source chunks, scores, diagnostics, and citations.
12. Let Titan Assistant use only approved context.
```

## Fusion strategy

Titan should support at least two fusion strategies.

### Relative score fusion

Default when both result legs produce stable score ranges.

```text
keyword_score_norm = normalize(keyword_score)
semantic_score_norm = normalize(semantic_similarity)
final_score = ((1 - semanticRatio) * keyword_score_norm) + (semanticRatio * semantic_score_norm)
```

### Reciprocal rank fusion fallback

Use when score scales are unreliable or after index/model changes.

```text
rrf_score = Σ 1 / (k + rank_i)
```

Titan should log which fusion method was used and expose diagnostics in dev builds.

## Browser-context boosts

Boosts must run after hard privacy filters. Boosts must never resurrect excluded data.

Candidate boosts:

- current workspace;
- currently open tab;
- recently active tab;
- user-created note;
- explicit collection membership;
- bookmark;
- exact title match;
- exact domain match;
- selected text source;
- recent user interaction;
- pinned workspace item.

Candidate penalties:

- stale summary;
- low-confidence extraction;
- known duplicate chunk;
- obsolete embedding model;
- page content changed since embedding;
- source outside current workspace.

## Privacy and deletion rules

Privacy is part of the retrieval system, not a UI-only preference.

Required rules:

- private browsing content is not indexed;
- site/workspace exclusions apply before FTS and vector search;
- history deletion removes derived chunks, embeddings, summaries, and diagnostics where appropriate;
- clearing site data must remove Titan semantic artifacts for that site when linked to browsing data;
- remote AI providers cannot receive raw chunks unless the user consents;
- Titan Assistant must show which local sources it intends to use before remote context submission;
- diagnostics must avoid storing raw sensitive query text unless explicitly enabled in debug builds.

## Embedding templates

Titan should use explicit templates per source type.

Examples:

```text
page_chunk_v1:
  title: {{title}}
  site: {{origin}}
  heading: {{heading}}
  text: {{chunk_text}}

note_v1:
  note title: {{title}}
  note: {{body}}
  linked page: {{canonical_url}}

bookmark_v1:
  bookmark title: {{title}}
  url: {{canonical_url}}
  user description: {{description}}

selection_v1:
  page title: {{title}}
  selected text: {{selection_text}}
  surrounding heading: {{heading}}
```

Templates should be short, source-aware, versioned, and benchmarked. Changing a template version must mark affected embeddings stale and schedule controlled reindexing.

## Embedding model policy

Initial default:

- local embedding model;
- small dimensions first, likely 384 dimensions for the initial benchmark set;
- remote embedding providers disabled by default;
- model identity stored with every vector;
- embedding license recorded before bundling or recommending a model;
- user-visible storage and reindex controls.

Titan should evaluate larger 768/1024/1536+ dimensional models only after benchmarks prove better retrieval quality worth the memory, disk, and latency cost.

## API shape

Do not expose raw SQL or unrestricted vector operations to web content.

Preferred privileged API shape:

```ts
interface TitanSemanticIndex {
  indexSource(source: TitanIndexableSource): Promise<TitanIndexResult>;
  deleteSource(sourceId: string, reason: TitanDeletionReason): Promise<void>;
  search(query: TitanSearchQuery): Promise<TitanSearchResult[]>;
  explain(resultId: string): Promise<TitanSearchExplanation>;
  getStorageUsage(): Promise<TitanSemanticStorageUsage>;
  setPrivacyRule(rule: TitanPrivacyRule): Promise<void>;
  scheduleReindex(plan: TitanReindexPlan): Promise<TitanJobId>;
}
```

Forbidden for web content:

```ts
query(sql: string): Promise<any>;
rawVectorSearch(vector: Float32Array): Promise<any>;
readAllEmbeddings(): Promise<any>;
```

## Benchmark requirements

Before shipping semantic search broadly, benchmark at minimum:

- 1k chunks;
- 10k chunks;
- 50k chunks;
- 250k chunks;
- cold start time;
- warm query latency;
- indexing throughput;
- storage footprint;
- memory footprint;
- deletion/reindex throughput;
- FTS-only quality;
- vector-only quality;
- hybrid quality;
- relative score fusion vs RRF;
- 384 vs 768+ dimensions;
- local embedding latency;
- remote embedding latency only if remote providers are implemented.

## Implementation phases

### Phase S0: Architecture decision record

Write `docs/titan/adr/0001-local-semantic-index.md` comparing:

- SQLite + FTS5 + sqlite-vec;
- PGlite + pgvector;
- ObjectBox;
- external local service;
- pure WebExtension storage;
- Firefox storage-only approach.

### Phase S1: Prototype outside Gecko internals

Build a small isolated prototype before touching engine code. Validate schema, FTS search, vector search, hybrid fusion, privacy filters, and diagnostics.

### Phase S2: Privileged browser integration

Expose the semantic index to Titan-owned browser chrome only. Do not expose it to ordinary web pages.

### Phase S3: Assistant integration

Allow Titan Assistant to retrieve local context with source citations and privacy preview.

### Phase S4: Reindex/deletion hardening

Implement controlled reindexing, history deletion propagation, site exclusion handling, and storage compaction.

### Phase S5: Native/deeper integration decision

Only consider deeper C++/Rust/Gecko integration after prototype benchmarks and rebase-risk review.

## Open questions

- Which sqlite-vec distance metric and vector format should be used for the first prototype?
- Which local embedding model should be benchmarked first?
- Should the first prototype run as privileged JS, a native component, or a separate local helper process?
- How should Titan link browser history deletion events to semantic artifact deletion?
- Should diagnostics be always-on with redaction, or debug-only?
- What is the first source type: notes, bookmarks, reader-mode pages, or explicit user-saved pages?

## Summary

Titan should not clone ObjectBox, Weaviate, or Meilisearch. Titan should synthesize their best ideas into a browser-native local semantic layer:

```text
ObjectBox      -> embedded, disk-backed, ACID, update-friendly vector storage
Weaviate       -> hybrid retrieval, alpha/semanticRatio, fusion, score explanations, thresholds
Meilisearch    -> document templates, embedding cache discipline, simple tuning, small-model pragmatism
Titan improves -> browser privacy gates, deletion safety, workspace awareness, local-first AI context control
```
