# Batch editing

## Opening a list of files

```sh
vim `ag -lQ TERM`
```

## Operating on a list of files outside vim using `argdo`

The following command takes a list of files including `TERM`, applies the replacement, and saves them.

```sh
vim `ag -lQ TERM` +"argdo %s/TERM/REPLACEMENT/g | update"
```

## Operating on a list of files within vim using `cdo`, `cfdo`

The following takes an existing quickfix list (for example, from `:Ack TERM`), applies the replacement, and saves them.

`cdo` runs the command for each entry in the quickfix list while `cfdo` runes the command for each file in the quickfix list.

```sh
:cfdo %s/TERM/REPLACEMENT/g
```
