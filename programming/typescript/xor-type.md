# XOR type

Generally, when dealing with a record value that may have one of several sets of keys, a [discriminated union](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions) (also known as a tagged union or an algebraic data type) is the correct tool for the job. In Typescript, we can add a **tag**: a shared key with a unique value to each interface. Using this tag, we can then narrow the range of possible interfaces down to a known one.

Sometimes, however, we're not in control of the interface in question, and cannot add this tag. Other times, it is simply awkward to do so. One common case where this occurs is in React components that provide flexible APIs, where several different props are provided to solve a particular use case, and where these props are mutually exclusive.

As a simple example, imagine a `<Button/>` component that has the following props: `{ size?: 'sm' | 'md' | 'lg' }`, and the following usage: `<Button size="sm"/>`. We get the bright idea to encode these props as booleans, such that our props are now `{ sm?: boolean; md?: boolean; lg?: boolean }`, which lets us do `<Button sm/>`, `<Button md/>`, and `<Button lg/>`. This is kind of neat, but what happens when someone does `<Button sm md/>`? This is an invalid combination of props.

While I generally recommend avoiding this pattern and using ADTs to achieve a similar effect, sometimes dealing with tagless unions is unavoidable.

## Real-world example: React Router

One frequently encountered case where this design pattern can be seen is in React Router's [`<Route/>`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/api/Route.md#route-props) component. Three props are provided to handle rendering: `component`, `render`, and `children`, and only one of the three should be provided.

The following usage note can be found in the documentation:

> You should use only one of these props on a given `<Route>`. See their explanations below to understand the differences between them.

We can see what the interface looks like from the unofficial DefinitelyTyped declarations (with irrelevant types excluded):

```typescript
export interface RouteProps {
  component?:
    | React.ComponentType<RouteComponentProps<any>>
    | React.ComponentType<any>;
  render?: (props: RouteComponentProps<any>) => React.ReactNode;
  children?:
    | ((props: RouteChildrenProps<any>) => React.ReactNode)
    | React.ReactNode;
}
```

There's nothing stopping us at the type level from passing all three props, or passing none of the props, both of which are invalid. So, how can we improve our types to catch these cases?

## At least one, but not multiple

Our problem statement is that we want to be able to require at least one of several possible values, but not more than one. This interface can be modeled as such:

```typescript
// Things like props.path that all route "flavors" use, with the following
// three props omitted.
type RoutePropsCommon = Omit<RouteProps, "render" | "component" | "children">;

// Notice the optionality has been removed
interface RoutePropsComponent {
  component:
    | React.ComponentType<RouteComponentProps<any>>
    | React.ComponentType<any>;
}

interface RoutePropsRender {
  render: (props: RouteComponentProps<any>) => React.ReactNode;
}

interface RoutePropsChildren {
  children:
    | ((props: RouteChildrenProps<any>) => React.ReactNode)
    | React.ReactNode;
}

// interface BetterRouteProps = RoutePropsCommon & (some combination of above 3)
```

We want our output type to ensure that at least one of `component`, `render` and `children` is provided, but not multiple. So how do we get our `BetterRouteProps`?

## Exclusive OR

For now, let's simplify our problem by pretending `children` doesn't exist, so we only have to deal with `component` and `render`. Now, we need some sort of binary type combinator that requires exactly one of the two arguments. This is just the exclusive OR (XOR).

Thus, we need to create a generic type `XOR<T, U>` that returns a type that enforces that either `T` or `U` is implemented, but not neither, and not both.

In attempting to solve this problem, I stumbled across a [very interesting discussion](https://github.com/Microsoft/TypeScript/issues/14094#issuecomment-373782604) on the Typescript Github repo, with several implementations of `XOR`. I've adapted the one that I thought was clearest to the example below:

```typescript
/**
 * @typedef Without
 *
 * Takes two record types `T` and `U`, and outputs a new type where the keys
 * are `keyof T - keyof U` and the values are `undefined | never`.
 *
 * Meant to be used as one operand of a product type to produce an XOR type.
 */

type Without<T, U> = { [P in Exclude<keyof T, keyof U>]?: never };
/**
 * @typedef XOR
 *
 * Takes two record types `T` and `U`, and produces a new type that allows only
 * the keys of T without U or the keys of U without T.
 */
type XOR<T, U> = (T | U) extends object
  ? (Without<T, U> & U) | (Without<U, T> & T)
  : T | U;
```

Using this type constructor, we can achieve our goal:

```typescript
type BetterRouteProps = RoutePropsCommon &
  XOR<RoutePropsComponent, RoutePropsRender>;

// TypeError: missing properties
const invalidRouteMissingProps: BetterRouteProps = {};

// TypeError: too many properties
const invalidRouteTooManyProps: BetterRouteProps = {
  render: () => <div>Hello</div>,
  component: () => <div>Hello</div>
};

// Valid!
const validRoute: BetterRouteProps = {
  render: () => <div>Hello</div>,
  component: undefined // this is allowed
};
```

## Going beyond binary

We're mostly there, but remember that we have three props we want to make exclusive, and our `XOR` is a binary type constructor. However, recall that `XOR` is both commutative and associative. Thus, we can compose our `XOR`s to allow for any number of interfaces to verify.

Unfortunately, Typescript does not provide syntax for the composition of types, so we have to do this manually:

```typescript
type EvenBetterRouteProps = RoutePropsCommon &
  XOR<RoutePropsComponent, XOR<RoutePropsRender, RoutePropsChildren>>;
```

We can extend this by chaining as many `XOR`s as we need.

{% hint style="info" %}
This declaration is quite verbose. If anyone can give advice on writing a type `type XOR<T, K extends keyof T>` that can be used like `type EvenBetterRouteProps = XOR<RouteProps, 'render' | 'component' | 'children'>`, I'd love to see it! Currently it doesn't seem possible to create a non-record mapped type, but I've been surprised before.
{% endhint %}

{% hint style="info" %}
Reddit user [/u/TwiNighty](https://old.reddit.com/r/typescript/comments/da4rp3/checking_for_interface_exclusivity_using_xor_when/f266w4e/) suggested an alternative, much simpler approach for this use case that uses the same principles (though doesn't use `XOR`). Thanks!

```typescript
type OneOf<T, K extends keyof T> = Omit<T, K> &
  {
    [k in K]: Pick<Required<T>, k> &
      {
        [k1 in Exclude<K, k>]?: never;
      };
  }[K];

type EvenBetterRouteProps = OneOf<
  RouteProps,
  "component" | "render" | "children"
>;
```

{% endhint %}

## Downsides

One major downside of this approach comes from the way Typescript "unrolls" this type. For example, here's a sample type error returned by Typescript:

```
 Type '{}' is not assignable to type 'EvenBetterRoute'.
  Type '{}' is not assignable to type 'Pick<RouteProps, "location" | "path" | "exact" | "sensitive" | "strict"> & Without<(Without<Required<Pick<RouteProps, "component">>, Required<Pick<RouteProps, "children">>> & Required<...>) | (Without<...> & Required<...>), Required<...>> & Required<...>'.
    Property 'render' is missing in type '{}' but required in type 'Required<Pick<RouteProps, "render">>'.
```

Pretty opaque if you're not already familiar with the internal workings of this type.

Even so, this is a nice trick to be aware of, when using an ADT is not an option.

## Resources

- [`XOR` discussion on Github](https://timhwang21.gitbook.io/index/programming/typescript/xor-type)
- [Properties of `XOR`](https://en.wikipedia.org/wiki/Exclusive_or#Properties)
