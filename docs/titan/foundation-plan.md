# Titan foundation plan

This document defines the next foundation layer for Titan before product implementation begins. Titan should remain a product-layer browser fork first: build a safe identity, feature-flag system, local-first data model, privacy boundary, AI service boundary, and release discipline before modifying Gecko internals or shipping public binaries.

## Foundation principle

Titan's foundation should make future product work safer, not merely faster.

Every foundational change should be judged by whether it improves:

- upstream rebase safety;
- user data ownership;
- privacy and deletion correctness;
- local-first operation;
- feature isolation;
- build and release clarity;
- eventual WebKit companion compatibility.

If a change makes Firefox or Zen security updates harder to absorb, it should be delayed unless it is essential.

## Foundation pillars

### Pillar 1: Fork identity and upstream discipline

Titan needs a distinct product identity before public binaries exist.

Scope:

- product name;
- vendor string;
- app id;
- binary name;
- package metadata;
- issue/support/security URLs;
- profile path strategy;
- release artifact names;
- upstream attribution.

Rules:

- Keep upstream attribution to Zen and Firefox.
- Remove ambiguity about whether a build is official Zen or Titan.
- Do not point public Titan builds at Zen update infrastructure.
- Do not change generated branding blindly without build verification.

Deliverables:

- branding/identity checklist;
- package metadata cleanup;
- Surfer metadata plan;
- profile-path decision;
- release/update ownership plan.

### Pillar 2: Non-publishing CI and build validation

Titan should have CI that validates repository health without publishing binaries.

Scope:

- dependency install;
- lint;
- tests;
- license checks;
- UI build verification;
- docs checks where useful;
- workflow quarantine for inherited release jobs.

Baseline checks:

```text
npm ci
npm run lint
npm run test
npm run lc
npm run build:ui
```

Rules:

- CI must not require Zen-owned secrets for basic validation.
- CI must not publish update archives or releases.
- Full browser builds should remain explicit/manual until infrastructure costs and signing requirements are understood.

Deliverables:

- non-publishing Titan CI workflow;
- inherited workflow audit;
- release workflow quarantine notes;
- local build instructions.

### Pillar 3: Titan feature flags and preferences

Titan-specific features should be gated before implementation.

Scope:

- `titan.*` preference namespace;
- feature flags for experimental systems;
- safe defaults;
- debug diagnostics flags;
- migration behavior for changed preferences.

Suggested preference families:

```text
titan.enabled
titan.ui.*
titan.localFirst.*
titan.semantic.*
titan.ai.*
titan.privacy.*
titan.diagnostics.*
titan.experimental.*
```

Rules:

- Do not hide Titan behavior behind ambiguous Zen preferences unless intentionally extending existing Zen behavior.
- Experimental features default off until validation exists.
- Privacy-sensitive features need explicit, reviewable defaults.

Deliverables:

- Titan preference namespace document;
- feature flag naming rules;
- default preference policy;
- debug/diagnostic preference policy.

### Pillar 4: Product shell surfaces

Before local AI features exist, Titan needs stable surfaces where they will live.

Scope:

- Titan sidebar shell;
- assistant panel placeholder;
- command palette namespace;
- settings section;
- workspace terminology map;
- onboarding surface;
- diagnostics/about page.

Rules:

- Product shell should not require semantic indexing or AI providers to exist.
- UI work should be feature-flagged.
- Shell surfaces should be rebase-friendly and avoid unnecessary engine changes.

Deliverables:

- UI surface inventory;
- first shell implementation plan;
- keyboard command map;
- settings map;
- accessibility requirements.

### Pillar 5: Local-first data foundation

Titan needs a clear local data model before AI features start writing persistent memory.

Scope:

- notes;
- collections;
- reading queue;
- annotations;
- tab sessions;
- semantic chunks;
- embeddings;
- summaries;
- assistant interactions;
- deletion logs;
- export/import.

Rules:

- Store Titan-owned data separately from Firefox/Zen internals at first.
- Use explicit schema versions and deterministic migrations.
- Every AI-derived artifact needs a source link and deletion path.
- Export must be possible without cloud sync.

Deliverables:

- local data schema ADR;
- migration policy;
- export/import policy;
- deletion propagation policy;
- backup/restore policy.

### Pillar 6: Local semantic retrieval

The current planned direction is SQLite + FTS5 + sqlite-vec.

Scope:

- source metadata;
- chunks;
- FTS5 lexical index;
- sqlite-vec vector index;
- embedding model metadata;
- embedding templates;
- privacy filters;
- search diagnostics;
- hybrid ranking.

Rules:

