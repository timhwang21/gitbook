# OmitTypes

One frequent annoyance that comes up in Typescript is redundant null checks. Take the following interface, which describes a standard JSONAPI resource:

```typescript
interface ApiResource<T, A, R> {
  type: T;
  id: string;
  attributes: A;
  relationships: R;
}
```

A resource must have a `type`, and may have `attributes` and `resources`. For example, we can define a `User` with attribute `name: string` and relationships to `Post`s and `Comment`s:

```typescript
type User = ApiResource<
  'user',
  {
    name: string;
  },
  {
    posts: Post[];
    comments: Comment[];
  }
>;
```

However, when working with resources that do not have attributes or relationships, our naive interface above is cumbersome to use, as our generic requires us to supply `attributes` and `relationships` even when they do not exist. Ideally, we want our generic to be flexible, while being ergonomic to use.

## Approach 1: optional properties

One alternative would be to mark these as optional, which seems reasonable because as said above, not every resource has these properties.

```typescript
interface ApiResource<T, A, R> {
  type: T;
  id: string;
  attributes?: A;
  relationships?: R;
}
```

However, this nullishness bleeds into our resources for which these properties exist:

```typescript
type User = ApiResource<'user', UserAttributes, UserRelationships>;

const user: User = fetchUser();

// I want to access my user's name...
user.attributes.name;
// => TypeError: Object is possibly undefined.
```

This necessitates a redundant null check: the type system thinks that `attributes` might not exist, though according to our schema, every `User` should always have `attribute`s. As a result, the type system is hindering rather than helping us.

## Approach 2: default type arguments

Alternatively, we can set the trailing generic arguments to be set to `null`:

```typescript
interface ApiResource<T, A = null, R = null> {
  type: T;
  id: string;
  attributes: A;
  relationships: R;
}
```

This solves the previous problem of needing to pass types: if a resource lacks attributes or relationships, we simply pass nothing and our the type will default to `null`. However, we now run into another hindrance when trying to create instances of a resource that do not have `attributes` or `relationships`:

```typescript
type UserWithNoRelationships = ApiResource<'user', UserAttributes, null>;

// I want to create a new user instance
const newUser: UserWithNoRelationships = {
  type: 'user' as const,
  id: 1,
  attributes: {
    name: 'John',
  },
};
// => TypeError: Type is missing the property 'relationships'
```

Again, we are fighting with the type system: we are being asked to supply a value that we know via our scheme will never exist.

This can be bypassed by either explicitly passing `relationships: null` \(which is cumbersome\), or by declaring my user as a `Partial<ApiResource<...>>`, which loses type information for other properties we want to maintain as non-nullable. We could write a helper type `MarkKeysAsOptional<T>`, which isn't a bad solution, but leaves us with two variants of the same type \(`User` and `UserWithOnlyRequiredProperties`\), which complicates our type system.

In a way, these two problems are opposites of each other. How can we construct our type in a way that fulfills both these use cases? More abstractly, how can we write a type constructor that omits "empty" keys from some other type?

## Approach 3: `OmitTypes`

We can accomplish this using mapped types and several intermediate types:

```typescript
/**
 * Takes an object type and a type condition, and returns a new object whose
 * value is either equal to the key or never, based on whether or not the value
 * extends the type of the condition.
 */
type NeverIfMatch<T, Cond> = {
  [P in keyof T]: T[P] extends Cond ? never : P
};

/**
 * Returns all non-`never` keys of a type passed through `NeverIfMatch`.
 */
type FilteredKeys<T, Cond> = NeverIfMatch<T, Cond>[keyof T];

/**
 * The final type. Takes a type, and filters out all keys whose values extend
 * the type `Cond`.
 */
type OmitTypes<T, Cond> = Pick<T, FilteredKeys<T, Cond>>;
```

Using `OmitTypes`, we can create type constructors that omit keys from interfaces based on the type of their values. We can now write a version of `ApiResource` that handles both our cases:

```typescript
type ApiResource<T, A = null, R = null> = OmitTypes<{
  type: T;
  id: string;
  attributes: A;
  relationships: R;
}, null>;
```

If `A` and `R` are not passed, the resultant type will entirely omit the keys `attributes` and `relationships`. This is incredibly useful when our resources are modeled as tagged unions, as we never have to null check these properties if our types are properly defined. If we see that `resource.type` corresponds to a resource which has attributes, the type system will know that `resource.attributes` _ALWAYS_ exists, and we don't have to check it. Conversely, when creating instances of a resource which doesn't have attributes or relationships, we don't have to specify these keys, as the type system knows these will _NEVER_ exist.

Importantly, notice that our `null`s are pushed to the absolute boundary of the type, where they do not "infect" our entire union of resources.

There are many other use cases for `OmitTypes`. For example, one could collect all event handlers in an interface of React props by filtering out props that aren't functions matching a specific interface. A reverse of `OmitTypes` can also be trivially constructed by replacing `Pick` with `Omit`. Below are some sample derived types:

```typescript
/**
 * Filters out all nullish types.
 */
type OmitNullish<T> = OmitTypes<T, undefined | null>

/**
 * Filters out "empty" types.
 */
type OmitEmpty<T> = OmitTypes<T, undefined | null | {}>
```

If you know of other potentially useful use cases, please let me know! [Or you can directly file a PR here.](https://github.com/timhwang21/gitbook)

{% hint style="info" %}
I've since found that this \(and other handy types\) can be found in the library [ts-essentials](https://github.com/krzkaczor/ts-essentials#OmitProperties). Check it out!
{% endhint %}

