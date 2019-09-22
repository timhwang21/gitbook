# `ag`

## Filename matching

* `-l`: return output as list of filenames
* `-Q`: parse literally, not as regexp
* `-g`: return filenames matching pattern; note this must be separately passed at the end, or the last flag

You can use it to operate on a list of related files by matching a directory, or to "fuzzy find" a file without needing to supply the entire path.

In all the following tips, you can leave out `-g` to instead search within the file. This is generally useful only if your search string is very specific.

```sh
# edit all matching files
vim `ag -lQg FILENAME`

# diff all matching files on the current branch against master
vimdiff master `ag -lQg FILENAME`

# plumb file in git without typing entire path
git log -p `ag -lQg FILENAME`
```
