# Titan preferences and feature flags

Titan-specific behavior should be controlled by a clear preference namespace before product implementation begins. This document defines the initial `titan.*` preference strategy, feature-flag lifecycle, privacy-sensitive defaults, and rollout rules.

## Goals

- Keep Titan-specific behavior separate from inherited Zen and Firefox behavior.
- Make experimental features easy to disable, test, and roll back.
- Avoid shipping hidden AI, indexing, privacy, or extension behavior without explicit gates.
- Keep defaults conservative until the feature has tests, documentation, and a deletion/privacy story.
- Give future WebKit companion work a stable conceptual settings model without binding it to Gecko preference internals.

## Non-goals

- Do not rename or replace every inherited Zen/Firefox preference during the foundation phase.
- Do not hide Titan behavior behind unrelated `zen.*`, `browser.*`, or `extensions.*` preferences.
- Do not expose every internal debug flag in user-facing settings.
- Do not make cloud or remote AI providers default-on.

## Namespace

All Titan-owned preferences should live under:

```text
titan.*
```

Suggested top-level families:

```text
titan.enabled
titan.ui.*
titan.shell.*
titan.workspace.*
titan.localFirst.*
titan.semantic.*
titan.ai.*
titan.privacy.*
titan.extensions.*
titan.diagnostics.*
titan.experimental.*
```

Use lower camel case after the family prefix.

Examples:

```text
titan.enabled
titan.ui.sidebar.enabled
titan.shell.commandPalette.enabled
titan.workspace.enabled
titan.localFirst.enabled
titan.semantic.enabled
titan.semantic.indexing.enabled
titan.semantic.hybridSearch.enabled
titan.semantic.remoteEmbeddings.enabled
titan.ai.enabled
titan.ai.localOnly.enabled
titan.ai.remoteProviders.enabled
titan.privacy.sensitiveSiteWarnings.enabled
titan.extensions.titanApis.enabled
titan.diagnostics.searchExplanations.enabled
titan.experimental.productShell.enabled
```

## Default policy

Default values should follow this policy:

| Feature category | Default before validation | Reason |
| --- | --- | --- |
| Branding and docs-only behavior | on/neutral | No user data risk |
| UI placeholders | off or dev-only | Avoid broken visible surfaces |
| Product shell experiments | off | Rebase and UX risk |
| Local-first storage | off | Migration/deletion risk |
| Semantic indexing | off | Sensitive browsing-derived data risk |
| Local embeddings | off until benchmarked | CPU/storage/performance risk |
| Remote embeddings | off | Privacy/network risk |
| Remote assistant providers | off | Privacy/API-key risk |
| Titan-specific extension APIs | off | Security/permission risk |
| Diagnostics with raw data | off | Sensitive data risk |
| Redacted diagnostics | dev-only until reviewed | Hidden data-retention risk |

## Feature-flag lifecycle

Each feature flag should move through explicit states.

```text
planned -> documented -> behind-pref -> dev-only -> nightly/experimental -> beta -> default-on -> stable -> cleanup/remove-pref
```

A feature should not advance unless it has:

- owner or tracking issue;
- user-data impact assessment;
- privacy/deletion behavior;
- rollback plan;
- test plan;
- default-value decision;
- release-note decision.

## Preference naming rules

Use names that describe behavior, not implementation accidents.

Good:

```text
titan.semantic.indexing.enabled
titan.ai.localOnly.enabled
titan.privacy.siteExclusions.enabled
titan.shell.commandPalette.enabled
```

Avoid:

```text
titan.useNewThing
titan.sqliteHack
titan.tmpAi
titan.foo
titan.enable2
```

Rules:

- Use `.enabled` for boolean feature gates.
- Use `.mode` for constrained string modes.
- Use `.max*` for numeric limits.
- Use `.debug.*` only under `titan.diagnostics.*`.
- Avoid embedding provider names in permanent generic preference names unless the provider is truly part of the public model.
- Do not reuse a preference for a different behavior after release.

## Suggested initial preferences

### Global

```text
titan.enabled = false
```

Master kill switch for Titan-only product behavior. This should not disable inherited Zen/Firefox baseline functionality.

### Product shell

```text
titan.ui.sidebar.enabled = false
titan.shell.commandPalette.enabled = false
titan.shell.assistantPanel.enabled = false
titan.workspace.enabled = false
```

These gates allow shell work to land without requiring AI or semantic storage.

### Local-first data

```text
titan.localFirst.enabled = false
titan.localFirst.database.enabled = false
titan.localFirst.export.enabled = false
titan.localFirst.migrations.strict = true
```

