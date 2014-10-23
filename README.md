# quickdupe: fast duplicate file detector
```sh
$ find some-path | ./quickdupe > dupe-list
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
