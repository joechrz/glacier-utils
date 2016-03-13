# glacier-utils

a few tools I threw together to be able to archive directories to amazon glacier

## tree-hash

a bash implementation of their tree-hash algorithm.  usage:

```bash
tree-hash [-b] filename

  -b  print the output hash in binary form
```

## glacier-archive-prepare

prepares a directory for archiving:

 1. `tar` the directory and split the result into equally sized chunks
 2. run `tree-hash` on each chunk 
 3. run `tree-hash-reduce` on the resulting hashes to get the tree-hash of the whole archive

usage:

```
glacier-archive-prepare dirname
```

