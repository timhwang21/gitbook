# Batch editing

## Opening a list of files

```bash
vim `ag -lQ TERM`
```

## Operating on a list of files within vim using `cdo`, `cfdo`

The following takes an existing quickfix list \(for example, from `:Ack TERM`\), applies the replacement, and saves them.

`cdo` runs the command for each entry in the quickfix list while `cfdo` runes the command for each file in the quickfix list.

```
:cfdo %s/TERM/REPLACEMENT/g
```

## Operating on a list of files using argument list

### Batch editing files outside vim using `argdo`

The following command takes a list of files including `TERM`, applies the replacement, and saves them.

```bash
vim `ag -lQ TERM` +"argdo %s/TERM/REPLACEMENT/g | update"
```

### Bulk editing files from within vim

The following allows adding all files matching a glob to the arglist, where you can then operate on them. This example uses `:exec` and string concatenation to open all files matching the extension `.tsx` in the parent directory of the current file.

```
:exec 'args ' . expand('%:h') . '/**.tsx'
```
