# Type inferrers

(Inspired by [this Reddit post](https://reddit.com/r/typescript/comments/fu5kkg/how_to_infer_callback_function_return_type/fmbispg/).)

We often have generic types that take a sum type as the type argument. For example, here is a type that creates a map of handlers for HTTP error codes:

```typescript
type StatusCodeHandler<Code extends number> = Record<Code, () => void>

const statusCodeHandlers: StatusCodeHandler<200 | 404 | 500> = {
  200: () => console.log('ok'),
  404: () => console.log('not found'),
  500: () => console.log('whoops'),
}
```

(This is a trivial example, but you can imagine a more complex type that uses `Code` in several places.)

The above implementation works fine, but it is redundant and annoying to have to supply `<200 | 404 | 500>` to both the type argument and the value.

## Alternative 1: entirely inferred type

We _could_ rely on Typescript's structural typing and let the entire type be inferred:

```typescript
const statusCodeHandlers = {
  200: () => console.log('ok'),
  404: () => console.log('not found'),
  500: () => console.log('whoops'),
}
// Inferred type:
// const statusCodeHandlers: {
//   200: () => void;
//   404: () => void;
//   500: () => void;
// }
// Seems correct...
```

However, we lose the known keys constraint:

```typescript
// This should error, but it doesn't!
const statusCodeHandlers = {
  200: () => console.log('ok'),
  404: () => console.log('not found'),
  500: () => console.log('whoops'),
  yabba: () => console.log('dabba doo'),
}
```

We also likely want to enforce the correctness of other properties as well for more complex types.

## Alternative 2: inferred type argument

We can try to have Typescript infer the type argument of the generic type. However, this is not permitted in Typescript.

```typescript
// Error:
// Generic type 'StatusCodeHandler' requires 1 type argument(s).
const statusCodeHandlersNoInference: StatusCodeHandler = {
  200: () => console.log('ok'),
  404: () => console.log('not found'),
  500: () => console.log('whoops'),
}
```

## Alternative 3: type inferrer

The above example didn't work because type argument inference only works for functions, not types.

However, we can trick Typescript into allowing inference for types by using a *type inferrer function*:

```typescript
type StatusCodeHandler<Code extends number> = Record<Code, () => void>
// This is just an identity function!
const StatusCodeHandler = <Code extends number>(x: StatusCodeHandler<Code>) => x

const statusCodeHandlersInferred = StatusCodeHandler({
  200: () => console.log('ok'),
  404: () => console.log('not found'),
  500: () => console.log('whoops'),
})
// Inferred type:
// const statusCodeHandlersInferred: Record<200 | 404 | 500, () => void>
// Works!
```

This works because the inferrer is a generic function that forwards its type argument to the generic type in its argument, and Typescript allows generic functions to omit the type parameter and tries its best to elide the type.

Note that we can define `StatusCodeHandler` as both a type and a value because in Typescript (and in most statically typed languages), types and values live in a separate namespace. This is convenient, because it allows for both the type and its inferrer to be imported in a single statement.

One downside to this approach is that it uses a fairly abstruse technique to accomplish a fairly common task, which can lead to confusion in others reading the code down the line. However, when working with ADTs with a long list of types spanning multiple lines, it can make an utterly illegible signature easier to understand.

{% hint style="info" %}
A logical next step would be to write a generalized `inferType` function:

```typescript
const inferType = <Generic, Arg>(x: Generic<Arg>) => x
```

Unfortunately, this does not work, as [Typescript does not (currently) allow generics to be used in generics](https://github.com/Microsoft/TypeScript/issues/1213).

This is a _very_ notable discussion that has been ongoing for over 5 years. If this is ever possible in Typescript, higher-kinded types will become natively available, and typeclasses like `Functor<T<~>>` and `Monad<T<~>>` will become easy to implement. (HKTs have been implemented in Typescript in the excellent [`fp-ts`](https://github.com/gcanti/fp-ts) package; however, the package is built upon an incredibly complex foundation of types.)
{% endhint %}
