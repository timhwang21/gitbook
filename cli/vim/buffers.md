# Buffers

## Close all but the current buffer

```
:%bd|e#|bd#
```

In English:

* `%bd` - close all buffers
* `e#` - open last buffer
* `bd#` - close last `[No Name]` buffer that's automatically created when all buffers are closed
