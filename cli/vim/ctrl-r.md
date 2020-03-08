# `<Ctrl-r>`

In insert and command mode, `<Ctrl-r>` followed by a sequence inserts an object.

Insert mode usage mainly deals with manipulating registers. However, in command mode, `<Ctrl-r><Ctrl-*>` inserts special objects for certain `*`:

- `F`: Current filename
- `P`: Current filename with expanded path
- `W`: Word under cursor
- `A`: WORD under cursor
- `L`: Line under cursor

For more, `:help c-^r`.