Local-first storage should remain gated until schema, migration, export/import, and deletion tests exist.

### Semantic retrieval

```text
titan.semantic.enabled = false
titan.semantic.indexing.enabled = false
titan.semantic.hybridSearch.enabled = false
titan.semantic.fts.enabled = false
titan.semantic.vector.enabled = false
titan.semantic.sqliteVec.enabled = false
titan.semantic.searchExplanations.enabled = false
```

Semantic features should be independently gated so FTS-only prototypes can run before vector indexing, and vector search can be benchmarked before assistant use.

### AI providers

```text
titan.ai.enabled = false
titan.ai.localOnly.enabled = true
titan.ai.remoteProviders.enabled = false
titan.ai.contextPreview.required = true
titan.ai.storeInteractions.enabled = false
```

Remote providers must require explicit user consent. Local-only mode must be meaningful, not cosmetic.

### Privacy

```text
titan.privacy.siteExclusions.enabled = true
titan.privacy.workspaceExclusions.enabled = true
titan.privacy.privateBrowsingIndexing.enabled = false
titan.privacy.sensitiveSiteWarnings.enabled = true
titan.privacy.remoteContextApproval.required = true
```

Privacy gates default conservative. Private browsing indexing should remain false.

### Extensions

```text
titan.extensions.titanApis.enabled = false
titan.extensions.semanticSearchApi.enabled = false
titan.extensions.assistantContextApi.enabled = false
titan.extensions.chromeStoreInstall.enabled = false
```

Titan-specific extension APIs and Chrome Web Store install support should stay off until a security review and compatibility test matrix exist.

### Diagnostics

```text
titan.diagnostics.enabled = false
titan.diagnostics.redacted.enabled = true
titan.diagnostics.rawQueries.enabled = false
titan.diagnostics.searchExplanations.enabled = false
titan.diagnostics.performanceMarks.enabled = false
```

Diagnostics should avoid storing raw queries, raw chunks, embeddings, or assistant context unless explicitly enabled in a development environment.

## Privacy-sensitive preference rules

Preferences that can expose, persist, upload, or infer from user browsing data require stricter handling.

Sensitive categories:

- semantic indexing;
- embeddings;
- summaries;
- assistant memory;
- remote model providers;
- extension access to Titan data;
- diagnostics containing raw queries or page chunks;
- history-derived artifacts;
- private browsing behavior.

Rules:

- Default off unless the feature is a protective control.
- Require documentation before implementation.
- Require deletion path before indexing or persistence.
- Require UI copy before remote submission.
- Require tests for exclusion/deletion behavior.

## Rollback policy

Every Titan feature flag should have a rollback plan.

A rollback plan should state:

- what disabling the preference does;
- whether existing local data remains, is hidden, or is deleted;
- whether migrations can be reversed;
- whether background jobs stop immediately;
- whether UI surfaces disappear or show a disabled state;
- whether remote providers are revoked or merely disabled.

For semantic/AI features, disabling the feature should stop new indexing and retrieval immediately. Deleting existing artifacts should be a separate user-visible action unless the data is unsafe or corrupt.

## Settings UI policy

Not every preference needs a visible settings control.

User-facing settings should exist for:

- enabling/disabling AI features;
- local-only mode;
- remote providers;
- site/workspace exclusions;
- storage usage and deletion;
- semantic search opt-in/out;
- extension access to Titan APIs;
- diagnostics/export controls.

Developer-only preferences may remain hidden if they do not change privacy-sensitive behavior silently.

## Testing requirements

Preference behavior should be testable.

Tests should verify:

- default values;
- feature disabled means no UI/action/background job starts;
- privacy defaults are conservative;
- remote provider gates are respected;
- private browsing indexing remains disabled;
- Titan-specific extension APIs remain unavailable when disabled;
- diagnostics do not store raw sensitive data by default;
- toggling a feature does not corrupt local data.

## Documentation requirements

Each implemented preference should eventually have an entry with:

```text
name:
type:
default:
scope:
user visible:
feature owner:
privacy impact:
data written:
rollback behavior:
removal criteria:
```

## Initial implementation recommendation

Add only the minimum preference scaffolding first:

```text
titan.enabled
titan.experimental.productShell.enabled
titan.diagnostics.enabled
```

Then add feature-specific preferences as concrete implementations land. Avoid creating a large unused preference surface before the code exists.

## Summary

Titan preferences should make product work safer:

```text
titan.* namespace
+ conservative defaults
+ local-only and privacy-first gates
+ explicit AI/semantic/extension controls
+ rollback plans
+ no hidden remote or indexing behavior
```
