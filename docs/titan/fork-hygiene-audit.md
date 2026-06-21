# Titan fork hygiene audit

This file tracks surfaces that still identify this fork as Zen Browser or that still point at Zen-owned infrastructure. These items are not automatically wrong during early development, but they must be explicitly handled before Titan builds are distributed.

## Current known inherited Zen surfaces

### Package metadata

`package.json` currently identifies the package as `zen-desktop` and points repository, bugs, and homepage metadata to `zen-browser/desktop`.

Required Titan decision:

- package name: likely `titan-desktop` or `titan-browser-desktop`;
- repository URL: `outlaw-dame/desktop.zen` until/unless renamed;
- bugs URL: Titan issue tracker;
- homepage: Titan project README or website;
- author/vendor metadata: Titan-owned identity.

### Surfer product metadata

`surfer.json` currently contains Zen product identity:

- display name;
- vendor;
- application id;
- binary name;
- release brand names;
- GitHub release repo pointers;
- update hostname;
- archive names.

Required Titan decision:

- app name: `Titan` or `Titan Browser`;
- app id: `titan`;
- binary name: `titan`;
- vendor: Titan-owned vendor string;
- display versions: Titan-owned versioning;
- release repo: this fork or a renamed Titan repo;
- update hostname: must not point at Zen unless Titan intentionally consumes Zen updates without publishing Titan builds.

### Branding assets

Current generated or checked-in branding may still use Zen names/logos. Before distribution, audit:

- `configs/branding/**`;
- generated app icons;
- About dialog strings;
- application menu names;
- desktop files;
- macOS bundle identifiers and plist values;
- Windows installer metadata;
- Linux AppImage metadata;
- crash/update URLs;
- release artifacts.

### Preferences and namespaces

Titan should introduce its own preference namespace before adding product features:

```text
titan.*
```

Avoid adding new Titan behavior under `zen.*` unless the behavior is intentionally extending an existing Zen mechanism.

### Workflows

GitHub Actions workflows still appear to be Zen release workflows. They reference Zen artifact names, release branches, self-hosted runner assumptions, deployment secrets, and release behavior.

Required Titan decision:

- which workflows are safe for PR validation;
- which workflows are release-only and should remain manual/quarantined;
- which secrets Titan actually owns;
- how artifact names should change;
- whether release workflows should be disabled until signing and update hosting exist.

## Pre-release hygiene checklist

Titan must not publish binaries until all of the following are true:

- [ ] Titan app name is distinct from Zen and Firefox.
- [ ] Titan profile directory is distinct from Zen and Firefox.
- [ ] Titan binary/app id/bundle id are distinct.
- [ ] Titan icons and branding are distinct.
- [ ] Titan update host does not point to Zen infrastructure.
- [ ] Titan release repository does not point to Zen infrastructure.
- [ ] Titan artifact names do not imply official Zen builds.
- [ ] Titan issue/security URLs point to Titan-owned locations.
- [ ] Titan release notes clearly state upstream base and fork status.
- [ ] MPL-2.0 source obligations are satisfied.
- [ ] Build provenance is documented.
- [ ] Security update process is documented and assigned.

## Principle

Do not remove upstream attribution. Do remove ambiguity. Titan can acknowledge that it is based on Zen and Firefox while still using its own product identity, update system, support channels, and release process.
