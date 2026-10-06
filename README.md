# Refindings 2.0.0

Review security findings from multiple scanners in one place: rate findings, add notes, assign reviewers, schedule reviews, and track changes between scan runs.

Refindings runs offline as a single HTML file in your browser. No installation, server, or account is required.

## Quick start

1. Open [refindings-2.0.0.html](refindings-2.0.0.html) in your browser.
2. Import reports using **Load scans** or drag files onto the page. **Load test examples** opens a demo project.
3. Enter your name and select **Use name**.
4. Review findings and export your project using **Project → Save project file**.

## Collaboration and reviewers

Multiple people can work on their own copies of a project and merge their changes afterward. Collaboration uses exchanged project files, without live synchronization.

- Create, rename, or deactivate reviewers under **Project → Reviewers**.
- Click an existing reviewer in a finding, or add a name using **+ New reviewer**.
- Hand a finding over by selecting another reviewer, or set it to **Unassigned**.
- Change history shows edits and the supplied reviewer name. Ratings, notes, and assignments are included in project exports.

Reviewer names identify who is working on a finding; they are not user accounts or access permissions.

**Workflow:** Import scans → assign findings → distribute the project file → edit and export individual copies → merge returned files and review conflicts.

## Schedule reviews

For **Accepted risk** findings, set the next review date in the finding. Configure a default interval for newly accepted risks under **Settings**. Review dates can also be set for rules that accept risks.

Due reviews are marked in the project, and the date can be extended when reviewing again. No calendar events or email reminders are generated.

## Merge projects

Use **Project → Merge project file** to combine multiple project files. Merging includes scans, findings, ratings, notes, assignments, and rules.

Before merging, the preview shows new and shared scans and detectable rating conflicts. Review conflicts afterward. Older project files without complete scan records are merged approximately.

## Supported imports

Formats are detected from file contents. A single project can contain reports from multiple scanners.

| Tool / format | Supported import |
|---|---|
| SARIF | SARIF 2.x, including reports from Semgrep, CodeQL, Snyk, Checkov, Trivy, KICS, 2ms, Bearer, and Metis |
| Semgrep | JSON |
| Bandit | JSON |
| Gitleaks | JSON |
| Trivy | JSON: package vulnerabilities, misconfigurations, and secrets |
| KICS | JSON |
| 2ms | JSON |
| Bearer | JSON |
| GitLab SAST | JSON |
| SonarQube | JSON in the generic/external issue format |
| Code Climate | JSON |
| reviewdog | rdjson |
| Checkmarx CxSAST | XML report |
| Checkmarx One | JSON |
| joern-scan | Console output saved as text |
| codescan | Console output saved as text |
| vulnhuntr | Log with one JSON event per line |
| Metis | JSON |
| Gito | JSON |
| Claude Code Security Review | JSON |
| promptfoo | JSON from code scans |
| vulnerability-agent | JSON |
| FalconEYE | Result JSON (`*_result.json`), report JSON, and HTML reports |
| Flowtrace | JSON, HTML, and `.refindings` |
| JSpringGuard | JSON, HTML, and SARIF |
| Refindings projects | Project JSON and encrypted project ZIP exports |

Gzip files (`.gz`) are decompressed when loaded. Support applies to recognized report structures; arbitrary JSON exports from a tool are not automatically compatible. Refindings imports existing reports and does not run scanners itself.

## Other features

Undo/Redo, scan comparison, filters, change history, and HTML, Markdown, and SARIF exports support the review process. Automatic navigation to the next finding is off by default and can be enabled under Settings.

## Browser storage

Changes are saved automatically in local browser storage. Each tab uses its own saved state. New tabs and page reloads start empty. Open saved browser copies through **Project → Recovery**, or exported project files through **Project → Open project or scan file**.

The save status distinguishes local saving from project export. When changes occur, recovery checkpoints are created approximately every two minutes; up to five are retained per tab session.

**A local save is not a backup.** Clearing browser data also removes projects and recovery checkpoints. Export your project regularly:

- **Save project file:** an AES-256 encrypted ZIP with a password you choose.
- **Save project file uncompressed:** an unencrypted JSON file.

| Action | Result |
|---|---|
| **Close project → Keep browser copy and close** | Clear the current tab and keep a browser copy for Recovery |
| **Close project → Delete browser copy and close** | Delete the project and its related browser copies and recovery checkpoints |
| **Delete all browser projects** in Project or Recovery | Delete all browser projects and backups, including older copies, without Undo or a new checkpoint |

Downloaded JSON and ZIP files remain on disk and must be deleted separately in your file manager. See the known deletion issues at the end of this README.

## Limitations and security

Scanner-suppressed findings are hidden by default. A finding disappearing from a later scan does not, by itself, prove that it was fixed.

ZIP encryption does not protect browser storage or unencrypted exports. The suggested password `refindings` is public; choose your own password for confidential projects.

### Known issues in 2.0.0

- Deleting an older Recovery copy can remove the browser copy of another project in the same tab session and prevent further saving. Export all affected projects before deleting.
- Deleting and restoring at the same time can recreate a deleted project. Perform these actions sequentially.

[Repository](https://github.com/NoAuthZone/Refindings) · [Report an issue](https://github.com/NoAuthZone/Refindings/issues) · MIT License
