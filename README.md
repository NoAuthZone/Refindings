# Refindings

**Track and triage findings from recurring security scans over time: every finding found again is recognised, with your rating attached.**

Repository: **<https://github.com/NoAuthZone/Refindings>** · [Download `refindings.html`](https://github.com/NoAuthZone/Refindings/raw/main/refindings.html) 

Refindings is a single HTML file that runs entirely in your browser. You load the results of recurring scans
from **Semgrep, CodeQL, Bandit, Gitleaks, Trivy, KICS, 2ms, Checkmarx (CxSAST / Checkmarx One), Joern, Bearer,
Snyk Code, Checkov, ESLint, AI reviewers such as Metis, Gito, vulnhuntr or Claude Code Security Review, any other
SARIF tool** or FalconEYE, rate each finding once (confirmed, false positive,
accepted risk, fixed), and the tool recognises the same findings in every later scan. You see how often a
finding is reported, what is new, what disappeared, and what came back. Reports go out as HTML (print to PDF),
Markdown, SARIF (with your false positives as suppressions) or as a baseline that FalconEYE can use to suppress
known findings.

No installation, no server, no network access. Your data stays on your machine.



Refindings was previously called *Scan-Triage*. Project files written under the old name still open and merge,
and data kept in the browser carries over.

---

## Contents

