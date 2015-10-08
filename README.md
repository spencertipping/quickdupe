# quickdupe: fast duplicate file detector
```sh
$ find some-path | ./quickdupe > dupe-list
$ find some-path | ./quickdupe --signatures > signature-per-file
$ ./deduplicate dupe-list
```

Quickdupe is designed to efficiently detect duplicate content across a large
collection of files, some of which may be hardlinked together. Its average-case
IO/time complexity is O(D + n + log N), where D is the combined size of all
duplicates, n is the number of files, and N is the combined size of distinct
inodes (so not counting any redundant hardlinks). Space complexity is O(n).

Internally, quickdupe uses repeated partitioning to differentiate files from
each other. Its initial partition is file size, then SHA-256 signatures of
exponentially-increasing ranges of data. Anytime a file is isolated from
others, it is removed from the list of possible duplicates.

Quickdupe operates on inodes rather than files, which means that it won't flag
hardlinked files as duplicates of each other. When two inodes have the same
contents, all aliased filenames are mentioned in the list of duplicates; this
way the deduplicator knows to relink all of the filenames to the same
underlying data. (Otherwise you could still have duplicates, since you wouldn't
be guaranteed to zero one of the inodes' reference counts.)

## The `--signatures` option
Written for @arrdem. The idea is that you want a short content-based name for
each of a series of files, but you don't want to SHA them all. That
content-based name is guaranteed to be unique with respect to the other files
being deduplicated, though it's not stable; i.e. if you later rerun with a
conflicting file, that signature will change.

```sh
$ ls | quickdupe --signatures 2>/dev/null
5012	quickdupe
1078	deduplicate
2367	README.md
```

## Caveats
Neither quickdupe nor deduplicate look for any of the following
possibly-problematic situations:

1. Differences in file ownership or permissions.
2. Differences in file suid status (very important; be sure not to deduplicate
   /usr/bin/sudo for this reason).
3. Files across different devices -- any such duplicates will be detected even
   though there is no way to merge them.
4. Files that contain tab or newline characters.

As a result, you should be careful when using this system on your data; I
recommend reading through the dupe-list TSV before running deduplicate.
