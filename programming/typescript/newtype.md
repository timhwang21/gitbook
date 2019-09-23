# `newtype`

In Haskell, a `newtype` is a construct that allows you to create a nominal type from another type. (Recall that Haskell and Typescript are both [structurally typed](https://en.wikipedia.org/wiki/Structural_type_system).) One use case is to allow differentiation of two values with the same underlying type, where one value needs to follow a separate set of constraints.

At [Nash](https://nash.io), we have to deal with two types of currency symbols, one for fiat and one for cryptocurrency tokens. These are represented under the hood as `strings`; however, we have several functions that want to ensure they only receive either fiat symbols or crypto symbols. By using nominal types, we can pass around values that are really strings and have functions check if they are the right "kind" of string, by tagging these strings with some hidden type information. This prevents us from making silly errors like passing a fiat symbol to a function that expects a crypto symbol.

Notably, besides the actual validation function, this logic takes place entirely in the type system, and has no runtime overhead.

Note that this is just one of many use cases for `newtype`s. Also note that this pattern doesn't _actually_ allow you to enforce invariants about the derived types. For that, it might be better to _actually_ make your `newtype` a separate type altogether, with its own internal logic. (For example, you can implement the type as a traditional OOP class with methods, or as an ADT with a functor, etc. instances for manipulation.)

```typescript
declare const Unique: unique symbol;
type NewType<T, Tag> = t & { [Unique]: Tag };

type CryptoCurrency = NewType<string, "CryptoCurrency">;

function getAsCrypto(value: string): CryptoCurrency | never {
  if (checkIfValueIsValidCryptoSymbol(value)) {
    return value as CryptoCurrency;
  } else {
    throw new Error("not a valid cryptocurrency symbol");
  }
}
```

## Breaking it down

```typescript
declare const Unique: unique symbol;
type NewType<T, Tag> = t & { [Unique]: Tag };
```

Here we create a utility for easily generating these nominal types. The `[Unique]` object key is a trick for hiding the tag from an editor's autocomplete (credits to [Dan Freeman's great article](https://dfreeman.io/whats-in-a-name/) for this trick).

```typescript
type CryptoCurrency = NewType<string, "CryptoCurrency">;
```

Here we are declaring our `type CryptoCurrency` which is really just a `string`, but will be handled nominally rather than structurally, such that if we try to pass a `string` to a function that expects a `CryptoCurrency`, we will get a type error. Contrast this with `export type CryptoCurrency = string`, which would be like the Haskell type synonym `type CryptoCurrency = String` instead of `newtype CryptoCurrency = String`.

```typescript
function getAsCrypto(value: string): CryptoCurrency | never {
  if (checkIfValueIsValidCryptoSymbol(value)) {
    return value as CryptoCurrency;
  } else {
    throw new Error("not a valid cryptocurrency symbol");
  }
}
```

This function takes a string and either returns the string coerced to a `CryptoCurrency`, or throws an error. Note that when transpiled to Javascript, the function's non-error case is actually a no-op -- the `newtype` lives entirely in the type system.

## Resources

* [`newtype` in Typescript](https://www.everythingfrontend.com/posts/newtype-in-typescript.html)
* [Nominal types in Typescript](https://dfreeman.io/whats-in-a-name/)
* [Pros and cons of `newtype`](http://degoes.net/articles/newtypes-suck)
