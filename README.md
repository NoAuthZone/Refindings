# Refindings 1.8.0

**Review recurring security findings, keep decisions across scans, and organise the work by reviewer.**

[Repository](https://github.com/NoAuthZone/Refindings) · [Application](refindings.html)

Refindings is a single HTML file that runs locally in your browser. Import scanner results, assign
findings, record decisions and notes, compare scans, and export reports or portable project files.
No installation, server, account or runtime network connection is required.

The interface and this guide use English labels.

## Contents

- [What's new in 1.8](#whats-new-in-18)
- [Quick start](#quick-start)
- [Projects and passwords](#projects-and-passwords)
- [Reviewers and assignments](#reviewers-and-assignments)
- [Review queue](#review-queue)
- [Change history](#change-history)
- [Supported input](#supported-input)
- [Ratings, rules and undo](#ratings-rules-and-undo)
- [Filters and scan comparison](#filters-and-scan-comparison)
- [Matching across scans](#matching-across-scans)
- [Reports and exports](#reports-and-exports)
- [Settings and shortcuts](#settings-and-shortcuts)
- [Storage, privacy and limitations](#storage-privacy-and-limitations)
- [Troubleshooting](#troubleshooting)
- [Development and tests](#development-and-tests)

## What's new in 1.8

This version brings together the features added since the original 1.7 release:

- A prioritised **Review queue**, explanations of priority, session progress and next-finding navigation.
- A sortable **Assignee** column, reviewer filter, individual and bulk assignments, and **My findings**.
- A **Change history** for assignment, status and note changes.
- Flowtrace JSON, HTML and `.refindings` import, with a source-to-sink path view.
- AES-256 encrypted ZIP projects with a selectable password and compatibility with older JSON/GZIP projects.

## Quick start

1. Save [refindings.html](refindings.html) and open it in a current Chrome, Edge, Firefox or Safari browser.
2. Choose **Load scans**, or drag files onto the page. JSON and supported HTML reports remain readable
   without encryption; Flowtrace `.refindings` files are accepted too.
3. Open a finding and rate it. Enter your reviewer name when prompted. Opening a saved project or
   restoring a browser project also asks for the current reviewer.
4. Click a name in **Assignee** to change its assignment, or select several rows and choose
   **Assign reviewer**. Use **My findings** to see your open work.
5. Choose **Review queue** to review findings in priority order.
6. Use **Project → Save project file** to create a portable backup. On the first ZIP export, choose
   a password; the suggested default is `refindings`.
7. Import subsequent scans to retain decisions and track new, recurring and disappearing findings.

The browser also saves work automatically in IndexedDB, with a localStorage fallback. Keep exported
project backups: browser storage is not a substitute for a portable copy.

## Projects and passwords

| Action | Behaviour |
|---|---|
| **Save project file** | Saves an AES-256 encrypted `.zip` containing `project.json`, including scans, ratings, assignments, notes, rules and change history. |
| **Save project file uncompressed** | Saves plain JSON, without encryption. |
| **Open project or scan file** | Detects the content. A project replaces the current state after the reviewer prompt; a scan report is added to the current project. |
| **Merge project file** | Combines project data. Shared scans count once; differing ratings can produce a rating conflict. |
| **Import ratings only** | Maps annotations onto matching findings without adding scans. |
| **Rename project** | Changes the name used in the header, reports and export filenames. |

### Saving an encrypted project

The first ZIP export of an opened project asks for a password and suggests **`refindings`**.
An empty password is not accepted; Cancel stops the export. The password is remembered only in memory
for that project and is not written to project JSON or browser storage. Reopening the project asks
again on its next export.

Use **Project → Change ZIP password** to choose a different password for future exports. This does
not change ZIP files already saved. An imported custom password is not automatically reused for exports.

### Opening an encrypted project

Refindings first tries the legacy default **`refindings`**. If decryption fails, it asks for the
password and allows another attempt. Cancel leaves the existing project intact. A damaged archive
may also fail to decrypt even with the correct password.

ZIP projects can be opened, merged or dropped onto the page. The importer expects one JSON file in
the archive, with a declared uncompressed size no greater than 512 MB. External extraction requires
an archive tool that supports WinZip AES.

Older `.json` and `.json.gz` projects, including projects named *Scan-Triage*, remain supported.
The Flowtrace `.refindings` format is a **scan report**, not a saved Refindings project.

The public default password is convenient but does not provide confidentiality. Choose a private
password for sensitive exports. ZIP encryption does not encrypt browser storage or guarantee that
antivirus warnings disappear.

## Reviewers and assignments

Two names have different purposes:

- **Current reviewer:** the name entered when opening a project, or set with **Project → Change reviewer**.
  This identifies the person performing subsequent edits. It is a local name, not an authenticated account.
- **Finding assignee:** the name in the **Assignee** column. Click it, or **Change** in the detail view,
  to assign a person. Enter an empty name to leave the finding explicitly unassigned.

Before an explicit assignment is made, editing a finding records the current reviewer as its assignee.
Once explicitly assigned, later edits by other reviewers preserve the assignment while the history
records who made the change. Opening a project does not reassign existing findings.

To assign several findings, select their checkboxes and choose **Assign reviewer**.
One Undo restores the entire batch. Assignments are included in project files and report detail cards.

The **Assignee** filter offers **All**, individual names and **Unassigned**, with counts.
The column can also be sorted. **My findings** resets other filters and selects open findings
assigned to the current reviewer. Scanner-rejected findings remain hidden by default.

## Review queue

Click **Review queue** to start a session. It resets the other filters and opens the first
eligible finding. The queue includes:

- Open and confirmed findings.
- Fixed findings reported again after the fix decision (regressions).
- Accepted risks whose review date is due.

Scanner-rejected findings are excluded. Fixed findings without a regression, false positives and
accepted risks that are not due do not enter the queue.

### Priority order

Priority is determined in this order, rather than by an opaque combined score:

1. Higher numerical severity.
2. Reported again despite being marked fixed.
3. Overdue review or remediation.
4. No assigned reviewer.
5. More days overdue for remediation.
6. File path and finding ID as stable tie-breakers.

The list and detail view show an explanation, such as **High severity (8/10) · Reported again after being marked fixed**.
The queue controls sorting while active; additional filters can narrow the displayed findings.

### Moving through the queue

Choose **Next finding**, or use `j`; `k` moves backwards. Navigation cycles through the remaining
matches. With **Jump to next finding after rating** enabled, a rating, renewed accepted-risk review,
or resolved rating conflict automatically advances.

Reviewed findings disappear from the queue for the current session, even when their status remains
open or confirmed. The counter shows remaining matches and the number reviewed in this session.
Undo of a finding edit makes that finding eligible for the queue again, subject to its status and filters.

Session progress is held in memory. Toggle the queue off and on, or reopen the project, to begin a
fresh session. Ratings and change history remain saved. Reaching zero remaining matches does not
necessarily mean all vulnerabilities are fixed.

## Change history

**Change history** in the finding detail records assignment, status and note changes with:

- The person who made the change and its timestamp.
- The changed field.
- Expandable before/after values.

Consecutive typing in a note is grouped into one edit. The most recent 100 entries per finding are
retained; imported note snapshots are limited to 10,000 characters. History survives project exports,
imports, merging and regrouping. Undo restores both the values and their previous history.

Existing projects do not receive a retroactive change log. Their older status history remains visible.
This is a local, editable activity history, not a tamper-proof audit trail.



## Supported input

Formats are detected from their content. File extensions control which files are offered in the picker.
Arbitrary HTML pages are not supported: HTML import recognises FalconEYE and Flowtrace reports.

| Format | What Refindings uses from it |
|---|---|
| **SARIF 2.1.0** (`.sarif`) from Semgrep, CodeQL, Snyk Code, Checkov, Trivy, ESLint, Bandit, Gitleaks, … | rule, message, rule descriptions and help, `security-severity` (else `level`), CWE from rule tags (`external/cwe/cwe-89`, `CWE-89`), taxa and relationships, file and region, snippet and context snippet, start time, `suppressions`; results with `kind` pass / notApplicable and `baselineState: absent` are skipped |
| **Semgrep JSON** (`semgrep scan --json`) | check id, message, severity, `metadata.cwe`, `security-severity`, confidence, fix, lines (Semgrep OSS writes "requires login" instead of the code, then the rule and message stand in) |
| **Bandit JSON** (`bandit -f json`) | test id and name, text, severity, confidence, CWE, code with line numbers, `generated_at` |
| **Gitleaks JSON** (`--report-format json`) | rule, description, file, line, match, commit. **The secret value is never stored**: it is replaced by `«redacted 1a2b3c»`, a short hash that changes when the secret is rotated |
| **Trivy JSON** (`trivy fs --format json`) | vulnerable packages (CVE, package, installed and fixed version, CWE), misconfigurations with cause lines (PASS results skipped), secrets (already masked by Trivy) |
| **KICS JSON** (`kics scan -o out --report-formats json`, `results.json`) | query name and id, severity, CWE, risk score, description, actual / expected value, resource, platform, remediation, start time. KICS writes no code: findings are matched by query and *search key* (resource + attribute), so they survive moved lines. Category from the CWE or title, otherwise *Configuration* |
| **2ms JSON** (`2ms filesystem --path . --report-path 2ms.json`) | rule, description, severity / CVSS, validation status, lines; `git show <commit>:<file>` sources become file + commit. **The secret value is never stored** (same masking as Gitleaks) |
| **Checkmarx CxSAST XML report** (`CxXMLResults`) | query, group, language, CWE, severity, every data flow. The **sink** (last node) is the location, the source and the whole flow go into the reasoning; state *Not Exploitable* / `FalsePositive="True"` counts as *suppressed by tool*; Checkmarx comments, state and deep link are kept |
| **Checkmarx One JSON** (`cx results show --report-format json`) | SAST (query, data flow, sink as location), IaC (KICS queries), SCA (CVE, package, recommended version), secret detection (snippet not stored), state *Not Exploitable* as *suppressed by tool* |
| **Joern** (`joern-scan` console output saved as `.txt`) | lines `Result: <score> : <title>: <file>:<line>:<method>`; the score is the severity |
| **Bearer JSON** (`bearer scan --format json`, also `jsonv2`) | rule, title, CWE, severity, sink code and lines, data categories; the rule description is split into *Description* (reasoning) and *Remediations* (recommendation) |
| **GitLab SAST report** (`gl-sast-report.json`, e.g. `kics --report-formats glsast`, `bearer --format gitlab-sast`) | name, description, solution, severity, confidence, identifiers (CWE, CVE), location, code extract (never for secret detection), dependency findings, *likely false positive* flags |
| **SonarQube generic issues** (e.g. `kics --report-formats sonarqube`) | rule, message, severity (blocker 10, critical 8, major 5, minor 1, info 0), location; KICS' secondary locations become findings of their own |
| **Code Climate** (e.g. `kics --report-formats codeclimate`) | check name, description, severity (same scale), location |
| **reviewdog rdjson** (e.g. `bearer --format reviewdog`) | message, rule code and URL, severity, range, suggestions |
| **Metis** (`--output-file review.json`, SARIF too) | issue, snippet, line, CWE, severity, reasoning, mitigation, confidence; Metis triage *invalid* counts as *self-refuted* |
| **Gito** (`code-review-report.json`) | title, details, tags, severity 1–5 (→ 10/8/5/1/0), confidence, affected lines with code and proposed change |
| **Claude Code Security Review** (`claudecode-results.json` or `findings.json`) | file, line, severity, category, description, exploit scenario, recommendation, confidence; findings the action's filter excluded are kept as *self-refuted* with the reason |
| **vulnhuntr** (`vulnhuntr.log`) | the last analysis per file and vulnerability type (LFI, RCE, SSRF, AFO, SQLI, XSS, IDOR): analysis, proof of concept, confidence; analyses that ended without a finding are skipped |
| **codescan** (console output saved as `.txt`) | per file: line, severity, type, issue, fix |
| **promptfoo code-scans** (`--json`) | file, lines, finding, fix, severity |
| **vulnerability-agent** (`--output json`) | npm advisories per repository: CVE/GHSA, package, version, CVSS, recommended upgrade |
| **Flowtrace** (`flowtrace.json`, `.refindings`, `.html`) | sink file, line and code, severity, CWE, source-to-sink trace, LLM assessment, reproduction and remediation where supplied. `not_vulnerable` is retained as a scanner rejection, not a manual rating. Multiple formats of the same run are counted once (root, scan minute and finding IDs). |
| FalconEYE raw result (`*_result.json`: `metadata`, `found_snippet`, `actual_snippet`, `verifier_data`) | findings, flagged code, full file content (for change detection), verifier verdict |
| FalconEYE 2.0 report as JSON (`review`, `location` object, `code_snippet` text, severity as word) | findings, flagged code, recommendation, confidence |
| FalconEYE report as HTML | same as the JSON report (read as text, no script is executed) |
| any of the above gzip-compressed (`.gz`) | decompressed in the browser |


Severities are normalised to 0–10. Findings rejected or suppressed by scanners are retained separately
from manual ratings. Tools without an embedded scan date generally use a date in the filename or the
file's modification time. Keep one file per run and preserve timestamps where possible.

Synthetic Flowtrace fixtures are available in [tests/fixtures/flowtrace](tests/fixtures/flowtrace/).

## Ratings, rules and undo

| Status | Meaning |
|---|---|
| Open | Not rated yet, or reopened. |
| Confirmed | A real vulnerability. |
| False positive | Not a vulnerability. |
| Accepted risk | Accepted with an optional review date. |
| Fixed | Marked fixed; a later sighting may become a regression. |

Rules apply a status to matching paths/categories; an individual rating takes precedence. Notes are
stored per finding and appear in reports. Bulk status changes ask for confirmation from 20 findings.
Accepted risks receive a review date by default and become **review due** once that date is reached.

**Undo** covers ratings, assignments, note edits, rules, merges, scan removal, renames, project merges
and reset. Undo history is limited and is not a permanent backup. Use `Ctrl/Cmd+Z` outside text fields
or the Undo action in a notification.

## Filters and scan comparison

Filters support status, severity, category, scanner, frequency, scan inclusion/exclusion, folders,
status-change date, scanner verdict and flags such as regression, overdue, conflict and merged.
The Assignee dropdown filters by assignment. Counts respect the other active filters; saved filter
preferences are local to the browser.

Use **More filters** for additional options. Search supports multiple terms, `-excluded`,
`"exact phrases"`, and the fields `file:`, `note:` and `cat:`.

The **Worklist** tile selects open, unrated findings reported in at least two scans. This is distinct
from the prioritised review queue and the reviewer-specific **My findings** view.

**Compare two scans** shows findings new in the second scan, gone since the first, present in both,
and changes in severity. File-change comparisons require source-file hashes from suitable raw results.
Click a comparison count to filter the finding list.

## Matching across scans

Matching uses relative file paths, vulnerability categories and normalised code. Exact fingerprints
are preferred; token similarity and overlapping line ranges provide fallback matching. Different
scanner wordings can therefore refer to the same finding.

Findings from multiple scanners share one list. The latest scan is evaluated per tool, so a Bandit
run does not make Semgrep findings disappear. Compact raw records allow chronological regrouping,
while ratings, notes, assignments, history and manual merges are carried forward.

Use **Merge** when findings describe the same issue under different categories, and **Split** to undo
a manual grouping. Review automatic matches when scanner output changes substantially. Older projects
without raw records can only be merged approximately.

## Reports and exports

| Export | Purpose |
|---|---|
| HTML report | Shareable report with print layout; use browser printing for PDF. |
| Markdown report | Text-based report for documentation. |
| SARIF 2.1.0 | Current findings with ratings represented in properties/suppressions. |
| FalconEYE baseline | JSON baseline for a compatible FalconEYE workflow. |
| Project ZIP / plain JSON | Portable working state, including assignments and change history. |

HTML/Markdown report options include scope, statuses, minimum severity, sections, code, detail cards,
path display and path-user-name redaction. Detail cards can cover selected key findings or every finding.
**Mark as reported** stores a snapshot for later change summaries. HTML reports contain no scripts.

The report decision log covers rating history; the complete assignment/note activity log is preserved
in the project file. Reports do not replace a project backup. Path redaction does not remove all possible
sensitive content, such as notes or code snippets.

A FalconEYE baseline contains manual false-positive and accepted-risk decisions and applicable rules.
Overdue accepted risks are omitted until reviewed. Applying a baseline requires compatible external
tooling; no `falconeye_baseline.py` helper is included in this repository.

## Settings and shortcuts

| Setting | Default / behaviour |
|---|---|
| Accepted-risk review interval | 90 days; 0 disables the automatic review date. |
| Remediation SLA | Critical 7, high 30, medium 90, low 180, info 0 days. |
| Editor links | Configure a local base path and supported editor/custom URL template. |
| Auto-next | Controls navigation after a rating. |
| Column widths | Drag column boundaries; double-click a handle to reset. Saved per browser. |

| Key | Action in the detail view |
|---|---|
| `o`, `c`, `f`, `a`, `x` | Open, confirmed, false positive, accepted risk, fixed. |
| `j` / `k` or arrow keys | Next / previous finding. |
| `Esc` | Close the detail view. |
| `Ctrl/Cmd+Z` | Undo outside text fields. |

A held-down status key rates only one finding.

## Storage, privacy and limitations

- The application works offline. Its Content Security Policy authorises three embedded scripts by
  SHA-256 hash and blocks runtime network requests. Imported HTML is never executed.
- Imported values are sanitised and escaped before display. Treat scanner outputs and exported
  project files as sensitive: code, notes and reproduction examples can contain confidential data.
- Browser storage is not protected by the ZIP password. Only encrypted ZIP exports receive that protection;
  plain JSON and report exports remain unencrypted.
- Reviewer names are self-declared. There is no authentication, permission system or live multi-user sync.
  Team members exchange project files and merge them.
- Date-based age/SLA calculations refer to scan dates. A disappeared finding is not proof of a fix.
- File-change detection requires hashes from FalconEYE raw results; most other formats cannot supply it.
- Categories are inferred from CWE, rules or titles and can be imperfect. Matching and merging may need review.
- Large histories consume more browser storage and memory. Export backups before clearing browser data.
- Some malformed, unsupported or externally repackaged archives/reports cannot be imported.

## Troubleshooting

| Symptom | What to check |
|---|---|
| Buttons do not respond after editing the HTML | Recompute the CSP hashes for every embedded script. See development notes below. |
| “Could not read” | Read the full message after the filename. Supply an anonymised sample when reporting the issue. |
| Findings appear to be missing | Reset filters and check the scanner-verdict filter. Flowtrace `not_vulnerable` findings are hidden by default. |
| “My findings” is empty | Check the current reviewer name, explicit assignments and whether findings are still Open. |
| Queue has no remaining findings | Entries may already be reviewed this session, excluded by filters, or ineligible. Toggle the queue off/on for a fresh session. |
| ZIP password keeps failing | Check the export password; the default is `refindings`. The archive may also be damaged. |
| Browser data disappeared | Open your exported project backup. Storage can depend on browser, profile and local file location. |

For issues, include the application version, browser, exact error and a sanitised example at the
[issue tracker](https://github.com/NoAuthZone/Refindings/issues).

## Development and tests

The application is self-contained in [refindings.html](refindings.html): styles, core logic,
the embedded ZIP library and UI logic. There is no build step or external runtime dependency.
`APP_VERSION` is defined in the core script; this release is **1.8.0**.

### Content Security Policy

After changing **any** embedded script, update the `script-src` hashes in the CSP meta tag. Each hash
is the base64 SHA-256 digest of the exact UTF-8 script text between its opening and closing tags,
including whitespace. Hash all three scripts and retain the existing restrictive policy.
`tests/project.cjs` checks that every shipped script is authorised; a syntax check alone will not catch
stale CSP hashes.