- [Why](#why)
- [Quick start](#quick-start)
- [Supported input](#supported-input)
- [Producing scan files](#producing-scan-files)
- [Several tools in one project](#several-tools-in-one-project)
- [How findings are matched across scans](#how-findings-are-matched-across-scans)
- [Rating findings](#rating-findings)
- [Filters](#filters)
- [Comparing scans](#comparing-scans)
- [Projects](#projects)
- [Exports](#exports)
- [FalconEYE baseline](#falconeye-baseline)
- [Settings](#settings)
- [Keyboard shortcuts](#keyboard-shortcuts)
- [Security and privacy](#security-and-privacy)
- [Limitations](#limitations)
- [Development notes](#development-notes)

---

## Why

Recurring scans produce the same findings again and again, mixed with a few new ones. Rule-based tools like
Semgrep or Bandit report the same place at a different line after every edit; LLM-based scanners are not
deterministic at all. The same code scanned twice yields different titles, different
severities and a different set of findings. In the DVWA test data used during development, three scans
produced 991 raw findings that group into 676 distinct findings; only 74 of them were reported by all three
scans, and 480 appeared only once.

Without a tool you end up re-reading the same false positives after every run, and you cannot tell whether a
finding that disappeared was fixed or just not reported this time. Refindings keeps one record per real
finding, carries your rating forward, and shows the history.

## Quick start

1. Download `refindings.html` from the [repository](https://github.com/NoAuthZone/Refindings) (or
   `git clone https://github.com/NoAuthZone/Refindings.git`) and open it in a current browser (Chrome, Edge,
   Firefox or Safari).
2. Click **Load scans** or drop one or more result files onto the page (`.sarif`, `.json`, `.xml`, `.txt`, `.log`, `.html`, also `.gz`).
3. Click a finding to open the detail view and rate it with a key: `c` confirmed, `f` false positive,
   `a` accepted risk, `x` fixed, `o` open.
4. Load the next scan whenever it is available. Ratings carry over automatically.
5. Use **Project → Save project file** to keep a copy of your work, and **Export** for reports.

Your state is also kept in the browser's IndexedDB, so closing and reopening the file continues where you left
off. The project file is the portable copy and the one to back up.

## Supported input

The format is recognised from the content, not from the file name. The [`examples/`](examples/) folder has a
sample report for every format below (a fictional project scanned by 13 tools), ready to drop onto the page.

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
| FalconEYE raw result (`*_result.json`: `metadata`, `found_snippet`, `actual_snippet`, `verifier_data`) | findings, flagged code, full file content (for change detection), verifier verdict |
| FalconEYE 2.0 report as JSON (`review`, `location` object, `code_snippet` text, severity as word) | findings, flagged code, recommendation, confidence |
| FalconEYE report as HTML | same as the JSON report (read as text, no script is executed) |
| any of the above gzip-compressed (`.gz`) | decompressed in the browser |

Severities become numbers 0–10: SARIF `security-severity` as given, otherwise error = 8, warning = 5, note = 1;
words critical = 10, high / ERROR = 8, medium / WARNING = 5, low / INFO = 1. Gitleaks findings count as 8.
Secrets from 2ms and Gitleaks SARIF are masked as well (their SARIF carries the secret as snippet).

Not importable, because these tools write no result file: kodus-ai (PR comments only), buttercup (patches and
proofs of vulnerability, no finding report), agentic-security (web UI / e-mail report).

For FalconEYE files, slightly different variants are accepted as well: other field names (`title` instead of `issue`, `results`
instead of `findings`, `file` instead of `file_path`), severities as number or word
(critical = 10, high = 8, medium = 5, low = 1, info = 0), findings as a bare array, dates without time zone.

The **same scan in several formats** (for example the raw result and the HTML report) is detected by its ID, or
by start time and number of findings, and counted once. Each format contributes what it has: the raw result
brings file hashes and the verifier verdict, the report brings the recommendation and confidence.

The **scan label** is the tool name plus whatever the file name adds (`semgrep-main.sarif` → *Semgrep main*);
for FalconEYE it is taken from the file name, such as `mlx` or `CoderNEXT`. Click the name under a bar in the
chart to rename a scan. Tools that write no start time (Semgrep JSON, Gitleaks, Bearer, 2ms, Joern, Metis …)
take the scan time from a date in the file name (`bearer-2026-09-15.json`, `joern_20260915_0800.txt`), otherwise
from the file's modification time, so put the date into the file name or keep the files as they were written.

## Producing scan files

Run the scanners from the repository root so that paths are relative to it, and keep one file per run:

```bash
# Semgrep (SARIF carries the code snippet and nosemgrep suppressions; JSON works too)
semgrep scan --config auto --sarif -o semgrep-$(date +%F).sarif
semgrep scan --config auto --json  -o semgrep-$(date +%F).json

# CodeQL
codeql database analyze db --format=sarif-latest --output=codeql-$(date +%F).sarif

# Bandit
bandit -r . -f json -o bandit-$(date +%F).json

# Gitleaks (JSON is preferred over its SARIF: only the JSON importer can redact the secret)
gitleaks dir . --report-format json --report-path gitleaks-$(date +%F).json

# Trivy (dependencies, IaC misconfigurations, secrets)
trivy fs --scanners vuln,misconfig,secret --format json  -o trivy-$(date +%F).json .
trivy fs --format sarif -o trivy-$(date +%F).sarif .

# Snyk Code, Checkov, ESLint
snyk code test --sarif-file-output=snyk-$(date +%F).sarif
checkov -d . -o sarif --output-file-path .
eslint . -f @microsoft/eslint-formatter-sarif -o eslint-$(date +%F).sarif

# KICS (IaC) – JSON has the most detail; sarif, glsast, sonarqube and codeclimate work too
kics scan -p . -o kics-$(date +%F) --report-formats json

# 2ms (secrets)
2ms filesystem --path . --report-path 2ms-$(date +%F).json

# Checkmarx: CxSAST XML report from the portal / CxCLI, or Checkmarx One CLI
cx results show --scan-id <id> --report-format json --output-name cxone-$(date +%F)

# Joern
joern-scan ./src > joern-$(date +%F).txt

# Bearer
bearer scan . --format json --output bearer-$(date +%F).json

# AI reviewers
metis --codebase-path . --output-file metis-$(date +%F).json   # then: review_code
vulnhuntr -r . && mv vulnhuntr.log vulnhuntr-$(date +%F).log
codescan … > codescan-$(date +%F).txt
```

In CI, collect the files as build artefacts and load them into Refindings whenever you triage.

## Several tools in one project

Findings from all tools live in one list. A **Tool** filter row appears as soon as more than one tool is
loaded, the scan list and the report appendix show the tool of every scan, and the detail view lists which
tools reported a finding.

- **"Latest scan" is judged per tool.** A Bandit run does not make all Semgrep findings "no longer reported";
  *new*, *gone*, *in latest scan* and *in every scan since first seen* compare each finding with the scans of
  the tool(s) that reported it.
- **Same code, same category, different tools → one finding.** When Semgrep and FalconEYE flag the same lines
  as SQL injection, they are matched into one finding, your rating applies to both, and the detail view shows
  both tools under *Reported by*.
- **Rule-based tools are matched strictly**: two findings of one scan never collapse into one, and the line is
  part of the fingerprint. A finding that moved a few lines is still recognised by its code (or, without code,
  by rule, message and nearby lines).
- **Suppressed in the tool** (SARIF `suppressions`, `nosemgrep`) counts as a scanner verdict: hidden by
  default, visible with the scanner-verdict chip *only suppressed by tool*.

## How findings are matched across scans

A finding is identified by:

1. **File path relative to the code base.** Absolute prefixes differ between machines
   (`C:/code/`, `/code/`), so they are stripped. The prefix is kept per scan
   and can be shown again with the **full path** toggle in the File column.
2. **Category** (SQL injection, XSS, path traversal, vulnerable dependency, …, 22 categories). Taken from the
   CWE when the tool reports one, otherwise from the title and rule id. The scanner's wording changes from run
   to run; the category is stable.
3. **Flagged code.** An exact fingerprint (FNV-1a over path, category and normalised code) matches first.
   Otherwise the token similarity (Jaccard) of the flagged code decides, with a bonus for overlapping line
   ranges. Each finding in a scan is matched to at most one existing finding.

Grouping is always recomputed from the stored raw records in chronological order. Loading scans in a different
order, loading an older scan later, removing a scan or merging projects therefore gives exactly the same result
as loading everything fresh.

When the scanner reports the same place under two categories (for example "Missing input validation" and
"SQL injection"), use **Merge** in the detail view or on a selection. The mapping is kept for future scans and
can be undone with **Split**.

## Rating findings

| Status | Meaning |
|---|---|
| Open | not rated yet, or explicitly reopened |
| Confirmed | real vulnerability |
| False positive | not a vulnerability |
| Accepted risk | known and accepted; gets a review date |
| Fixed | fixed; reported again later → flagged as **regression** |

- **Rules** set a status for whole paths or categories, for example `external/**` as accepted risk. An
  individual rating always wins over a rule.
- **Notes** are stored per finding and appear in reports.
- **Bulk actions** work on a selection. From 20 findings on, the tool asks for confirmation.
- **Undo** with `Ctrl/Cmd+Z` or the *Undo* button in the message covers ratings, bulk actions, rules, merges,
  scan removal, renames, project merges and reset.
- **Review dates**: an accepted risk gets a review date (default 90 days). Once reached, it is flagged
  *review due* and left out of the FalconEYE baseline until you review it again.
- **Code change detection** (needs raw results): *code changed, still reported* means a fix did not work;
  *gone, file changed* means probably fixed; *gone, file unchanged* means the scanner just did not report it.

## Filters

All filters are chips with live counts that respect the other active filters.

| Row | Behaviour |
|---|---|
| Status | several at once (OR) |
| Severity | critical 9–10, high 7–8, medium 4–6, low 1–3, info 0 (OR) |
| Flags | new, regression, no longer reported, gone with file changed/unchanged, code changed, unrated, severity varies, many wordings, SLA overdue, review due, rating conflict, by rule, with note, merged (AND) |
| Category | several at once (OR), most frequent first |
| Tool | several at once (OR); only shown with more than one tool |
| Frequency | in latest scan, in every scan since first seen, in at least N scans, only in one scan |
| Reported in / Not reported in | per scan, combinable, e.g. "only reported by model A and B" |
| Status changed | today, last 7 days, last 30 days, never |
| Scanner verdict | hide findings the scanner rejected or suppressed (default), show all, only rejected, only hallucinations, only self-refuted (FalconEYE verifier), only suppressed by tool (SARIF suppressions, nosemgrep) |

Less-used rows fold away under **More filters**; a row with an active choice stays visible. Folder and
exclude-folder filters are dropdowns (tree). The search understands several terms (AND), `-term`, `"phrases"`
and the fields `file:`, `note:` and `cat:`.

The **Worklist** tile selects open, unrated findings reported in at least two scans, which filters out the
noise of single runs.

## Comparing scans

**Compare two scans** shows, for any scan A and B: new in B, gone from A, gone with file changed or unchanged,
in both, severity up and severity down. Click a number to filter the list. Each row then shows
`A→B: sev. 5 → 8`. Useful when the scans come from different LLMs: the comparison shows what one model finds
that the other does not.

## Projects

- **Save project file** writes a gzip-compressed `.json.gz` (roughly a quarter of the plain size);
  an uncompressed variant is available too.
- **Open project file** replaces the current state.
- **Merge project file** adds the scans, findings, ratings and rules of another project. Scans present in both
  count once. If both projects rated a finding differently, the newer rating wins and the finding is flagged
  *rating conflict*. The detail view lists all competing ratings with project, date and note and resolves the
  conflict with one click.
- **Import ratings only** maps status and notes onto matching findings without adding scans.
- **Rename project**: the name appears in the header, reports and export file names.

## Exports

### HTML and Markdown report

**Export → HTML report …** or **Markdown report …** opens a dialog:

- scope: all findings or the current filter; statuses; minimum severity
- sections: summary, changes since last report, scan history, comparison, breakdown, age and fix deadlines,
  detail cards, lists by status, rules, appendix
- detail cards for confirmed findings and regressions only, or **every finding in full**
  (code, reasoning, recommendation, history, CWE/OWASP, note)
- paths relative to the code base or as reported; **redact user names** in `C:/code/…`, `/code/…` and
  `/home/…` everywhere (on by default)
- **Mark as reported**: the next report lists what changed since this one

The report contains a generated summary, CWE and OWASP Top 10 (2021) per category, breakdowns (severity × status,
category × severity, OWASP, top files and modules), age and fix-deadline statistics, a table of contents,
collapsible sections, a print layout with page footer, and an appendix with the input scans, a SHA-256 checksum
over scans and ratings, and a decision log. The HTML report carries its own Content-Security-Policy and contains
no script.

### SARIF 2.1.0

Findings currently reported (latest scan of each tool) with Refindings as driver and the source tools listed
as extensions, rule descriptions, CWE tags (`external/cwe/cwe-89`), `security-severity`,
baseline state (`new`/`unchanged`), and false positives / accepted risks as `suppressions` with your note as
justification. Validated against the official SARIF schema; works with the VS Code SARIF viewer and GitHub code
scanning. With a base path set under *Settings*, the viewer opens the files directly.

### FalconEYE baseline

See the next section.

## FalconEYE baseline

The baseline is for FalconEYE only. For other tools use the SARIF export: its `suppressions` carry your false
positives and accepted risks, which GitHub code scanning and most SARIF viewers honour.

**Export → FalconEYE baseline** writes a JSON file with your false positives, accepted risks (not yet due for
review), rules, the category regexes and the matching parameters. `falconeye_baseline.py` applies it to a new
result with exactly the same matching as the tool:

```bash
# mark known findings as filtered (writes result.baselined.json)
python falconeye_baseline.py baseline.json result.json

# other output file / remove suppressed findings instead of marking them
python falconeye_baseline.py baseline.json result.json -o out.json
python falconeye_baseline.py baseline.json result.json --drop

# CI gate: exit code 1 if open findings with severity >= 8 remain
python falconeye_baseline.py baseline.json result.json --fail-on 8
```

Suppressed findings get `verifier_data.filtered = true` and `filter_reason = "baseline: …"`. Refindings
recognises this prefix and keeps such findings in the history instead of treating them as rejected by the
scanner. Accepted risks whose `review_at` date has been reached are reported again, even with an old baseline.

It also works as a module, for example directly inside FalconEYE:

```python
from falconeye_baseline import Baseline

bl = Baseline.load("baseline.json")
hit = bl.match(finding, relpath)   # relpath: path relative to the code base
if hit:
    ...                            # {"id", "status", "match", "note"}
```

Both the raw result and the FalconEYE 2.0 JSON report are accepted as input.

## Settings

| Setting | Default | Effect |
|---|---|---|
| Review interval for accepted risks | 90 days | review date for new accepted risks; 0 = none |
| Fix deadlines (SLA) per severity | critical 7, high 30, medium 90, low 180, info 0 days | open/confirmed findings older than this are flagged *SLA overdue* |
| Editor link | none | local base path and editor (VS Code, VSCodium, Cursor, JetBrains or a custom URL template) so that file paths open at the flagged line |

Column widths in the list can be dragged (double-click to reset) and are remembered per browser.

## Keyboard shortcuts

In the detail view:

| Key | Action |
|---|---|
| `o` `c` `f` `a` `x` | open, confirmed, false positive, accepted risk, fixed |
| `j` / `k` or arrow keys | next / previous finding |
| `Esc` | close |
| `Ctrl/Cmd+Z` | undo (anywhere outside text fields) |

A held-down status key rates only one finding. With *Jump to next finding after rating* the view advances
automatically.

## Security and privacy

- **No network.** A Content-Security-Policy in the page allows only its own two inline scripts (by SHA-256
  hash) and blocks every network request, including images, fonts and `fetch`. A crafted scan or project file
  cannot load or send anything.
- **Untrusted input is treated as untrusted.** Scan files are LLM output about foreign code. All values are
  type-checked on import, keys like `__proto__` are neutralised, output is escaped, editor links only allow app
  URL schemes, and project files are validated before use.
- **Local storage only.** State lives in the browser's IndexedDB (compressed) for this file; nothing leaves the
  machine unless you export it.
- **Reports** redact user names in paths by default and carry their own restrictive CSP.

## Limitations

- **Categories come from CWE or titles.** A finding without CWE and with an unusual title can land in *Other*
  or in a different category than expected. Merging fixes individual cases.
- **Trivy is tested against its documented JSON schema**, Semgrep, Bandit (JSON and SARIF) and Gitleaks against
  real runs, CodeQL against a SARIF sample. Report oddities of other SARIF producers as an
  [issue](https://github.com/NoAuthZone/Refindings/issues) with an anonymised sample.
- **Change detection needs FalconEYE raw results.** File hashes exist only in `_result.json` files; other
  formats cannot tell whether a file changed, so *gone* is not split into *file changed / unchanged* for them.
- **Ages refer to scan dates.** Age and fix deadlines count from the first scan that reported a finding, not
  from when the code was written.
- **Large projects grow.** Every scan keeps compact raw records so that regrouping is exact. Four scans with
  about 1,400 raw findings are roughly 1 MB as `.json.gz`.
- **Older project files** (before 1.1.1) have no raw records. They still open and merge approximately; load
  their raw scans again to enable exact regrouping.

## Development notes

- Everything is in `refindings.html`: styles, a core script (normalisation, matching, replay) and a UI
  script. There are no dependencies and no build step.
- **After editing either inline script, update the two `sha256-…` hashes in the
  `Content-Security-Policy` meta tag**, otherwise the browser refuses to run the page. The hash is the base64
  SHA-256 of the exact text between `<script>` and `</script>`.
- The version is `APP_VERSION` in the UI script; the changelog is the comment at the top of the file. Increase
  the version with every change.
- Importers for other tools are the `from…` functions next to `adaptForeign` in the core script; each one
  returns findings in the internal report shape (title, severity, file, lines, `found_snippet`, CWE, rule).
- `falconeye_baseline.py` must compute fingerprints, tokens and categories exactly like the tool
  (FNV-1a over UTF-16 code units, same token regex, category regexes shipped inside the baseline file).

