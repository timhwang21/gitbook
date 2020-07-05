# :global

`:s` \(`:substitute`\) is one of the first commands any Vim user learns. There are several similar commands that are more generic. Like `:s`, these commands can take a range; however, unlike `:s`, there is no `%` variant, as these commands are global by nature.

These commands can be seen as a simpler, less general-purpose alternative to macros that takes less keypresses.

## Commands

### `:g` \(`:global`\)

Instead of substituting strings matching the first part with the second, runs the \(command mode\) command on strings matching the first part.

### `:v` \(`:vglobal`\) \(equivalent to `:g!`\)

Same as `:g`, but targets strings that do NOT match.

## Use cases

### Delete lines containing pattern

```bash
g/pattern/d
```

Practical usage:

```bash
g/TODO:/d
g/debugger/d
g/binding.pry/d
```

Optimization: `g/pattern/_d` puts deleted lines in the null register, which has no performance cost.

### Conditional find/replace

```bash
g/pattern/s/word/newword
```

This "scopes" the substitution to matching lines \(for example, to scope your substitution to comments\).

### Yank all matches

```bash
g/pattern/y A
```

Note that we are yanking to an uppercase register, meaning we append matches. If we did `y a`, each yank would overwrite the previous yank.

### `:norm`

```bash
g/pattern/norm ^ihello
```

This applies the normal mode instruction \(here, "go to start and insert hello"\) to matching lines.

Practical usage:

```bash
g/debugger/norm gcc
g/pattern/norm @q
```

### `:exe`

```bash
g/pattern/exec "norm!" "A\r"
```

When using `:norm`, you might notice that you can't insert linebreaks -- `\r` and `<CR>` is literally printed as the strings "\r" and "". `:exec(ute)` lets us actually pass in commands as strings to be executed, like `eval()`.

This is a somewhat meta pattern that follows the last example. It executes the two separate commands `"norm!"` and `"A\r"` to the pattern. Before execution, the second is evaluated to "append carriage return."

