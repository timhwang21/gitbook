# `:global`

`:s` (`:substitute`) is one of the first commands any Vim user learns. There are several similar commands that are more generic. Like `:s`, these commands can take a range; however, unlike `:s`, there is no `%` variant, as these commands are global by nature.

These commands can be seen as a simpler, less general-purpose alternative to macros that takes less keypresses.

## Commands

### `:g` (`:global`)

Instead of substituting strings matching the first part with the second, runs the (command mode) command on strings matching the first part.

### `:v` (`:vglobal`) (equivalent to `:!g`)

Same as `:g`, but targets strings that do NOT match.

## Use cases

### Delete lines containing pattern

```sh
g/pattern/d
```

Practical usage:

```sh
g/TODO:/d
g/debugger/d
g/binding.pry/d
```

Optimization: `g/pattern/_d` puts deleted lines in the null register, which has no performance cost.

### Conditional find/replace

```sh
g/pattern/s/word/newword
```

This "scopes" the substitution to matching lines (for example, to scope your substitution to comments).

### Yank all matches

```sh
g/pattern/y A
```

Note that we are yanking to an uppercase register, meaning we append matches. If we did `y a`, each yank would overwrite the previous yank.

### `:norm`

```sh
g/pattern/norm ^ihello
```

This applies the normal mode instruction (here, "go to start and insert hello") to matching lines.

Practical usage:

```sh
g/debugger/norm gcc
g/pattern/norm @q
```

