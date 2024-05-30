---
title: "rclone serve ftp"
description: "Serve remote:path over FTP."
slug: rclone_serve_ftp
url: /commands/rclone_serve_ftp/
groups: Filter
versionIntroduced: v1.44
# autogenerated - DO NOT EDIT, instead edit the source code in cmd/serve/ftp/ and as part of making a release run "make commanddocs"
---
# rclone serve ftp

Serve remote:path over FTP.

## Synopsis


Run a basic FTP server to serve a remote over FTP protocol.
This can be viewed with a FTP client or you can make a remote of
type FTP to read and write it.

## Server options

Use --addr to specify which IP address and port the server should
listen on, e.g. --addr 1.2.3.4:8000 or --addr :8080 to listen to all
IPs.  By default it only listens on localhost.  You can use port
:0 to let the OS choose an available port.

If you set --addr to listen on a public or LAN accessible IP address
then using Authentication is advised - see the next section for info.

### Authentication

By default this will serve files without needing a login.

You can set a single username and password with the --user and --pass flags.

## VFS - Virtual File System

This command uses the VFS layer. This adapts the cloud storage objects
that rclone uses into something which looks much more like a disk
filing system.

Cloud storage objects have lots of properties which aren't like disk
files - you can't extend them or write to the middle of them, so the
VFS layer has to deal with that. Because there is no one right way of
doing this there are various options explained below.

The VFS layer also implements a directory cache - this caches info
about files and directories (but not the data) in memory.

## VFS Directory Cache

Using the `--dir-cache-time` flag, you can control how long a
directory should be considered up to date and not refreshed from the
backend. Changes made through the VFS will appear immediately or
invalidate the cache.

    --dir-cache-time duration   Time to cache directory entries for (default 5m0s)
    --poll-interval duration    Time to wait between polling for changes. Must be smaller than dir-cache-time. Only on supported remotes. Set to 0 to disable (default 1m0s)

However, changes made directly on the cloud storage by the web
interface or a different copy of rclone will only be picked up once
the directory cache expires if the backend configured does not support
polling for changes. If the backend supports polling, changes will be
picked up within the polling interval.

You can send a `SIGHUP` signal to rclone for it to flush all
directory caches, regardless of how old they are.  Assuming only one
rclone instance is running, you can reset the cache like this:

    kill -SIGHUP $(pidof rclone)

If you configure rclone with a [remote control](/rc) then you can use
rclone rc to flush the whole directory cache:

    rclone rc vfs/forget

Or individual files or directories:

    rclone rc vfs/forget file=path/to/file dir=path/to/dir

## VFS File Buffering

The `--buffer-size` flag determines the amount of memory,
that will be used to buffer data in advance.

Each open file will try to keep the specified amount of data in memory
at all times. The buffered data is bound to one open file and won't be
shared.

This flag is a upper limit for the used memory per open file.  The
buffer will only use memory for data that is downloaded but not not
yet read. If the buffer is empty, only a small amount of memory will
be used.

The maximum memory used by rclone for buffering can be up to
`--buffer-size * open files`.

## VFS File Caching

These flags control the VFS file caching options. File caching is
necessary to make the VFS layer appear compatible with a normal file
system. It can be disabled at the cost of some compatibility.

For example you'll need to enable VFS caching if you want to read and
write simultaneously to a file.  See below for more details.

Note that the VFS cache is separate from the cache backend and you may
find that you need one or the other or both.

    --cache-dir string                     Directory rclone will use for caching.
    --vfs-cache-mode CacheMode             Cache mode off|minimal|writes|full (default off)
    --vfs-cache-max-age duration           Max time since last access of objects in the cache (default 1h0m0s)
    --vfs-cache-max-size SizeSuffix        Max total size of objects in the cache (default off)
    --vfs-cache-min-free-space SizeSuffix  Target minimum free space on the disk containing the cache (default off)
    --vfs-cache-poll-interval duration     Interval to poll the cache for stale objects (default 1m0s)
    --vfs-write-back duration              Time to writeback files after last use when using cache (default 5s)

