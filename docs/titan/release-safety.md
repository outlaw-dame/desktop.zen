# Titan release safety

Titan is not ready to publish binaries until release, update, signing, and security ownership are separated from Zen. This file defines the release guardrails that should block accidental unsafe releases.

## Current risk

The inherited release workflow is designed for Zen Browser. Before Titan has its own release process, running inherited release jobs may produce artifacts that are incorrectly named, incorrectly branded, or configured to use Zen update infrastructure.

## Release blockers

A Titan release must be blocked if any of the following are true:

- the build still uses Zen app identity for public distribution;
- the update host points at Zen infrastructure;
- release archives point at `zen-browser/desktop`;
- artifact names imply official Zen Browser builds;
- signing/notarization ownership is unclear;
- the fork is behind a critical Firefox or Zen security fix;
- release notes do not identify the Firefox and Zen base versions;
- no source archive or source access path is provided for MPL-covered modifications;
- crash/update/telemetry endpoints are not intentionally configured;
- CI requires Zen-private secrets that Titan does not own.

## Safe workflow policy during Phase 0 and Phase 1

Allowed:

- documentation-only PRs;
- metadata audit PRs;
- build smoke-test workflow edits that do not publish releases;
- local build documentation;
- non-publishing CI validation.

Restricted:

- public binary releases;
- auto-update configuration changes;
- installer/notarization/signing changes;
- artifact publishing;
- changing app id/profile paths without a rollback plan;
- enabling any workflow that deploys to non-Titan infrastructure.

## Required release checklist

Before the first public Titan build:

- [ ] CI can run without Zen-owned deploy secrets.
- [ ] Titan has its own artifact names.
- [ ] Titan has its own update-host decision: disabled, self-hosted, or explicitly documented.
- [ ] Titan has its own signing/notarization plan.
- [ ] Titan has its own security policy.
- [ ] Titan has a published upstream-base statement.
- [ ] Titan has a documented emergency update process.
- [ ] Release workflow is manual and reviewed.
- [ ] Release workflow cannot accidentally publish from the wrong branch.
- [ ] Release build is reproducible enough to document inputs and source refs.

## Recommended early action

Create a separate non-publishing CI workflow for Titan that only checks repository health. Keep inherited Zen release workflows manual until Titan owns the infrastructure.

Suggested early checks:

```text
npm ci
npm run lint
npm run test
npm run lc
```

For full browser builds, prefer explicit manual workflow dispatch until build time, cache requirements, and infrastructure requirements are understood.
