# glacier-utils

a few tools I threw together to be able to archive directories to Amazon Glacier

## tree-hash

a bash implementation of their tree-hash algorithm. 

**Usage:**
```
tree-hash [-b] filename

  -b  print the output hash in binary form
```

tree-hash contains 2 "subcommands" that can be run individually:

### tree-hash-reduce

Implements the 'merge level' portion of the tree hash algorithm.  Given
a starting directory of leaf nodes, will combine 2 hashes at a time, and 
repeat this process until only 1 hash is left over.

**Usage:**
```
tree-hash-reduce directory
```

A common task is building the root treehash from a list of subnode hashes.
This is implemented in the `tree-hash` command as:

```bash
mkdir -p hashes
cp hash.* hashes
tree-hash-reduce hashes
xxd -ps -c 100 "hashes/$(/bin/ls -1 hashes | head -n 1)"
```

### tree-hash-combine

**Almost never need to run this directly**.

Takes 2 binary hash files (`left-file` and `right-file`), concatenates the values
together and sha256 hashes the result.  The resulting hash is written back to `left-file`
and `right-file` is deleted.

**Usage:**
```
tree-hash-combine left-file right-file
```

## glacier-archive

Archives a directory to Amazon Glacier.

**Usage:**
```
glacier-archive dirname
```

### glacier-archive-prepare

Prepares a directory for archiving:

 1. creates an `archive.$dirname` sidecar directory that holds the prepared data
 2. `tar` the directory and split the result into equally sized chunks
 3. run `tree-hash` on each chunk 
 4. run `tree-hash-reduce` on the resulting hashes to get the tree-hash of the whole archive

**Usage:**

```
glacier-archive-prepare dirname
```

### glacier-archive-upload

Uploads a directory (after running `Glacier-archive-prepare` to Glacier using
the multi-part uploader.  After uploading, autotags the original (unprepared)
directory with the metadata returned from the Glacier API.

**Usage:**
```
glacier-archive-upload dirname
```

**Note:** `glacier-archive-upload` should take the same `dirname` argument as
`glacier-archive-prepare` and *not* the prepared `archive.*` directory.
