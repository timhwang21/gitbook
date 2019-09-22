# External commands

## `%` and `!`

`%` resolves to the current filename. `!` calls an external function. Together, `%!` executes a command over the file, and replaces the current buffer with the output.

## Mapping over files with `:w`

You can generate derived JSON from an existing file with `jq`, which I sometimes use to generate and manipulate test vectors:

```sh
:w !jq '.key-to-filter-by' > filtered.json
```

You can easily pass Markdown (or any other text format) files straight through `pandoc`. This is useful for exporting documentation for sharing with non-engineers.

```sh
:w !pandoc -o FILENAME.pdf
```

## Buffer processing with `:%:`

Processing can be achieved by calling `%! external_fn` to replace buffer contents in-place. In this example, we pipe a JSON file through `jq` to prettify.

```sh
%: jq '.'
```

