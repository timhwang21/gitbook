# Replace JSX tag

> (Note: this is redundant if you are using an editor with LSP support, where you can rename symbols using the language server instead of regex!)

The following (Vim-regex flavored) `sed` string replaces a JSX tag name while maintaining children and props:

```sed
%s/\v\<Tag(%(.|\n){-})\>(%(.|\n){-})\<\/Tag\>/\<NewTag\1\>\2\<\/NewTag\>/g
```

This is a very powerful regex that can be used to rename all instances of a component, **as long as it never occurs as a child of itself**. For example, this can be combined with `argdo` to refactor an entire codebase.

## Caveat

Note that this will fail when ran on a tag that appears as a child of itself, because the closing tag of the inner child will be matched. For this reason, this trick does not work well with many nested `<div/>`s. E.g.:

```jsx
<Tag prop1="value">
  <NewTag>
    <Tag prop2="value">
      <Baz/>
    </Tag>
  </NewTag>
</Tag>
```

## Breaking it down

#### `%s/\v`

Global match with the "very magic" flag (`:h magic`). Because we use so many capture groups, this makes the regex much more readable (though still pretty unreadable).

#### `\<Tag(%(.|\n){-})\>`

(Note we have to escape `<` and `>`. Annoying.)

We are matching the tag `<Tag/>` with a capture group (first set of parentheses) that consists of any character plus the newline character (`%(.|\n)`) that occur 0 or more times. The following `{-}` is Vim regex's version of `*?` (making 0+ match nongreedy), and the `%` is Vim regex's version of a non-capturing group (since we do not care about this inner group for interpolation). This capture group represents the **component's props**.

#### `(%(.|\n){-})\<\/Tag\>/`

This is an exact repeat of the "match all or newline 0 or more times nongreedily" from above, and the component's closing tag. This capture group represents the **component's children**. For a self-closing component, simply use `\/\>` here and leave out the children capture group.

#### `\<NewTag\1\>\2\<\/NewTag\>/g`

Now we get to the replace step in the statement. We replace `<Tag/>` with `<NewTag/>`, and interpolate the previous two capture groups (props and children). Similarly, if using a self-closing tag, use `\<NewTag\1\>\2\/\>/g`

## Demo

```sh
vim `ag -l "<Tag\W"` +"argdo %s/\v\<Tag(%(.|\n){-})\>(%(.|\n){-})\<\/Tag\>/\<NewTag\1\>\2\<\/NewTag\>/g | update"
```

