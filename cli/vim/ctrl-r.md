# `<Ctrl-r>`

In insert and command mode, `<Ctrl-r>` followed by a sequence inserts an object.

Insert mode usage mainly deals with manipulating registers. However, in command mode, `<Ctrl-r><Ctrl-*>` inserts special objects for certain `*`:

- `F`: Current filename under cursor
- `P`: Current filename with expanded path under cursor
- `W`: Word under cursor
- `A`: WORD under cursor
- `L`: Line under cursor

For more, `:help c-^r`.

## Insert word under cursor

Very useful with `:Ag`, or when doing find/replace.

```sh
:Ag <Ctrl-r><Ctrl-w>
:%s/<Ctrl-r><Ctrl-w>/replacement-string
```