If run with `-vv` rclone will print the location of the file cache.  The
files are stored in the user cache file area which is OS dependent but
can be controlled with `--cache-dir` or setting the appropriate
environment variable.

The cache has 4 different modes selected by `--vfs-cache-mode`.
The higher the cache mode the more compatible rclone becomes at the
cost of using disk space.

Note that files are written back to the remote only when they are
closed and if they haven't been accessed for `--vfs-write-back`
seconds. If rclone is quit or dies with files that haven't been
uploaded, these will be uploaded next time rclone is run with the same
flags.

If using `--vfs-cache-max-size` or `--vfs-cache-min-free-size` note
that the cache may exceed these quotas for two reasons. Firstly
because it is only checked every `--vfs-cache-poll-interval`. Secondly
because open files cannot be evicted from the cache. When
`--vfs-cache-max-size` or `--vfs-cache-min-free-size` is exceeded,
rclone will attempt to evict the least accessed files from the cache
first. rclone will start with files that haven't been accessed for the
longest. This cache flushing strategy is efficient and more relevant
files are likely to remain cached.

The `--vfs-cache-max-age` will evict files from the cache
after the set time since last access has passed. The default value of
1 hour will start evicting files from cache that haven't been accessed
for 1 hour. When a cached file is accessed the 1 hour timer is reset to 0
and will wait for 1 more hour before evicting. Specify the time with
standard notation, s, m, h, d, w .

You **should not** run two copies of rclone using the same VFS cache
with the same or overlapping remotes if using `--vfs-cache-mode > off`.
This can potentially cause data corruption if you do. You can work
around this by giving each rclone its own cache hierarchy with
`--cache-dir`. You don't need to worry about this if the remotes in
use don't overlap.

### --vfs-cache-mode off

In this mode (the default) the cache will read directly from the remote and write
directly to the remote without caching anything on disk.

This will mean some operations are not possible

  * Files can't be opened for both read AND write
  * Files opened for write can't be seeked
  * Existing files opened for write must have O_TRUNC set
  * Files open for read with O_TRUNC will be opened write only
  * Files open for write only will behave as if O_TRUNC was supplied
  * Open modes O_APPEND, O_TRUNC are ignored
  * If an upload fails it can't be retried

### --vfs-cache-mode minimal

This is very similar to "off" except that files opened for read AND
write will be buffered to disk.  This means that files opened for
write will be a lot more compatible, but uses the minimal disk space.

These operations are not possible

  * Files opened for write only can't be seeked
  * Existing files opened for write must have O_TRUNC set
  * Files opened for write only will ignore O_APPEND, O_TRUNC
  * If an upload fails it can't be retried

### --vfs-cache-mode writes

In this mode files opened for read only are still read directly from
the remote, write only and read/write files are buffered to disk
first.

This mode should support all normal file system operations.

If an upload fails it will be retried at exponentially increasing
intervals up to 1 minute.

### --vfs-cache-mode full

In this mode all reads and writes are buffered to and from disk. When
data is read from the remote this is buffered to disk as well.

In this mode the files in the cache will be sparse files and rclone
will keep track of which bits of the files it has downloaded.

So if an application only reads the starts of each file, then rclone
will only buffer the start of the file. These files will appear to be
their full size in the cache, but they will be sparse files with only
the data that has been downloaded present in them.

This mode should support all normal file system operations and is
otherwise identical to `--vfs-cache-mode` writes.

When reading a file rclone will read `--buffer-size` plus
`--vfs-read-ahead` bytes ahead.  The `--buffer-size` is buffered in memory
whereas the `--vfs-read-ahead` is buffered on disk.

When using this mode it is recommended that `--buffer-size` is not set
too large and `--vfs-read-ahead` is set large if required.

**IMPORTANT** not all file systems support sparse files. In particular
FAT/exFAT do not. Rclone will perform very badly if the cache
directory is on a filesystem which doesn't support sparse files and it
will log an ERROR message if one is detected.

### Fingerprinting

Various parts of the VFS use fingerprinting to see if a local file
copy has changed relative to a remote file. Fingerprints are made
from:

- size
- modification time
- hash

where available on an object.

