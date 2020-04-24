# Safe JSON clone

Calling `JSON.serialize` and then `JSON.parse` on a value is the fastest general-purpose way to deeply clone an object in Javascript. However, the downside is that it only works on values for which `x => JSON.parse(JSON.serialize(x))` is the identity function. In other words, this function will only work correctly on strings, numbers, booleans, `null`, `undefined`, and arrays and objects built off of these types.

It's common to choose a function name or leave comments warning about this constraint. However, in Typescript we can enforce this constraint at the type level using a recursive type:

```typescript
// note that the primitives Symbol and BigInt are not serializable
type FastClonablePrimitive = string | number | null | undefined | boolean

type FastClonable =
  | FastClonablePrimitive
  | FastClonable[]
  | { [x: string]: FastClonable }

const fastClone = <T extends FastClonable>(x: T): T = JSON.parse(JSON.serialize(x))
```

Now, when passing any value that will not be transparently cloned via this function, Typescript will emit an error.

```typescript
// valid
fastClone(123)
fastClone([123, true, 'foo'])
fastClone({ foo: 123 })

// invalid
fastClone(Symbol('foo'))
fastClone(123n)
fastClone(new Foo())
```

This also works generically. However, note that this approach does not work when passed an interface, [which is due to an intentional Typescript design decision](https://github.com/microsoft/TypeScript/issues/15300#issuecomment-332366024). (Note that this may change in future versions of Typescript.)

```typescript
type FooType = {
  bar: {
    baz: number
  }
}

const myFoo: FooType = {
  bar: {
    baz: 123
  }
}

// correctly returns a value with type FooType
fastClone(myFoo)

interface FooInterface {
  bar: {
    baz: number
  }
}

const myFoo2: FooInterface = {
  bar: {
    baz: 123
  }
}

// Argument of type 'FooInterface' is not assignable to parameter of type 'FastClonable'.
//   Type 'FooInterface' is not assignable to type '{ [x: string]: FastClonable; }'.
//     Index signature is missing in type 'FooInterface'.
fastClone(myFoo2)
```
