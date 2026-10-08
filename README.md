# Refindings 2.4.0

Review security findings from multiple scanners in one offline HTML application. Runs in a modern browser on Windows, Linux and macOS, without installation or a server.

## Start

1. Download and open **refindings.html** in your browser.
2. Click **Load scans / triage** to import reports, or **Load test examples** to try the app.
3. Enter a reviewer name, review findings and assign categories or ratings.
4. Export your work with **Project → Save project file** (password-protected ZIP) or **Save project file uncompressed** (JSON).

## Features

- Import SARIF and recognized scanner reports, including FalconEYE, Flowtrace, JSpringGuard and Semgrep.
- Import JSpringGuard finding and flow/control triage. Unmatched reviews are preserved separately.
- Inspect code context, scanner explanations and structured Flowtrace/FalconEYE evidence.
- Filter columns, compare scans, assign reviewers, apply bulk changes and use Undo/Redo.
- Edit categories and rating names, colors and shortcuts under **Global settings**. Remove/Restore controls affect new manual assignments while preserving existing decisions.
- Configure accepted-risk review intervals: default **90 days**, quick choices **30/90/180/365 days**.
- Merge project files and export HTML, Markdown or SARIF reports.

## Storage

Global preferences apply within the same browser storage context. They are not synchronized between computers. Export projects regularly; browser storage is not a backup. Use **Project → Recovery** for saved browser copies.

Choose your own ZIP password; the suggested password `refindings` is public. Browser storage and plain exports are not encrypted by ZIP export. Only source code included in imported reports is displayed.

36 automated regression tests passed for this release. Browser layout and interaction were not live-tested. Earlier recovery-deletion issues have not been verified as resolved; export projects before deleting recovery copies and avoid simultaneous delete/restore operations.

## License

Refindings: [MIT License](LICENSE), copyright 2026 NoAuthZone.

The ZIP library `@zip.js/zip.js` 2.20.0 is bundled inside the HTML. Its original BSD-3-Clause license and copyright notice are included in that file and must be retained.

[Repository](https://github.com/NoAuthZone/Refindings) · [Issues](https://github.com/NoAuthZone/Refindings/issues)