On some backends some of these attributes are slow to read (they take
an extra API call per object, or extra work per object).

For example `hash` is slow with the `local` and `sftp` backends as
they have to read the entire file and hash it, and `modtime` is slow
with the `s3`, `swift`, `ftp` and `qinqstor` backends because they
need to do an extra API call to fetch it.

If you use the `--vfs-fast-fingerprint` flag then rclone will not
include the slow operations in the fingerprint. This makes the
fingerprinting less accurate but much faster and will improve the
opening time of cached files.

If you are running a vfs cache over `local`, `s3` or `swift` backends
then using this flag is recommended.

Note that if you change the value of this flag, the fingerprints of
the files in the cache may be invalidated and the files will need to
be downloaded again.

## VFS Chunked Reading

When rclone reads files from a remote it reads them in chunks. This
means that rather than requesting the whole file rclone reads the
chunk specified.  This can reduce the used download quota for some
remotes by requesting only chunks from the remote that are actually
read, at the cost of an increased number of requests.

These flags control the chunking:

    --vfs-read-chunk-size SizeSuffix        Read the source objects in chunks (default 128M)
    --vfs-read-chunk-size-limit SizeSuffix  Max chunk doubling size (default off)

Rclone will start reading a chunk of size `--vfs-read-chunk-size`,
and then double the size for each read. When `--vfs-read-chunk-size-limit` is
specified, and greater than `--vfs-read-chunk-size`, the chunk size for each
open file will get doubled only until the specified value is reached. If the
value is "off", which is the default, the limit is disabled and the chunk size
will grow indefinitely.

With `--vfs-read-chunk-size 100M` and `--vfs-read-chunk-size-limit 0`
the following parts will be downloaded: 0-100M, 100M-200M, 200M-300M, 300M-400M and so on.
When `--vfs-read-chunk-size-limit 500M` is specified, the result would be
0-100M, 100M-300M, 300M-700M, 700M-1200M, 1200M-1700M and so on.

Setting `--vfs-read-chunk-size` to `0` or "off" disables chunked reading.

## VFS Performance

