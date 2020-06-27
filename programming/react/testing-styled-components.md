# Testing Styled Components

## Wrapper components

The `styled()` function will wrap the target in several intermediate React nodes. This is annoying when used with Enzyme's `mount()` because if you are finding nodes using a `data-test` attribute or something similar, you will get multiple matches because Styled Components forwards all props on the intermediate nodes.

To avoid this, call [`.hostNodes()`](https://enzymejs.github.io/enzyme/docs/api/ReactWrapper/hostNodes.html) on the `find` result to strip out all non-DOM nodes.

This solves two common cases:

1. Asserting there are n nodes that match your selector (you will not count the wrapper nodes)
2. Finding a node to simulate events on (you will not find the wrapper nodes)

As a side note, you should definitely not find by DOM node type in your tests. The actual node type is almost always an implementation detail, and having this be part of your assertion increases test fragility and makes refactoring difficult.

As another side note, if you use `styled()` to wrap a non-DOM node component that doesn't properly forward the data attribute, you will still be able to find this attribute in tests. This can lead to false matches in cases where the wrapped component will conditionally render the node that ultimately receives the data attribute, because the wrapper node will always have the attribute.