- Hybrid retrieval is the default.
- Privacy filters run before retrieval and context assembly.
- No raw SQL/vector APIs to web content.
- Embeddings are versioned by model, template, content hash, dimensions, and chunking strategy.

Deliverables:

- local semantic architecture document;
- semantic index ADR;
- benchmark plan;
- prototype plan outside Gecko internals.

### Pillar 7: AI provider boundary

Titan's AI layer should be a provider abstraction, not hardcoded calls to one model or service.

Scope:

- local embeddings;
- optional remote embeddings;
- local assistant model;
- optional remote assistant provider;
- prompt registry;
- context builder;
- privacy gate;
- provenance/citations.

Suggested interfaces:

```text
TitanAIProvider
TitanEmbeddingProvider
TitanLocalModelProvider
TitanRemoteModelProvider
TitanPromptRegistry
TitanContextBuilder
TitanPrivacyGate
TitanModelRegistry
```

Rules:

- Local-only mode must be real.
- Remote model use must be explicit.
- Context sources must be inspectable.
- AI outputs must never be treated as browser security decisions.
- Provider failures should degrade gracefully.

Deliverables:

- AI provider boundary document;
- local-only mode policy;
- remote-provider consent policy;
- prompt/versioning policy;
- context provenance model.

### Pillar 8: Privacy, deletion, and trust boundaries

Privacy should be part of Titan's architecture, not only UI preferences.

Scope:

- private browsing exclusion;
- site exclusion;
- workspace exclusion;
- sensitive-site handling;
- history deletion propagation;
- clearing site data;
- AI context preview;
- diagnostics redaction;
- local-only enforcement.

Rules:

- Excluded data must not be indexed.
- Deleted data must not be retrievable through derived artifacts.
- Diagnostics must not become a hidden sensitive-data store.
- Remote context submission requires explicit consent.

Deliverables:

- privacy boundary document;
- deletion propagation lifecycle;
- sensitive-site rules;
- diagnostics redaction policy;
- AI context approval UX requirements.

### Pillar 9: Security architecture

Titan should not expand the browser attack surface without clear boundaries.

Scope:

- privileged APIs;
- web content isolation;
- extension exposure;
- local database access;
- native helpers;
- model files;
- downloads;
- update channels;
- secrets/API keys.

Rules:

- Do not expose raw SQL to web content.
- Do not expose arbitrary local file or embedding access.
- Do not let normal websites trigger background indexing without user-level policy.
- Treat bundled model files and local helpers as supply-chain-sensitive artifacts.
- Keep secrets out of profile exports unless explicitly designed and encrypted.

Deliverables:

- threat model;
- privileged API policy;
- extension exposure policy;
- model supply-chain policy;
- secure storage decision.

### Pillar 10: WebKit companion constraints

Titan's desktop fork should not try to embed WebKit into Gecko. WebKit support should be a future companion target with shared concepts, not shared engine code.

Scope:

- shared data model;
- shared export/import format;
- shared AI provider contracts;
- shared privacy rules;
- shared design language;
- platform-specific storage implementation.

Rules:

- Do not bind Titan's core data model to Gecko-only APIs.
- Keep semantic architecture conceptually portable.
- Treat WebKit companion as a later product, not a blocker for desktop foundation.

Deliverables:

- WebKit companion constraints document;
- cross-platform data contract notes;
- export/import compatibility plan.

## Recommended documentation sequence

The next docs should land in this order:

1. `docs/titan/foundation-plan.md` — this document.
2. `docs/titan/preferences-and-feature-flags.md` — Titan preference namespace and feature flag policy.
3. `docs/titan/privacy-and-deletion-boundary.md` — privacy gates, deletion propagation, and AI context approval.
4. `docs/titan/ai-provider-boundary.md` — local/remote providers, prompt registry, context builder, model registry.
5. `docs/titan/security-threat-model.md` — privileged API, model supply chain, local DB, web content isolation.
6. `docs/titan/product-shell-plan.md` — sidebar, assistant panel, settings, command palette, diagnostics.
7. `docs/titan/build-and-ci-plan.md` — non-publishing CI, workflow quarantine, build verification.
8. `docs/titan/adr/0001-local-semantic-index.md` — formal storage/search ADR.

## Implementation sequence after documentation

1. Non-publishing CI.
2. Package metadata cleanup.
3. Titan preference namespace.
4. Product shell placeholders.
5. Local data prototype outside Gecko internals.
6. Semantic index prototype outside Gecko internals.
7. Privacy/deletion tests.
8. AI provider boundary prototype.
9. Assistant UI integration.
10. Deeper native integration review only after benchmarks.

## Immediate next step

Document the Titan preference and feature flag system. This should come before product shell or AI work so every experimental feature has a clear gate, default, and rollback path.
