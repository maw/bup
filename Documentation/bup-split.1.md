% bup-split(1) Bup %BUP_VERSION%
% Avery Pennarun <apenwarr@gmail.com>
% %BUP_DATE%

# NAME

bup split - save individual files to bup backup sets

# SYNOPSIS

bup split [-r *host*:*path*] <-b|-t|-c|-n *name*> [-v] [-q]
  [--bench] [--max-pack-size=*bytes*]
  [--max-pack-objects=*n*] [--fanout=*count] [filenames...]

# DESCRIPTION

`bup split` concatenates the contents of the given files
(or if no filenames are given, reads from stdin), splits
the content into chunks of around 8k using a rolling
checksum algorithm, and saves the chunks into a bup
repository.  Chunks which have previously been stored are
not stored again (ie. they are "deduplicated").

Because of the way the rolling checksum works, chunks
tend to be very stable across changes to a given file,
including adding, deleting, and changing bytes.

For example, if you use `bup split` to back up an XML dump
of a database, and the XML file changes slightly from one
run to the next, nearly all the data will still be
deduplicated and the size of each backup after the first
will typically be quite small.

Another technique is to pipe the output of the `tar`(1) or
`cpio`(1) programs to `bup split`.  When individual files
in the tarball change slightly or are added or removed, bup
still processes the remainder of the tarball efficiently. 
(Note that `bup save` is usually a more efficient way to
accomplish this, however.)

To get the data back, use `git-join`(1).

# OPTIONS

-r, --remote=*host*:*path*
:   save the backup set to the given remote server.  If
    *path* is omitted, uses the default path on the remote
    server (you still need to include the ':')
    
-b, --blobs
:   output a series of git blob ids that correspond to the
    chunks in the dataset.

-t, --tree
:   output the git tree id of the resulting dataset.
    
-c, --commit
:   output the git commit id of the resulting dataset.

-n, --name=*name*
:   after creating the dataset, create a git branch
    named *name* so that it can be accessed using
    that name.  If *name* already exists, the new dataset
    will be considered a descendant of the old *name*. 
    (Thus, you can continually create new datasets with
    the same name, and later view the history of that
    dataset to see how it has changed over time.)
    
-v, --verbose
:   increase verbosity (can be used more than once).

-q, --quiet
:   disable progress messages.

--bench
:   print benchmark timings to stderr.

--max-pack-size=*bytes*
:   never create git packfiles larger than the given number
    of bytes.  Default is 1 billion bytes.  Usually there
    is no reason to change this.

--max-pack-objects=*numobjs*
:   never create git packfiles with more than the given
    number of objects.  Default is 200 thousand objects. 
    Usually there is no reason to change this.
    
--fanout=*numobjs*
:   when splitting very large files, never put more than
    this number of git blobs in a single git tree.  Instead,
    generate a new tree and link to that.  Default is
    4096 objects per tree.

# EXAMPLE
    
    $ tar -cf - /etc | bup split -r myserver: -n mybackup-tar
    tar: Removing leading /' from member names
    Indexing objects: 100% (196/196), done.
    
    $ bup join -r myserver: mybackup-tar | tar -tf - | wc -l
    1961
    

# SEE ALSO

`bup-join`(1), `bup-index`(1), `bup-save`(1)

# BUP

Part of the `bup`(1) suite.
