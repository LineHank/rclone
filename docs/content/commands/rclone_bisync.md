---
title: "rclone bisync"
description: "Perform bidirectional synchronization between two paths."
slug: rclone_bisync
url: /commands/rclone_bisync/
groups: Filter,Copy,Important
status: Beta
versionIntroduced: v1.58
# autogenerated - DO NOT EDIT, instead edit the source code in cmd/bisync/ and as part of making a release run "make commanddocs"
---
# rclone bisync

Perform bidirectional synchronization between two paths.

## Synopsis

Perform bidirectional synchronization between two paths.

[Bisync](https://rclone.org/bisync/) provides a
bidirectional cloud sync solution in rclone.
It retains the Path1 and Path2 filesystem listings from the prior run.
On each successive run it will:
- list files on Path1 and Path2, and check for changes on each side.
  Changes include `New`, `Newer`, `Older`, and `Deleted` files.
- Propagate changes on Path1 to Path2, and vice-versa.

Bisync is **in beta** and is considered an **advanced command**, so use with care.
Make sure you have read and understood the entire [manual](https://rclone.org/bisync)
(especially the [Limitations](https://rclone.org/bisync/#limitations) section) before using,
or data loss can result. Questions can be asked in the [Rclone Forum](https://forum.rclone.org/).

See [full bisync description](https://rclone.org/bisync/) for details.


```
rclone bisync remote1:path1 remote2:path2 [flags]
```

## Options

```
      --backup-dir1 string                   --backup-dir for Path1. Must be a non-overlapping path on the same remote.
      --backup-dir2 string                   --backup-dir for Path2. Must be a non-overlapping path on the same remote.
      --check-access                         Ensure expected RCLONE_TEST files are found on both Path1 and Path2 filesystems, else abort.
      --check-filename string                Filename for --check-access (default: RCLONE_TEST)
      --check-sync string                    Controls comparison of final listings: true|false|only (default: true) (default "true")
      --compare string                       Comma-separated list of bisync-specific compare options ex. 'size,modtime,checksum' (default: 'size,modtime')
      --conflict-loser ConflictLoserAction   Action to take on the loser of a sync conflict (when there is a winner) or on both files (when there is no winner): , num, pathname, delete (default: num)
      --conflict-resolve string              Automatically resolve conflicts by preferring the version that is: none, path1, path2, newer, older, larger, smaller (default: none) (default "none")
      --conflict-suffix string               Suffix to use when renaming a --conflict-loser. Can be either one string or two comma-separated strings to assign different suffixes to Path1/Path2. (default: 'conflict')
      --create-empty-src-dirs                Sync creation and deletion of empty directories. (Not compatible with --remove-empty-dirs)
      --download-hash                        Compute hash by downloading when otherwise unavailable. (warning: may be slow and use lots of data!)
      --filters-file string                  Read filtering patterns from a file
      --force                                Bypass --max-delete safety check and run the sync. Consider using with --verbose
  -h, --help                                 help for bisync
      --ignore-listing-checksum              Do not use checksums for listings (add --ignore-checksum to additionally skip post-copy checksum checks)
      --max-lock Duration                    Consider lock files older than this to be expired (default: 0 (never expire)) (minimum: 2m) (default 0s)
      --no-cleanup                           Retain working files (useful for troubleshooting and testing).
      --no-slow-hash                         Ignore listing checksums only on backends where they are slow
      --recover                              Automatically recover from interruptions without requiring --resync.
      --remove-empty-dirs                    Remove ALL empty directories at the final cleanup step.
      --resilient                            Allow future runs to retry after certain less-serious errors, instead of requiring --resync. Use at your own risk!
  -1, --resync                               Performs the resync run. Equivalent to --resync-mode path1. Consider using --verbose or --dry-run first.
      --resync-mode string                   During resync, prefer the version that is: path1, path2, newer, older, larger, smaller (default: path1 if --resync, otherwise none for no resync.) (default "none")
      --slow-hash-sync-only                  Ignore slow checksums for listings and deltas, but still consider them during sync calls.
      --workdir string                       Use custom working dir - useful for testing. (default: {WORKDIR})
```


## Copy Options

Flags for anything which can Copy a file.

```
      --check-first                                 Do all the checks before starting transfers
  -c, --checksum                                    Check for changes with size & checksum (if available, or fallback to size only).
      --compare-dest stringArray                    Include additional comma separated server-side paths during comparison
      --copy-dest stringArray                       Implies --compare-dest but also copies files from paths into destination
      --cutoff-mode HARD|SOFT|CAUTIOUS              Mode to stop transfers when reaching the max transfer limit HARD|SOFT|CAUTIOUS (default HARD)
      --ignore-case-sync                            Ignore case when synchronizing
      --ignore-checksum                             Skip post copy check of checksums
      --ignore-existing                             Skip all files that exist on destination
      --ignore-size                                 Ignore size when skipping use modtime or checksum
  -I, --ignore-times                                Don't skip items that match size and time - transfer all unconditionally
      --immutable                                   Do not modify files, fail if existing files have been modified
      --inplace                                     Download directly to destination file instead of atomic download to temp/rename
      --max-backlog int                             Maximum number of objects in sync or check backlog (default 10000)
      --max-duration Duration                       Maximum duration rclone will transfer data for (default 0s)
      --max-transfer SizeSuffix                     Maximum size of data to transfer (default off)
  -M, --metadata                                    If set, preserve metadata when copying objects
      --modify-window Duration                      Max time diff to be considered the same (default 1ns)
      --multi-thread-chunk-size SizeSuffix          Chunk size for multi-thread downloads / uploads, if not set by filesystem (default 64Mi)
      --multi-thread-cutoff SizeSuffix              Use multi-thread downloads for files above this size (default 256Mi)
      --multi-thread-streams int                    Number of streams to use for multi-thread downloads (default 4)
      --multi-thread-write-buffer-size SizeSuffix   In memory buffer size for writing when in multi-thread mode (default 128Ki)
      --no-check-dest                               Don't check the destination, copy regardless
      --no-traverse                                 Don't traverse destination file system on copy
      --no-update-dir-modtime                       Don't update directory modification times
      --no-update-modtime                           Don't update destination modtime if files identical
      --order-by string                             Instructions on how to order the transfers, e.g. 'size,descending'
      --partial-suffix string                       Add partial-suffix to temporary file name when --inplace is not used (default ".partial")
      --refresh-times                               Refresh the modtime of remote files
      --server-side-across-configs                  Allow server-side operations (e.g. copy) to work across different configs
      --size-only                                   Skip based on size only, not modtime or checksum
      --streaming-upload-cutoff SizeSuffix          Cutoff for switching to chunked upload if file size is unknown, upload starts after reaching cutoff or when file ends (default 100Ki)
  -u, --update                                      Skip files that are newer on the destination
```

## Important Options

Important flags useful for most commands.

```
  -n, --dry-run         Do a trial run with no permanent changes
  -i, --interactive     Enable interactive mode
  -v, --verbose count   Print lots more stuff (repeat for more)
```

## Filter Options

Flags for filtering directory listings.

```
      --delete-excluded                     Delete files on dest excluded from sync
      --exclude stringArray                 Exclude files matching pattern
      --exclude-from stringArray            Read file exclude patterns from file (use - to read from stdin)
      --exclude-if-present stringArray      Exclude directories if filename is present
      --files-from stringArray              Read list of source-file names from file (use - to read from stdin)
      --files-from-raw stringArray          Read list of source-file names from file without any processing of lines (use - to read from stdin)
  -f, --filter stringArray                  Add a file filtering rule
      --filter-from stringArray             Read file filtering patterns from a file (use - to read from stdin)
      --ignore-case                         Ignore case in filters (case insensitive)
      --include stringArray                 Include files matching pattern
      --include-from stringArray            Read file include patterns from file (use - to read from stdin)
      --max-age Duration                    Only transfer files younger than this in s or suffix ms|s|m|h|d|w|M|y (default off)
      --max-depth int                       If set limits the recursion depth to this (default -1)
      --max-size SizeSuffix                 Only transfer files smaller than this in KiB or suffix B|K|M|G|T|P (default off)
      --metadata-exclude stringArray        Exclude metadatas matching pattern
      --metadata-exclude-from stringArray   Read metadata exclude patterns from file (use - to read from stdin)
      --metadata-filter stringArray         Add a metadata filtering rule
      --metadata-filter-from stringArray    Read metadata filtering patterns from a file (use - to read from stdin)
      --metadata-include stringArray        Include metadatas matching pattern
      --metadata-include-from stringArray   Read metadata include patterns from file (use - to read from stdin)
      --min-age Duration                    Only transfer files older than this in s or suffix ms|s|m|h|d|w|M|y (default off)
      --min-size SizeSuffix                 Only transfer files bigger than this in KiB or suffix B|K|M|G|T|P (default off)
```

See the [global flags page](/flags/) for global options not listed here.

# SEE ALSO

* [rclone](/commands/rclone/)	 - Show help for rclone commands, flags and backends.
