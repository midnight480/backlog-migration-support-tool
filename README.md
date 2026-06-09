# backlog-migration-support-tool

> 🇯🇵 [日本語版 README](README_ja.md)

A browser-based support tool for resolving common errors when using the [Backlog Migration Tool](https://backlog.com/ja/backlog-migration/releases.html).

## Overview

A web app that makes it easy to edit CSV files generated after running the `init` command.  
No server or installation required — just open `index.html` in your browser.

## Features

### 1. User Mapping CSV Editor

| Step | Description |
|------|-------------|
| **STEP 1** Select working folder | Select the extracted migration tool folder to auto-load `mapping/` and `log/` |
| **STEP 2** Review auto-matching | Automatically suggests destination users by email match. Approve/reject per row or in bulk |
| **STEP 3** Manual edit & save | Set destination via dropdown, filter search, download CSV in UTF-8 |

### 2. Log Collection (planned)

Parses log files after migration runs, matches known error patterns, and generates a support inquiry report.

## Target Files

Running the `init` command generates the following files in the `mapping/` directory.

| File | Description |
|------|-------------|
| `mapping/users.csv` | User mapping file (edit target) |
| `mapping/users_list.csv` | Destination space user list (reference) |

### `users.csv` Columns

| Column | Description |
|--------|-------------|
| `Source Backlog user id` | Source user ID |
| `Source Backlog user display name` | Source display name |
| `Source Backlog user email` | Source email address |
| `Destination Backlog user name` | **Destination user ID** (must be filled in) |

> ⚠️ If `Destination Backlog user name` is left empty, migration will fail with "Failed to convert user" error.

## Usage

### Prerequisites

1. Download and extract the [Backlog Migration Tool](https://backlog.com/ja/backlog-migration/releases.html)
2. Run the `init` command to generate the `mapping/` directory

```bash
# Zip version (macOS)
./bin/backlog-migration init \
  --src.key <source API key> \
  --src.url https://<space-name>.backlog.com/ \
  --dst.key <destination API key> \
  --dst.url https://<space-name>.backlog.com/ \
  --projectKey <source project key>:<destination project key>
```

### Launching the Tool

```bash
# Open index.html in your browser (double-click also works)
open index.html
```

> ⚠️ **Folder selection (File System Access API) is supported only in Chrome / Edge.**  
> If you use Safari or Firefox, expand the "Load individual files" section.

### Workflow

```
① Select working folder
   └─ Select the entire backlog-migration-1.7.0/ folder
      → mapping/users.csv ✅
      → mapping/users_list.csv ✅
      → log/*.log ✅ (auto-loaded)

② Review auto-matching
   └─ Users with matching email addresses are suggested automatically
      → Approve/reject per row with ✅ / ✖
      → "Approve all" for bulk processing

③ Manual edit & save
   └─ Set unconfigured rows (red highlight) via dropdown
      → Save with "Download users.csv"
      → Overwrite in the migration tool's mapping/ and re-run migration
```

## Supported Errors

Addresses the following errors documented in [Migration tool errors prevent migration](https://support-ja.backlog.com/hc/ja/articles/29471143249817).

| Error | Cause | Resolution |
|-------|-------|------------|
| `Failed to convert user` | `Destination Backlog user name` not set | Configure with this tool |
| `The maximum number of status are 12` | Status count exceeds limit | Detect & guide via log report (planned) |
| `No such Attachment` | Attachment already deleted | Classify as warning (planned) |
| `project.EditProject.err.exist.key` | Duplicate project key | Guide via log report (planned) |

## Tech Stack

- HTML / Vanilla CSS / Vanilla JavaScript (no build step)
- [PapaParse](https://www.papaparse.com/) (CSV parsing, via CDN)
- File System Access API (folder selection)

## References

- [Backlog Migration Tool Download](https://backlog.com/ja/backlog-migration/releases.html)
- [Manual (Zip version)](https://backlog.com/ja/backlog-migration/backlog-migration-manual-zip-1.7.0.pdf)
- [Manual (Jar version)](https://backlog.com/ja/backlog-migration/backlog-migration-manual-jar-1.7.0.pdf)
- [Migration errors support article](https://support-ja.backlog.com/hc/ja/articles/29471143249817)

## License

MIT License — Copyright (c) 2026 Taro Yamada
