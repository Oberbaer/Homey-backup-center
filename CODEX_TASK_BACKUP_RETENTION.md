# Codex Task: Configurable Network Backup Retention

Branch: `feature/backup-retention`

## Goal

Add a safe configurable retention policy for network backups in Homey Backup Center.

## First inspect

Before changing code, inspect these areas and reuse the existing architecture:

- `lib/network.js`
- `lib/network-worker.js`
- `lib/transfers.js`
- `lib/scheduler.js`
- `settings/network.js`
- `settings/ui.js`
- `settings/index.html`
- `settings/i18n.js`
- existing tests under `test/`
- network backup documentation under `docs/`

Identify:
1. where destinations are persisted,
2. where successful network backups complete,
3. how SMB2/SFTP/FTP/WebDAV operations are abstracted,
4. whether each transport already supports safe directory listing and file deletion.

Summarize the implementation plan before editing.

## Feature

Add optional retention settings per network destination:

- `retentionEnabled: boolean`
- `retentionDays: integer`
- `minimumBackupsToKeep: integer`

Defaults:

- `retentionEnabled = false`
- `retentionDays = 60`
- `minimumBackupsToKeep = 3`

Existing destinations without these fields must behave exactly as before.

## UI

Add a retention section to each network destination:

- Enable automatic cleanup
- Keep backups for: X days
- Always keep at least: X backups

Validation:

- retentionDays >= 1
- minimumBackupsToKeep >= 1

Retention controls should be disabled/hidden appropriately when retention is off.

## Cleanup behavior

After a network backup completes successfully:

1. Run cleanup for that same destination.
2. List files in that destination.
3. Positively identify only Backup Center backup files.
4. Sort eligible backups by reliable timestamp/date.
5. Delete files older than `retentionDays`.
6. Never reduce retained valid backups below `minimumBackupsToKeep`.
7. Never delete the backup just created.

## Safety requirements

- Never delete unrelated files.
- Never recursively delete directories.
- Never delete active `.partial`/temporary files.
- Use a strict filename/parser strategy, not a broad wildcard.
- Validate remote metadata before deleting.
- Cleanup must only run after a successful backup.
- Cleanup failure must not turn the completed backup into a failed backup.
- Log/report cleanup failures separately.
- Do not weaken SMB/SFTP/FTP/WebDAV security.
- If a transport cannot safely list/delete, skip cleanup for that transport and report the limitation.

## Protocols

Support retention where safe for:

- SMB2
- SFTP
- FTP
- WebDAV

Prefer one shared retention service/helper with thin transport-specific list/delete methods.

## Tests

Add automated tests for at least:

- retention disabled
- no expired backups
- backup younger than retention period
- one expired backup
- multiple expired backups
- minimumBackupsToKeep protection
- newest/current backup never deleted
- unrelated files ignored
- malformed backup filenames ignored
- directories ignored
- partial/temp files ignored
- listing failure
- deletion failure
- successful backup remains successful when cleanup fails
- supported transport adapters

Run all existing tests plus new tests.

## Documentation

Update relevant README/docs explaining:

- retention is optional and disabled by default
- example: keep backups for 60 days
- minimum backup safety floor
- cleanup runs only after successful backups
- protocol-specific limitations, if any

## Constraints

- Do not change the backup file format unless necessary.
- Do not bump the public app version yet.
- Avoid unrelated refactors.
- Do not merge automatically.
- At the end, report:
  - changed files
  - tests run and results
  - any limitations
  - any follow-up work recommended
