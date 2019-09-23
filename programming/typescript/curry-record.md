# `curryRecord`

This is an example of a function whose logic lives in the type definition rather than in the function body.

```typescript
/**
 * @name `curryRecord`
 *
 * Takes a unary function that takes a record as an argument, and makes the
 * record partially applicable. Returns a new function that takes a partial of
 * the record, which itself returns a new function that takes only the missing
 * properties of the record.
 *
 * Can be thought of as `curry()` that curries a record as opposed to a list of
 * arguments.
 */
export const curryRecord = <F extends (x: Record<any, any>) => any>(fn: F) => <
  T extends Partial<Parameters<F>[0]>
>(
  a: T
) => (b: Omit<Parameters<F>[0], keyof T>): ReturnType<F> =>
  fn({ ...a, ...b } as Parameters<F>[0]);
```

## Breaking it down

#### `export const curryRecord = <F extends (x: Record<any, any>) => any>(fn: F) => ...`

The first argument is a unary function. We use the generic `<F extends...>` to enforce arity and to enforce the passed function's argument being a record. The exact type of `x` and the return type will be inferred by Typescript once a function is actually passed.

#### `<T extends Partial<Parameters<F>[0]>>(a: T) => ...`

Once we pass our unary function we get a new function that takes `a: T`. `T` must be assignable to `Partial<Parameters<F>[0]>`, which means "a partial version of whatever record your unary function expects as an argument."

#### `(b: Omit<Parameters<F>[0], keyof T>): ...`

Once we pass our partial record, we get a new function that takes a record `b` that must contain all keys of `Parameters<F>[0]` that were NOT included in `a`. It checks this by omitting all keys of `a` from `Parameters<F>[0]`: the remaining keys are the keys that must be passed as a part of `b`.

#### `ReturnType<F>`

The return type of the curried function should match the return type of the function we're currying in the first place. We don't know what this is, so we use `ReturnType<F>` to infer the proper type once a function is actually passed.

#### `... => fn({ ...a, ...b } as Parameters<F>[0])`

We trivially spread both partial arguments into the original function. We have to coerce this here because Typescript can't figure out that the above constraints means that `{ ...a, ...b }` will always be of type `Parameters<F>[0]`.

Interestingly enough, once compiled to Javascript this function becomes `curryRecord => fn => a => b => fn({ ...a, ...b })`, which does absolutely zero validation. All the logic is handled by the type system!

## Demo and test cases

```typescript
interface FooBar {
  foo: string;
  bar: string;
}

const handleFooBar = (foobar: FooBar): string => foobar.foo + foobar.bar;

// Works trivially
handleFooBar({
  foo: "foo",
  bar: "bar"
});

// TypeError: Property "bar" is missing
handleFooBar({
  foo: "foo"
});

// Returns a version of handleFooBar ready for partialization
const curryRecordHandleFooBar = curryRecord(handleFooBar);

// Returns a function that only allows the missing properties
const handleFooBarWithFooProvided = curryRecordHandleFooBar({
  foo: "foo"
});

// TypeError: "foo" is not assignable to Omit<FooBar, 'foo'>
handleFooBarWithFooProvided({ foo: "foo2" });

// TypeError: Property "bar" is missing
handleFooBarWithFooProvided({});

// Works!
handleFooBarWithFooProvided({ bar: "bar" });

// There are no missing properties, hence this is called with an empty record
const handleFooBarWithAllKeysProvided = curryRecordHandleFooBar({
  foo: "foo",
  bar: "bar"
});

handleFooBarWithAllKeysProvided({});

// TypeError: rejects functions that do not take records
curryRecord((x: number) => 123);

// TypeError: rejects non-unary functions
curryRecord((x: Record<any, any>, y: Record<any, any>) => 123);
```
