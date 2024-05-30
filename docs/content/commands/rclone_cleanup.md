---
title: "rclone cleanup"
description: "Clean up the remote if possible."
slug: rclone_cleanup
url: /commands/rclone_cleanup/
groups: Important
versionIntroduced: v1.31
# autogenerated - DO NOT EDIT, instead edit the source code in cmd/cleanup/ and as part of making a release run "make commanddocs"
---
# rclone cleanup

Clean up the remote if possible.

## Synopsis


Clean up the remote if possible.  Empty the trash or delete old file
versions. Not supported by all remotes.


```
rclone cleanup remote:path [flags]
```

## Options

```
  -h, --help   help for cleanup
```


## Important Options

Important flags useful for most commands.

```
  -n, --dry-run         Do a trial run with no permanent changes
  -i, --interactive     Enable interactive mode
  -v, --verbose count   Print lots more stuff (repeat for more)
```

See the [global flags page](/flags/) for global options not listed here.

# SEE ALSO

* [rclone](/commands/rclone/)	 - Show help for rclone commands, flags and backends.
