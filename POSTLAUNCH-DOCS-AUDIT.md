# Post-launch documentation survivor audit

2026-09-16 · org · **7 matching source lines in 6 files**.

Scan: every tracked text file plus new documentation in this PR, case-insensitive:

```text
beta\.[234]|private|not published|under consideration|no npm|clawkeeper|openclaw-os
```

This generated report is excluded from its own input to avoid recursive matches; its search terms, quoted snippets and source paths intentionally repeat the findings below. No other path exclusion is applied. Old-version release notes, changelog, PROGRESS and migration matches are retained and enumerated, even though those histories are exempt from rewriting. One row accounts for every matching source line; multiple terms on that line are listed together. Line numbers refer to this PR tree.

Current state: five packages at 0.1.0-beta.5, beta = latest, no stable release; all three repositories public. Historical failures are not reclassified. Native approval advisory GHSA-22jj-m53c-524m was closed as requiring no upstream change on 2026-09-12; synchronous approval remains default-off by GatekeeperOS log-hygiene choice. No product logic, version, tag, registry write or upstream post is part of this change. Non-Markdown edits are documentation only: core installer comments/help, one installer-test receipt description, and the community snapshot-checker docstring.

## Survivors

| Source | Matched terms | Reason |
|---|---|---|
| [CODE_OF_CONDUCT.md:7](CODE_OF_CONDUCT.md#L7) | `private` | Confidential reporting/user-data protection guidance; remains necessary after public release. |
| [ISSUE_TEMPLATE/bug.yml:33](ISSUE_TEMPLATE/bug.yml#L33) | `private` | Confidential reporting/user-data protection guidance; remains necessary after public release. |
| [ISSUE_TEMPLATE/config.yml:5](ISSUE_TEMPLATE/config.yml#L5) | `private` | Confidential reporting/user-data protection guidance; remains necessary after public release. |
| [LICENSE:3](LICENSE#L3) | `clawkeeper` | Legal provenance: preserve original copyright, attribution and source URLs verbatim. |
| [SECURITY.md:9](SECURITY.md#L9) | `private` | Confidential reporting/user-data protection guidance; remains necessary after public release. |
| [SECURITY.md:29](SECURITY.md#L29) | `private` | Confidential reporting/user-data protection guidance; remains necessary after public release. |
| [profile/README.md:24](profile/README.md#L24) | `private` | Owner-only audience/resource boundary or protected conversation data; public repositories do not make user data public. |
