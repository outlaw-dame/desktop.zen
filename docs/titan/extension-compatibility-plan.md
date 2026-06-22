# Titan extension compatibility plan

Titan currently inherits extension behavior from its Zen/Firefox base. This means Firefox WebExtensions support should be treated as inherited capability, while Chrome extension support should be treated as compatibility-by-overlap, not as a guaranteed Chrome-equivalent platform.

This document defines Titan's extension and plugin stance before any product work exposes Titan-specific APIs.

## Terminology

Use precise language:

- **Firefox add-ons**: extensions distributed through Mozilla Add-ons or side-loaded using Firefox-compatible WebExtension APIs.
- **Chrome extensions**: extensions built for Chromium/Chrome WebExtensions APIs and usually distributed through the Chrome Web Store.
- **WebExtensions**: the cross-browser extension model shared in part by Firefox, Chrome, Edge, and other browsers.
- **Legacy plugins**: old NPAPI-style browser plugins. These are not part of Titan's target platform.
- **Titan APIs**: future privileged extension APIs or browser-chrome APIs created specifically by Titan.

## Current inherited state

The fork is Firefox-based through Zen. The repository's `surfer.json` identifies the product as Firefox-derived, and the build scripts use Surfer to download/import/build Firefox-derived source.

Known current state:

- Firefox WebExtensions support is inherited from Firefox/Zen.
- The fork does not currently declare Titan-specific extension APIs.
- The fork does not currently configure bundled Titan add-ons in `surfer.json`.
- The fork does not currently show first-class Chrome Web Store installation support.
- Chrome extension compatibility is limited to the API/manifest overlap that Firefox supports.

## Product stance

Titan should support Firefox add-ons first.

Titan may support Chrome-extension compatibility where Firefox already supports the relevant APIs, but Titan should not claim full Chrome extension compatibility unless it implements, tests, and documents that explicitly.

Titan should not support legacy browser plugins.

## Firefox add-ons

Firefox add-ons should be the baseline extension ecosystem for Titan.

Goals:

- preserve compatibility with Firefox-compatible WebExtensions;
- preserve user expectations around extension permissions;
- avoid breaking existing Firefox add-ons through Titan UI changes;
- document any Titan-specific incompatibilities introduced by sidebar/workspace/AI features.

Requirements:

- Titan UI shell changes must account for extension toolbar buttons and panels;
- sidebar/product shell work must not silently hide extension surfaces;
- extension permissions must remain understandable and user-controlled;
- Titan-specific data should not be exposed to extensions by default.

## Chrome extension compatibility

Chrome extension support should be described carefully.

Titan can likely run some Chrome-oriented extensions when they only use APIs Firefox supports. However, Chrome and Firefox still differ in WebExtension APIs, manifest support, service worker behavior, store policy, signing, permissions, and browser-specific namespaces.

Titan should therefore use the following wording until proven otherwise:

```text
Titan supports Firefox WebExtensions and may run many Chrome-compatible WebExtensions where Firefox compatibility exists. Titan does not currently guarantee full Chrome extension or Chrome Web Store compatibility.
```

## Chrome Web Store

Titan should not advertise Chrome Web Store support until it has an explicit implementation and security review.

Open questions:

- Should Titan allow direct install from the Chrome Web Store?
- Would direct install violate store terms, expectations, or signing assumptions?
- How should Titan handle extension updates from a non-Mozilla source?
- How should Titan warn users about extensions built and reviewed for a different browser?
- Should Chrome Web Store support be disabled by default even if technically possible?

Initial recommendation:

- Do not implement Chrome Web Store install support in the foundation phase.
- Revisit only after Titan has stable identity, release signing, extension threat model, and compatibility tests.

## Manifest V2 and Manifest V3

Titan inherits Firefox's WebExtension platform behavior unless it deliberately diverges.

Rules:

- Do not make Titan-specific MV2/MV3 promises until the current Firefox base behavior is verified.
- Track Firefox's MV3 implementation and limitations rather than Chrome's policy alone.
- Avoid depending on Chrome-only MV3 behavior for Titan features.
- Document which extension APIs are required by Titan's own optional extensions, if any.

## Titan-specific extension APIs

Titan should be conservative about exposing custom APIs.

Potential future Titan APIs:

- workspace metadata;
- command palette hooks;
- local collections/reading queue integration;
- assistant context contribution;
- semantic search over user-approved extension-provided data;
- sidebar surface integration.

Forbidden without a dedicated security review:

- raw SQL access;
- raw vector database access;
- unrestricted embedding reads;
- unrestricted browsing history export;
- silent access to Titan notes, summaries, or assistant memory;
- arbitrary local model file access;
- APIs that let web content or extensions bypass privacy exclusions.

Preferred API shape:

- narrow;
- permissioned;
- revocable;
- auditable;
- source-scoped;
- user-visible;
- local-first by default.

## Extension access to Titan local-first data

Titan local-first data should not automatically become extension-readable.

Default policy:

- Titan notes are private to Titan unless shared explicitly.
- Titan semantic index is not extension-readable by default.
- Titan AI summaries are not extension-readable by default.
- Titan browsing-derived chunks and embeddings are never exposed directly.
- Extensions can request narrow capabilities only after a permission model exists.

Future possible permission examples:

```text
titanCollections.read
titanCollections.write
titanReadingQueue.read
titanReadingQueue.write
titanWorkspace.current
titanAssistant.contextContribute
titanSemantic.searchUserApproved
```

Do not implement these names until a permissions and threat-model document exists.

## Extension and AI interaction

Extensions must not silently feed browsing data into Titan AI or remote model providers.

Rules:

- Extension-provided AI context must be labeled as extension-provided.
- Extension access to Titan Assistant must be user-approved.
- Remote AI calls triggered through extension surfaces must pass Titan's privacy gate.
- Extensions must not receive hidden Titan Assistant context unless the user explicitly shares it.

## Security requirements

Before adding Titan-specific extension APIs, document:

- threat model;
- permission prompts;
- revocation behavior;
- audit/debug visibility;
- extension data retention;
- extension interaction with private browsing;
- extension interaction with site/workspace exclusions;
- update and signing expectations;
- compatibility test matrix.

## Compatibility test matrix

Titan should eventually test:

- Firefox add-ons from Mozilla Add-ons;
- side-loaded Firefox WebExtensions;
- common Chrome-compatible WebExtensions that avoid Chrome-only APIs;
- toolbar button placement;
- sidebar/panel behavior;
- content scripts;
- background scripts/service workers;
- MV2 and MV3 behavior in the current Firefox base;
- extension permissions UI;
- private browsing extension behavior;
- interactions with Titan workspaces and sidebars.

## Foundation phase decision

For the foundation phase:

1. Treat Firefox add-ons as inherited baseline support.
2. Do not advertise full Chrome extension support.
3. Do not implement Chrome Web Store installation.
4. Do not expose Titan-specific APIs yet.
5. Add a future threat model before any Titan extension API ships.
6. Ensure Titan UI shell planning accounts for extension toolbar/panel/sidebar surfaces.

## Summary

Titan's extension position is:

```text
Baseline: Firefox WebExtensions inherited from Zen/Firefox.
Compatibility: some Chrome-oriented extensions may work through Firefox's WebExtension compatibility layer.
Not guaranteed: full Chrome extension or Chrome Web Store support.
Not supported: legacy plugins.
Future work: Titan-specific extension APIs only after privacy/security review.
```