These flags may be used to enable/disable features of the VFS for
performance or other reasons. See also the [chunked reading](#vfs-chunked-reading)
feature.

In particular S3 and Swift benefit hugely from the `--no-modtime` flag
(or use `--use-server-modtime` for a slightly different effect) as each
read of the modification time takes a transaction.

    --no-checksum     Don't compare checksums on up/download.
    --no-modtime      Don't read/write the modification time (can speed things up).
    --no-seek         Don't allow seeking in files.
    --read-only       Only allow read-only access.

Sometimes rclone is delivered reads or writes out of order. Rather
than seeking rclone will wait a short time for the in sequence read or
write to come in. These flags only come into effect when not using an
on disk cache file.

    --vfs-read-wait duration   Time to wait for in-sequence read before seeking (default 20ms)
    --vfs-write-wait duration  Time to wait for in-sequence write before giving error (default 1s)

When using VFS write caching (`--vfs-cache-mode` with value writes or full),
the global flag `--transfers` can be set to adjust the number of parallel uploads of
modified files from the cache (the related global flag `--checkers` has no effect on the VFS).

    --transfers int  Number of file transfers to run in parallel (default 4)

## VFS Case Sensitivity

Linux file systems are case-sensitive: two files can differ only
by case, and the exact case must be used when opening a file.

File systems in modern Windows are case-insensitive but case-preserving:
although existing files can be opened using any case, the exact case used
to create the file is preserved and available for programs to query.
It is not allowed for two files in the same directory to differ only by case.

Usually file systems on macOS are case-insensitive. It is possible to make macOS
file systems case-sensitive but that is not the default.

The `--vfs-case-insensitive` VFS flag controls how rclone handles these
two cases. If its value is "false", rclone passes file names to the remote
as-is. If the flag is "true" (or appears without a value on the
command line), rclone may perform a "fixup" as explained below.

The user may specify a file name to open/delete/rename/etc with a case
different than what is stored on the remote. If an argument refers
to an existing file with exactly the same name, then the case of the existing
file on the disk will be used. However, if a file name with exactly the same
name is not found but a name differing only by case exists, rclone will
transparently fixup the name. This fixup happens only when an existing file
is requested. Case sensitivity of file names created anew by rclone is
controlled by the underlying remote.

Note that case sensitivity of the operating system running rclone (the target)
may differ from case sensitivity of a file system presented by rclone (the source).
The flag controls whether "fixup" is performed to satisfy the target.

If the flag is not provided on the command line, then its default value depends
on the operating system where rclone runs: "true" on Windows and macOS, "false"
otherwise. If the flag is provided without a value, then it is "true".

The `--no-unicode-normalization` flag controls whether a similar "fixup" is
performed for filenames that differ but are [canonically
equivalent](https://en.wikipedia.org/wiki/Unicode_equivalence) with respect to
unicode. Unicode normalization can be particularly helpful for users of macOS,
which prefers form NFD instead of the NFC used by most other platforms. It is
therefore highly recommended to keep the default of `false` on macOS, to avoid
encoding compatibility issues.

In the (probably unlikely) event that a directory has multiple duplicate
filenames after applying case and unicode normalization, the `--vfs-block-norm-dupes`
flag allows hiding these duplicates. This comes with a performance tradeoff, as
rclone will have to scan the entire directory for duplicates when listing a
directory. For this reason, it is recommended to leave this disabled if not
needed. However, macOS users may wish to consider using it, as otherwise, if a
remote directory contains both NFC and NFD versions of the same filename, an odd
situation will occur: both versions of the file will be visible in the mount,
and both will appear to be editable, however, editing either version will
actually result in only the NFD version getting edited under the hood. `--vfs-block-
norm-dupes` prevents this confusion by detecting this scenario, hiding the
duplicates, and logging an error, similar to how this is handled in `rclone
sync`.

## VFS Disk Options

This flag allows you to manually set the statistics about the filing system.
It can be useful when those statistics cannot be read correctly automatically.

    --vfs-disk-space-total-size    Manually set the total disk space size (example: 256G, default: -1)

## Alternate report of used bytes

Some backends, most notably S3, do not report the amount of bytes used.
If you need this information to be available when running `df` on the
filesystem, then pass the flag `--vfs-used-is-size` to rclone.
With this flag set, instead of relying on the backend to report this
information, rclone will scan the whole remote similar to `rclone size`
and compute the total used space itself.

_WARNING._ Contrary to `rclone size`, this flag ignores filters so that the
result is accurate. However, this is very inefficient and may cost lots of API
calls resulting in extra charges. Use it as a last resort and only with caching.

## Auth Proxy

If you supply the parameter `--auth-proxy /path/to/program` then
rclone will use that program to generate backends on the fly which
then are used to authenticate incoming requests.  This uses a simple
JSON based protocol with input on STDIN and output on STDOUT.

**PLEASE NOTE:** `--auth-proxy` and `--authorized-keys` cannot be used
together, if `--auth-proxy` is set the authorized keys option will be
ignored.

There is an example program
[bin/test_proxy.py](https://github.com/rclone/rclone/blob/master/bin/test_proxy.py)
in the rclone source code.

The program's job is to take a `user` and `pass` on the input and turn
those into the config for a backend on STDOUT in JSON format.  This
config will have any default parameters for the backend added, but it
won't use configuration from environment variables or command line
options - it is the job of the proxy program to make a complete
config.

This config generated must have this extra parameter
- `_root` - root to use for the backend

And it may have this parameter
- `_obscure` - comma separated strings for parameters to obscure

If password authentication was used by the client, input to the proxy
process (on STDIN) would look similar to this:

```
{
	"user": "me",
	"pass": "mypassword"
}
```

If public-key authentication was used by the client, input to the
proxy process (on STDIN) would look similar to this:

```
{
	"user": "me",
	"public_key": "AAAAB3NzaC1yc2EAAAADAQABAAABAQDuwESFdAe14hVS6omeyX7edc...JQdf"
}
```

And as an example return this on STDOUT

```
{
	"type": "sftp",
	"_root": "",
	"_obscure": "pass",
	"user": "me",
	"pass": "mypassword",
	"host": "sftp.example.com"
}
```

This would mean that an SFTP backend would be created on the fly for
the `user` and `pass`/`public_key` returned in the output to the host given.  Note
that since `_obscure` is set to `pass`, rclone will obscure the `pass`
parameter before creating the backend (which is required for sftp
backends).

The program can manipulate the supplied `user` in any way, for example
to make proxy to many different sftp backends, you could make the
`user` be `user@example.com` and then set the `host` to `example.com`
in the output and the user to `user`. For security you'd probably want
to restrict the `host` to a limited list.

Note that an internal cache is keyed on `user` so only use that for
configuration, don't use `pass` or `public_key`.  This also means that if a user's
password or public-key is changed the cache will need to expire (which takes 5 mins)
before it takes effect.

This can be used to build general purpose proxies to any kind of
backend that rclone supports.  


```
rclone serve ftp remote:path [flags]
```

## Options

```
      --addr string                            IPaddress:Port or :Port to bind server to (default "localhost:2121")
      --auth-proxy string                      A program to use to create the backend from the auth
      --cert string                            TLS PEM key (concatenation of certificate and CA certificate)
      --dir-cache-time Duration                Time to cache directory entries for (default 5m0s)
      --dir-perms FileMode                     Directory permissions (default 0777)
      --file-perms FileMode                    File permissions (default 0666)
      --gid uint32                             Override the gid field set by the filesystem (not supported on Windows) (default 1000)
  -h, --help                                   help for ftp
      --key string                             TLS PEM Private key
      --no-checksum                            Don't compare checksums on up/download
      --no-modtime                             Don't read/write the modification time (can speed things up)
      --no-seek                                Don't allow seeking in files
      --pass string                            Password for authentication (empty value allow every password)
      --passive-port string                    Passive port range to use (default "30000-32000")
      --poll-interval Duration                 Time to wait between polling for changes, must be smaller than dir-cache-time and only on supported remotes (set 0 to disable) (default 1m0s)
      --public-ip string                       Public IP address to advertise for passive connections
      --read-only                              Only allow read-only access
      --uid uint32                             Override the uid field set by the filesystem (not supported on Windows) (default 1000)
      --umask int                              Override the permission bits set by the filesystem (not supported on Windows) (default 2)
      --user string                            User name for authentication (default "anonymous")
      --vfs-block-norm-dupes                   If duplicate filenames exist in the same directory (after normalization), log an error and hide the duplicates (may have a performance cost)
      --vfs-cache-max-age Duration             Max time since last access of objects in the cache (default 1h0m0s)
      --vfs-cache-max-size SizeSuffix          Max total size of objects in the cache (default off)
      --vfs-cache-min-free-space SizeSuffix    Target minimum free space on the disk containing the cache (default off)
      --vfs-cache-mode CacheMode               Cache mode off|minimal|writes|full (default off)
      --vfs-cache-poll-interval Duration       Interval to poll the cache for stale objects (default 1m0s)
      --vfs-case-insensitive                   If a file name not found, find a case insensitive match
      --vfs-disk-space-total-size SizeSuffix   Specify the total space of disk (default off)
      --vfs-fast-fingerprint                   Use fast (less accurate) fingerprints for change detection
      --vfs-read-ahead SizeSuffix              Extra read ahead over --buffer-size when using cache-mode full
      --vfs-read-chunk-size SizeSuffix         Read the source objects in chunks (default 128Mi)
      --vfs-read-chunk-size-limit SizeSuffix   If greater than --vfs-read-chunk-size, double the chunk size after each chunk read, until the limit is reached ('off' is unlimited) (default off)
      --vfs-read-wait Duration                 Time to wait for in-sequence read before seeking (default 20ms)
      --vfs-refresh                            Refreshes the directory cache recursively in the background on start
      --vfs-used-is-size rclone size           Use the rclone size algorithm for Used size
      --vfs-write-back Duration                Time to writeback files after last use when using cache (default 5s)
      --vfs-write-wait Duration                Time to wait for in-sequence write before giving error (default 1s)
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

* [rclone serve](/commands/rclone_serve/)	 - Serve a remote over a protocol.
