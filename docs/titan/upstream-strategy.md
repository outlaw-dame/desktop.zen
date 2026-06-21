# Titan upstream strategy

Titan currently inherits from Zen Browser, which itself inherits from Firefox. That gives Titan a fast UX starting point, but it also creates a two-hop upstream chain:

```text
Firefox -> Zen Browser -> Titan
```

This is acceptable only if Titan keeps its own changes small, well-documented, and easy to rebase.

## Non-negotiable rules

1. **Security updates outrank product features.**
   Titan must not ship public builds while knowingly behind critical Firefox or Zen security fixes.

2. **Prefer browser-chrome and product-layer changes over engine changes.**
   Avoid Gecko, SpiderMonkey, networking, sandbox, media, TLS, certificate, profile, and updater internals unless there is a clear security-reviewed reason.

3. **Keep Titan changes namespaced.**
   Prefer `titan.*` preferences, Titan-owned modules, and clearly named files. Avoid modifying existing Zen code paths without leaving comments or documentation explaining why.

4. **Rebase frequently.**
   Long-lived divergence from Zen increases merge conflicts and security risk.

5. **Keep an escape hatch.**
   If Zen’s own patch set becomes a blocker, Titan should be able to reassess and move closer to direct Firefox upstream.

## Branch model

Recommended branch layout:

```text
dev                         # current default branch inherited from Zen
titan/main                  # future reviewed Titan integration branch
upstream/zen-dev            # local mirror of Zen dev when using a local clone
titan/upstream-sync-*       # dedicated upstream sync branches
titan/phase-*               # focused Titan work branches
titan/feature-*             # product feature branches
```

Until the project has validated build and release behavior, Titan work should land through focused PRs rather than direct commits to `dev`. Upstream syncs should be performed on dedicated `titan/upstream-sync-*` branches, verified locally, recorded in the rebase log, and merged into the integration branch only through a reviewed pull request.

## Upstream sync workflow

Use a structured workflow for every Zen/Firefox sync:

1. Create a dedicated upstream sync branch from the current Titan integration branch.
2. Fetch the selected Zen upstream ref.
3. Apply the upstream sync with the chosen strategy: rebase, merge, or Surfer-driven update.
4. Resolve conflicts with minimal Titan-specific changes.
5. Update `docs/titan/upstream-rebase-log.md` with base refs, versions, conflicts, validation commands, and security-review status.
6. Run local validation before opening the PR.
7. Open a reviewed PR back into the Titan integration branch.
8. Do not mix product features into upstream sync PRs.

Recommended local validation for an upstream sync:

```text
npm ci
npm run lint
npm run test
npm run lc
npm run build:ui
```

Full browser build validation should be run when infrastructure and time permit, especially before any public build.

## Rebase log requirement

Every upstream sync should add or update an entry in `docs/titan/upstream-rebase-log.md` with:

- date;
- previous Titan base;
- new Zen commit/ref;
- Firefox version before and after;
- Surfer version before and after;
- conflicts encountered;
- files changed by Titan during conflict resolution;
- build/test commands run;
- security advisory review status.

## First upstream questions to answer

- How far is this fork behind current `zen-browser/desktop`?
- Which branch should Titan track: Zen `dev`, Zen stable/release, or a pinned release branch?
- How often can Titan realistically rebase?
- Can GitHub Actions build Titan without Zen’s private deploy/self-hosted secrets?
- Which release workflows should be disabled until Titan has its own signing and update infrastructure?

## Initial recommendation

Track Zen `dev` during early experimentation, but do not ship public binaries from this fork until Titan has its own branding, signing, update host, artifact names, release notes, and security process.
