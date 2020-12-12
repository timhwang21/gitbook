# Registers

## Special registers

`@` followed by a register symbol represents a register. In addition to the standard numbered registers `@[0-9]`, there are several special registers.

* `:`: command register
* `/`: search register
* `%`: filepath register
* `"`: last yanked value register
* `+`: system clipboard register
* `=`: expression register

`:reg` for a complete list.

## Modify register contents with `<Ctrl-r>REGISTER`

Note that since macros are just command sequences stored as a string in a register, you can edit macros this way as well.

```
:let @q='<Ctrl-r>q' # ...and then modify
```

## Reuse contents of search register

Useful for when you search for something unwieldy to type and need to reuse it later \(for a replace for example\).

```
/some-long-and-complicated-search-string # search something arcane
:%s/<Ctrl-r>//replacement-string # reuse without retyping
```

## Yank to system clipboard

```
"+y
```

## Set system clipboard contents to current filepath

```
:let @+ = expand("%")
```

## Run function on search results

Useful for transformations. The following will dashcase all capture group matches using [abolish.vim](https://github.com/tpope/vim-abolish).

```
g/\attribute :(\w\+\)/\=g:Abolish.dashcase(submatch(0))
```
