# Refindings 2.6.1

Review security findings from multiple scanners in one offline HTML application. Runs in a modern browser on Windows, Linux and macOS, without installation or a server. The page blocks all network access (strict Content Security Policy).

## Start

1. Download and open **refindings.html** in your browser.
2. Click **Load scans / triage** to import reports, or **Load test examples** to try the app (opens a separate demo project; your current work stays recoverable).
3. Enter a reviewer name, review findings and assign categories, ratings or assignees.
4. Export your work with **Project → Save project file** (password-protected ZIP) or **Save project file uncompressed** (JSON).

## Supported inputs

The format is detected from the content; several tools can live in one project. Identical findings are recognised across scans by file, category and flagged code, even if the scanner words the title differently.

- **SARIF 2.1.0** (Semgrep, CodeQL, Snyk, Checkov, Trivy, KICS, 2ms, Bearer, Metis and others)
- **JSON:** Semgrep, Bandit, Gitleaks, Trivy, KICS, 2ms, Bearer, Checkmarx One, Metis, Gito, Claude Code Security Review, promptfoo, vulnerability-agent, GitLab SAST, SonarQube generic issues, Code Climate, reviewdog (rdjson)
- **XML / text:** Checkmarx CxSAST XML, joern-scan and codescan console output, `vulnhuntr.log`
- **FalconEYE:** raw `*_result.json`, report JSON and HTML
- **Flowtrace:** JSON, HTML and `.refindings`
- **JSpringGuard:** JSON, HTML and SARIF reports, plus finding and flow/control triage files (unmatched reviews are preserved separately)
- Project files (`.zip` / `.json`) and gzip-compressed inputs

Secrets reported by secret scanners are masked on import and never stored.

## Features

- Inspect code context, scanner explanations and structured Flowtrace / FalconEYE evidence (source, sink, trace steps, verifier verdict).
- Track every finding across scans: history strip, regressions, "no longer reported", code-change detection, age and fix deadlines (SLA).
- Compare any two scans (new, gone, severity up/down, gone with changed or unchanged file).
- Filter by status, severity, category, tool, flags, folder, scan, scanner verdict, search syntax (`-term`, `"phrase"`, `file:`, `note:`, `cat:`) and per-column filters; resize columns.
- **Review queue** (priority by severity, regression, overdue, unassigned) and **My findings**.
- Assign reviewers, manage reviewers (create, rename, deactivate), apply bulk changes, merge and split findings, use Undo/Redo.
- Per-finding **change history** (assignee, status, category, note) and status history.
- Edit categories and rating names, colors and shortcuts under **Global settings**. Remove/Restore controls affect new manual assignments while preserving existing decisions.
- Configure accepted-risk review intervals: default **90 days**, quick choices **30/90/180/365 days**.
- Path rules, editor links (VS Code, VSCodium, Cursor, JetBrains, custom) and scan renaming.
- Merge project files or import ratings only; conflicting ratings are flagged for resolution.

## Exports

- HTML and Markdown reports with selectable sections, statuses, severity threshold, path handling and user-name redaction; "changes since last report" tracking
- SARIF 2.1.0 (ratings as suppressions)
- FalconEYE baseline (false positives and accepted risks)
- JSpringGuard triage file (your ratings, keyed by the scanner fingerprint)

## Storage and privacy

Global preferences apply within the same browser storage context. They are not synchronized between computers. Export projects regularly; browser storage is not a backup.

Under **Project → Browser storage** choose how projects are kept between visits:

- **Save in the browser** (default): automatic copy and recovery snapshots, unencrypted.
- **Save in the browser, encrypted:** AES-256-GCM with a key derived from your password (PBKDF2, 600,000 rounds). The password is kept in memory only and asked once per tab. ZIP project files then use the same password.
- **Do not save in the browser:** memory only; closing the tab discards the project, and you are warned about unsaved changes.

Use **Project → Recovery** for saved browser copies and snapshots. Project files saved as ZIP use AES-256; choose your own password, because the suggested default `refindings` is public. Plain JSON exports and unencrypted browser storage are not protected. Only source code included in imported reports is displayed.

Export projects before deleting recovery copies or closing a project, and avoid simultaneous delete/restore operations in several tabs.

## License

Refindings: [MIT License](LICENSE), copyright 2026 NoAuthZone.

The ZIP library `@zip.js/zip.js` 2.20.0 is bundled inside the HTML. Its original BSD-3-Clause license and copyright notice are included in that file and must be retained.

[Repository](https://github.com/NoAuthZone/Refindings) · [Issues](https://github.com/NoAuthZone/Refindings/issues)
