# Exhaustive conditionals with ADTs

If you've used languages with pattern matching like Haskell, Elixir, or Scala, you've surely encountered the "pattern match not exhaustive" error, which signifies that you failed to account for some possible type in a case statement. It's great to leave the work of ensuring that you handled every possible type in your sum type to the type checker, instead of mentally tracking it in your head. It's also extremely robust against regressions when refactoring.

This pattern is fully supported in Typescript as well, and in my opinion is one of the most powerful features of Typescript that every Typescript developer should be aware of. We'll use the example of a Redux reducer here.

## Exhaustiveness checking with sum types

The following example describes a reducer modeling state for a counter, which has actions to increment, decrement, and inexplicably, sing and dance. In regular Javascript, if `"SING_AND_DANCE"` were passed as an action, it would be silently ignored without error. However, in Typescript, because we specify the return type as `number`, we will get a type error, as `undefined` is returned when `"SING_AND_DANCE"` fails to match.

```typescript
type CounterAction = "INCREMENT" | "DECREMENT" | "SING_AND_DANCE";

// TypeError: return type does not include 'undefined'.
const reducer = (state: number, action: CounterAction): number => {
  switch (action) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    // case "SING_AND_DANCE":
    //   console.log('Singing in the rain...')
    //   return state
  }
};
```

What happens here is that `action` is initially typed as `"INCREMENT" | "DECREMENT" | "SING_AND_DANCE"`. With every `case` that is handled, the handled literal value is ejected from the type; as such, the inferred type of `action` becomes `"DECREMENT" | "SING_AND_DANCE"` after the first case, and then just `"SING_AND_DANCE"` after the second (a process known as [type narrowing](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-guards-and-differentiating-types)). Because the function exits at this point, the type checker knows that there is an unhandled type remaining, and emits an error.

## Exhaustiveness checking with ADTs

While the above example demonstrates catching nonexhaustive matches as a type error, it's not very useful -- we generally want to be able to pass a payload of data along with our action. Thus, instead of being a sum type of strings, our `CounterAction` needs to be a sum type of records. Fortunately, Typescript is [structurally typed](https://en.wikipedia.org/wiki/Structural_type_system), meaning that a record whose type isn't explicitly declared can be recognized as being of a following type based on its structure, or the types of its key-value pairs. This allows us to construct record types that specify a certain internal structure, and to use plain Javascript objects as values of this type without explicit declaration as long as the object _structurally_ matches the type.

Given this, we can refactor our `CounterAction`s to all be records that correspond to a shared interface:

```typescript
interface GenericAction {
  readonly type: string;
  payload?: Record<string, any>;
}

interface CounterUpdateAction extends GenericAction {
  readonly type: "COUNTER_UPDATE";
  payload: {
    changeBy: number;
  };
}

interface CounterSingAndDanceAction extends GenericAction {
  readonly type: "COUNTER_SING_AND_DANCE";
  payload: {
    lyrics: string;
  };
}

type CounterAction = CounterUpdateAction | CounterSingAndDanceAction;
```

Here, note that `CounterUpdateAction` and `CounterSingAndDanceAction` both extend a shared generic interface. When there are combined into `CounterAction`, they end up forming a **discriminated union** or **algebraic data type** (ADT). A discriminated union is composed of a **discriminant**, which is a propery whose value is unique across types, and a **union**, which is a sum type of all the discriminated types. Here, `type: "COUNTER_UPDATE" | "COUNTER_SING_AND_DANCE"` is the discriminant, while `type CounterAction` is the union. (Note that we never explicitly specified that `type` should be the discriminant -- it is automatically inferred. It's possible to have multiple discriminants, [as long as you adhere to the rules of discriminated unions.](https://github.com/Microsoft/TypeScript/issues/10586))

We can update our example to use these types. Let's refactor our `INCREMENT` and `DECREMENT` actions into a single update action that takes a number to add (ignoring the fact that this is now no longer really a "counter").

```typescript
// TypeError: return type does not include 'undefined'.
const reducer = (state: number, action: CounterAction): number => {
  switch (action.type) {
    case "COUNTER_UPDATE":
      return state + action.payload.changeBy;
    // case "COUNTER_SING_AND_DANCE":
    //   console.log(action.payload.lyrics);
    //   return state;
  }
};
```

Whereas before, our switch statement checked the exhaustivity of a sum type of string literals, we are now checking the exhaustivity of a sum type of records types, discriminated on the `type` field. With this, we've added the ability to pass largely arbitrary values, as long as they are tagged with the a discriminant.

## Javascript interop; handling `any`

Our system above is quite robust... as long as the input types are valid. Type systems are generally garbage-in, garbage-out, and if someone calls the reducer with untyped or mistyped code, there will still be runtime errors.

While the best way to handle this would simply be to be very strict about types, we can ameliorate this by adding a special `default` handler to the switch statement. In the process of narrowing, if all permissible types are narrowed out, the resultant type is `never`, a special type signifying [a value that can never occur](https://www.typescriptlang.org/docs/handbook/basic-types.html#never). As such, our `default` handler must emit a value of type `never` for our function to type-check.

Thrown errors are classified as being of type `never`. Thus, we can simply throw an error here, to make us aware something went wrong at runtime. While a simple `throw` is sufficient, I describe a utility here that emits a slightly more informative error message for error tracking purposes.

```typescript
export function matchNotExhaustive(x: never): never {
  throw new Error(`Non-exhaustive match: case ${x} was not handled.`);
}

// TypeError: Argument of type 'CounterSingAndDanceAction' is not assignable to parameter of type 'never'.
const reducer = (state: number, action: CounterAction): number => {
  switch (action.type) {
    case "COUNTER_UPDATE":
      return state + action.payload.changeBy;
    // case "COUNTER_SING_AND_DANCE":
    //   console.log(action.payload.lyrics);
    //   return state;
    default:
      return matchNotExhaustive(action.type);
  }
};
```

This pattern can be useful when we are handling an ADT but not necessarily returning a value (for example, if we are enacting a side effect based on an ADT). The previous pattern relies on the return type to properly typecheck, and if the "reducer" returns `void`, the unhandled case would not be reported as a type error. Here, because we are passing a possibly non-`never` value into `matchNotExhaustive` which expects a `never`, we get a type error independent of the reducer's return type.

As an added benefit, our type error is also much more descriptive now. Whereas previously it only told us that something was unhandled, now we are told exactly which type was not handled. (We're better than Haskell, guys!)

