# Titan Browser

Titan is the codename for this fork of Zen Browser. The near-term goal is to evaluate whether Zen is the right upstream base for a UI/UX-first, local-first, AI-native browser without prematurely taking on browser-engine maintenance risk.

Titan should be treated as a product-layer fork first:

- keep Gecko/Firefox engine changes minimal;
- keep Zen upstream rebases routine and auditable;
- isolate Titan-specific behavior behind `titan.*` preferences and Titan-owned modules;
- avoid release/update infrastructure changes until signing, hosting, and security-update ownership are ready;
- design local-first and AI features as privacy-preserving browser-layer capabilities, not silent data-upload features.

## Phase 0 documents

- [`upstream-strategy.md`](./upstream-strategy.md) — how Titan should track Firefox and Zen safely.
- [`fork-hygiene-audit.md`](./fork-hygiene-audit.md) — current Zen identity, release, and metadata surfaces that must be handled before shipping Titan builds.
- [`release-safety.md`](./release-safety.md) — release/update guardrails so Titan does not accidentally publish Zen-branded or unsafe artifacts.
- [`architecture-roadmap.md`](./architecture-roadmap.md) — staged plan for UI/UX, local-first, AI, and future WebKit companion work.
- [`foundation-plan.md`](./foundation-plan.md) — foundation pillars for Titan identity, CI, feature flags, product shell, local-first data, privacy, AI, security, and WebKit constraints.
- [`extension-compatibility-plan.md`](./extension-compatibility-plan.md) — Firefox add-on baseline support, Chrome extension compatibility limits, Chrome Web Store stance, MV2/MV3 notes, and Titan-specific API constraints.
- [`local-first-semantic-architecture.md`](./local-first-semantic-architecture.md) — SQLite + FTS5 + sqlite-vec plan, including what Titan emulates and improves from ObjectBox, Weaviate, and Meilisearch.
- [`upstream-rebase-log.md`](./upstream-rebase-log.md) — template for tracking future upstream syncs.

## Current status

This repository is still structurally a Zen Browser fork. Do not treat the current `dev` branch as a Titan-branded browser release until the fork hygiene checklist and release safety checklist are complete.
