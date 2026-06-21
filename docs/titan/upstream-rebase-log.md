# Titan upstream rebase log

Use this log for every upstream sync from Zen Browser or Firefox-derived sources.

## Template

```markdown
## YYYY-MM-DD — upstream sync

- Previous Titan base:
- New Zen ref/commit:
- Firefox version before:
- Firefox version after:
- Surfer version before:
- Surfer version after:
- Security advisories reviewed:
- Conflicts:
- Conflict-resolution files:
- Build commands run:
- Test commands run:
- Release impact:
- Notes:
```

## Entries

### 2026-06-21 — phase 0 setup

- Previous Titan base: inherited `dev` branch from `outlaw-dame/desktop.zen`.
- New Zen ref/commit: no upstream sync performed in this change.
- Firefox version before: inherited from `surfer.json`.
- Firefox version after: unchanged.
- Surfer version before: inherited from `package.json`.
- Surfer version after: unchanged.
- Security advisories reviewed: not yet reviewed in-repo; release is blocked until security/update process exists.
- Conflicts: none.
- Conflict-resolution files: none.
- Build commands run: none; documentation-only change.
- Test commands run: none; documentation-only change.
- Release impact: no release should be produced from this branch.
- Notes: Added Titan phase 0 documentation to make subsequent fork hygiene, CI, branding, local-first, AI, and WebKit planning reviewable.
