# Titan architecture roadmap

Titan should evolve as a browser product layer before it becomes a browser engine project. The safest architecture is to keep Gecko/Firefox internals as close to upstream as possible and place Titan features in browser chrome, privileged modules, WebExtension-compatible surfaces, local services, and clearly namespaced product code.

## North star

Titan is a local-first, AI-native, privacy-respecting productivity browser built from a Zen/Firefox base.

Core product themes:

- fast, opinionated UI/UX;
- workspace-centered browsing;
- local-first notes, collections, summaries, and research trails;
- AI features that disclose context and respect local-only modes;
- careful long-term path to a WebKit companion target without forcing WebKit into the Gecko desktop fork.

## Phase 1: Titan identity and fork hygiene

Goals:

- distinct product metadata;
- distinct package metadata;
- distinct app id and binary name;
- distinct issue/security URLs;
- documented upstream attribution;
- non-publishing CI guardrails.

Avoid deep product feature work until this is complete.

## Phase 2: Build and rebase validation

Goals:

- local build instructions that work on at least one platform;
- clear Surfer import/build/package notes;
- upstream rebase log;
- non-release CI checks;
- release workflows quarantined or clearly documented.

Exit criteria:

- Titan can rebase onto Zen and still run basic validation.
- The project can identify when Firefox/Zen security updates require action.

## Phase 3: Titan product shell

Initial product-layer surfaces:

- Titan sidebar placeholder;
- Titan command palette namespace;
- Titan settings page/section;
- Titan workspace terminology map;
- Titan onboarding surface;
- Titan feature flags behind `titan.*` prefs.

Do not require AI or local-first storage yet. Build the surfaces where those features will live.

## Phase 4: Local-first data foundation

Initial Titan-owned entities:

```text
Workspace
TabSession
Collection
ReadingItem
Annotation
Note
PageSummary
EmbeddingRecord
ResearchTrail
SavedPrompt
AIInteraction
```

Requirements:

- local by default;
- exportable;
- versioned schema;
- migration tests;
- no silent remote sync;
- user-readable export format where practical;
- encryption-at-rest decision documented before sensitive AI memory ships.

## Phase 5: AI provider abstraction

Define AI as a service boundary, not a pile of feature-specific calls.

Suggested interfaces:

```text
TitanAIProvider
TitanLocalModelProvider
TitanRemoteModelProvider
TitanEmbeddingProvider
TitanPrivacyGate
TitanPromptRegistry
TitanContextBuilder
```

Early feature candidates:

- summarize current page;
- answer questions over current page;
- summarize selected text;
- cluster open tabs;
- name workspaces;
- identify duplicate tabs;
- summarize a session;
- local semantic search over Titan notes/collections/summaries.

Privacy requirements:

- show the context used;
- require clear consent before remote model use;
- provide real local-only mode;
- allow per-site and per-workspace AI exclusions;
- do not embed or summarize sensitive sites silently.

## Phase 6: Assistant UX

Assistant surfaces:

- sidebar assistant;
- page assistant;
- selection assistant;
- workspace assistant;
- command palette assistant;
- research-trail assistant.

Assistant behavior:

- explain what sources/context were used;
- store user-approved outputs locally;
- keep generated summaries editable/deletable;
- never treat AI output as browser security truth;
- avoid manipulating settings, credentials, downloads, or permissions without explicit user action.

## Phase 7: WebKit companion planning

Do not attempt to embed WebKit as a second engine inside the Zen/Firefox desktop fork.

Instead, plan a future companion target:

```text
Titan WebKit companion = Swift/WebKit shell + shared Titan data and AI contracts
```

Shared concepts:

- local-first schema;
- collections and reading queue;
- summaries and annotations;
- AI provider protocol;
- privacy rules;
- design language;
- export/import format.

Not shared:

- Gecko internals;
- Zen chrome internals;
- Firefox-specific extension internals;
- desktop release/update machinery.

## Phase 8: Release engineering

Release engineering becomes its own phase only after Titan identity, security process, and build validation exist.

Required decisions:

- signing;
- notarization;
- update hosting;
- crash reporting;
- telemetry policy;
- SBOM generation;
- release channels;
- rollback process;
- vulnerability disclosure process.

## Architectural principle

Titan should remain easy to rebase. Every major feature should be judged by whether it increases or decreases the long-term ability to track Firefox and Zen security updates.
