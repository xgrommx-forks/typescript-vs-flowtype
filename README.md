# TypeScript vs Flow

Both TypeScript and Flow are very similar products and they share most of their syntax with some important differences.
In this document I've tried to compile the list of differences and similarities between Flowtype and TypeScript 2.1 -- specifically the syntax, usage and usability.

## Disclaimer

This might be incomplete and/or contain mistakes.
I'm open to contributions and comments.

# Differences in usage and usability

|   | TypeScript            | Flow |
|---|------------------|--------|
| IDE integrations | top-notch | sketchy, must save file to run type-check; some IDEs do hacky workarounds to run real-time |
| speed | real-time                  | good       |
| autocomplete | both during declaration and usage | [only for usage](https://github.com/facebook/flow/issues/3074) |
| expressiveness | great (since TS @ 2.1) | great |
| type safety | very good (7 / 10) (since TS @ 2.0) | great (8 / 10) |
| typings for public libraries | plenty of well maintained typings | a handful of mostly incomplete typings |
| unique features | <ul><li>autocomplete for object construction</li><li>declarable `this` in functions (typing `someFunction.bind()`)</li><li>large library of typings</li><li>more flexible [type mapping via iteration](https://github.com/Microsoft/TypeScript/pull/12114)</li><li>namespacing</li><li>[type spread operator](https://github.com/Microsoft/TypeScript/pull/11150) (not yet released)</li></ul> | <ul><li>variance</li><li>existential types `*`</li><li>testing potential code-paths when types not declared for maximum inference</li><li>`$Diff<A, B>` type</li></ul> |
| ecosystem flexibility | [work in progress](https://github.com/Microsoft/TypeScript/issues/6508) | no extensions |
| programmatic hooking | architecture prepared, work in progress | work in progress |
| documentation and resources | <ul><li>very good docs</li><li>many books</li><li>videos</li><li>e-learning resources</li></ul> | incomplete and often vague docs |
| commercial support | no | no |
| readability of errors | good | good in most, vague in some cases |

# Differences in syntax

## bounded polymorphism

### Flow

```js
function fooGood<T: { x: number }>(obj: T): T {
  console.log(Math.abs(obj.x));
  return obj;
}
```

### TypeScript

```ts
function fooGood<T extends { x: number }>(obj: T): T {
  console.log(Math.abs(obj.x));
  return obj;
}
```

### Reference

https://flowtype.org/blog/2015/03/12/Bounded-Polymorphism.html

## maybe & nullable type

### Flow

```js
let a: ?string

// equvalent to:

let a: string | null | void
```

### TypeScript

```ts
let a: string | null | undefined
```

Optional parameters implicitly add `undefined`:

```ts
function f(x?: number) { }
// same as:
function f(x?: number | undefined) { }
```

## type casting

### Flow

```js
(1 + 1 : number);
```

### TypeScript

```ts
(1 + 1) as number;

// OR (old version, not recommended):

<number> (1 + 1);
```

## mapping dynamic module names

### Flow

`.flowconfig`

```ini
[options]
module.name_mapper='^\(.*\)\.css$' -> '<PROJECT_ROOT>/CSSModule.js.flow'
```

`CSSModule.js.flow`

```js
// @flow

// CSS modules have a `className` export which is a string
declare export var className: string;
```

### TypeScript

```ts
declare module "*.css" {
  export const className: string;
}
```

### Reference

- https://www.typescriptlang.org/docs/handbook/modules.html
- https://flowtype.org/docs/modules.html

## Exact/Partial Object Types

By default objects in Flow are not exact (can contain more properties than declared), whereas in TypeScript they are always exact (must contain only declared properties).

### Flow

When using flow, `{ name: string }` only means “an object with **at least** a name property”.

```js
type ExactUser = {| name: string, age: number |};
type User = { name: string, age: number };
type OptionalUser = $Shape<User>; // all properties become optional
```

### TypeScript

TypeScript is more strict here, in that if you want to use a property which is not declared, you must explicitly say so by defining the indexed property. You will be allowed to use custom properties, but will have to access them through the bracket access syntax, i.e. UserInstance['someProperty']. At the moment, you cannot define "open" (non-exact) types using TypeScript. This is mostly a design decision as it forces you to write the typings upfront.

```js
type ExactUser = { name: string, age: number };
type User = { name: string, age: number, [otherProperty: string]: any };
type OptionalUser = Partial<{ name: string, age: number }>; // all properties become optional
```

### Reference

- https://flowtype.org/docs/objects.html
- https://github.com/Microsoft/TypeScript/issues/2710

## Importing types

### Flow

```js
import type {UserID, User} from "./User.js";
```

### TypeScript

TypeScript does not treat Types in any special way when importing.

```ts
import {UserID, User} from "./User.js";
```

## typeof

Works the same in both cases, however Flow has an additional syntax to directly import a `typeof`:

### Flow

```js
import typeof {jimiguitar as GuitarT} from "./User";

// OR (below also works in TypeScript)

import {jimiguitar} from "./User.js";
type GuitarT = typeof jimiguitar;
```

### TypeScript

```ts
import {jimiguitar} from "./User";
type GuitarT = typeof jimiguitar;
```

## Accessing the type of a Class

### Flow

```js
class Test {};
type TestType = Class<Test>;
// This should be equivalent to (if you can confirm, please send a PR):
type TestType = typeof Test;
```

### TypeScript

```ts
class Test {};
type TestType = typeof Test;
```

## Keys/Props Of Type

### Flow

```js
var props = {
  foo: 1,
  bar: 'two',
  baz: 'three',
}

type PropsType = typeof props;
type KeysOfProps = $Enum<PropsType>;

function getProp<T>(key: KeysOfProps): T {
  return props[key]
}
```

### TypeScript

```ts
var props = {
  foo: 1,
  bar: 'two',
  baz: 'three',
}

type PropsType = typeof props
type KeysOfProps = keyof PropsType;

function getProp<T>(key: KeysOfProps): T {
  return props[key]
}
```

## Records

### Flow

```js
type $Record<T, U> = {[key: $Enum<T>]: U}
type SomeRecord = $Record<{ a: number }, string>
```

### TypeScript

```ts
type SomeRecord = Record<{ a: number }, string>
```

## Lookup Types

### Flow

```js
type A = {
  thing: string
}

type lookedUpThing = $PropertyType<A, 'thing'>
```

### TypeScript

```ts
type A = {
  thing: string
}

type lookedUpThing = A['thing']
```

## Mapped Types / Foreach Property

### Flow

```js
type InputType = { hello: string };
type MappedType = $ObjMap<InputType, ()=>number>;
```

Reference:

- https://gist.github.com/gabro/bb83ed574690645053b815da2082b937
- https://twitter.com/andreypopp/status/782192355206135808

### TypeScript

A bit more flexibility here, as you have access to each individual key name and can combine with Lookup types and even do simple transformations.

```ts
type InputType = { hello: string };
type MappedType = {
  [P in keyof InputType]: number;
};
```

## Read-only Types

### Flow

```js
type A = {
  +b: string
}

let a: A = { b: 'something' }
a.b = 'something-else'; // ERROR
```

### TypeScript

```ts
type A = {
  readonly b: string
}

let a: A = { b: 'something' }
a.b = 'something-else'; // ERROR
```

One caveat that makes TypeScript's `readonly` less safe is that the same `non-readonly` property in a type is compatible with a `readonly` property. This essentially means that you can pass an object with `readonly` properties to a function which expects non-readonly properties and TypeScript will *not* throw errors: [example](https://www.typescriptlang.org/play/index.html#src=%0D%0Afunction%20test(x%3A%20%7B%20foo%3A%20string%20%7D)%20%7B%20%0D%0A%20%20%20%20x.foo%20%3D%20'bar'%3B%0D%0A%7D%0D%0A%0D%0Aconst%20x%3A%20%7B%20readonly%20foo%3A%20string%20%7D%20%3D%20%7B%20foo%3A%20'baz'%20%7D%3B%0D%0A%0D%0Atest(x)%3B).

## "Impossible flow" type

### Flow

`empty`

```js
function returnsImpossible() {
  throw new Error();
}

// type of returnsImpossible() is 'empty'
```

### TypeScript

`never`

```ts
function returnsImpossible() {
  throw new Error();
}

// type of returnsImpossible() is 'never'
```

# Same syntax

Most of the syntax of Flow and TypeScript is the same. TypeScript is more expressive for certain use-cases (advanced mapped types with keysof, readonly properties), and Flow is more expressive for others (e.g. `$Diff`).

## optional parameters

### Flow and TypeScript

```js
function(a?: string) {}
```

# TypeScript-only concepts

## Declarable arbitrary `this` in functions (outside of objects)

```ts
function something(this: { hello: string }, firstArg: string) {
  return this.hello + firstArg;
}
```

## Private and Public properties in classes

```ts
class SomeClass {
  constructor(public prop: string, private prop2: string) {
    // transpiles to:
    // this.prop = prop;
    // this.prop2 = prop2;
  }
  private prop3: string;
}
```

## [Non-null assertion operator](https://github.com/Microsoft/TypeScript/pull/7140)

Add `!` to signify we know an object is non-null.

```ts
// Compiled with --strictNullChecks
function validateEntity(e: Entity?) {
  // Throw exception if e is null or invalid entity
}

function processEntity(e: Entity?) {
  validateEntity(e);
  let s = e!.name;  // Assert that e is non-null and access name
}
```

# Flow-only concepts

## Difference types

```js
type C = $Diff<{ a: string, b: number }, { a: string }>
// C is { b: number}
```

TypeScript has a [proposal](https://github.com/Microsoft/TypeScript/issues/12215) for an equivalent.

## Inferred existential types

`*` as a type or a generic parameter signifies to the type-checker to infer the type if possible

```js
Array<*>
```

TypeScript has a proposal for an equivalent (needs link).

## Variance

https://flowtype.org/docs/variance.html

```js
function getLength(o: {+p: ?string}): number {
  return o.p ? o.p.length : 0;
}
```

[TypeScript proposal](https://github.com/Microsoft/TypeScript/issues/10717)

## Flow's "mixed" type

The TypeScript equivalent of the `mixed` type is simply:

```ts
type mixed = {}
```

Reference: https://flowtype.org/docs/quick-reference.html#mixed

## Useful References

* https://github.com/Microsoft/TypeScript/issues/1265
* Undocumented Flow modifiers https://github.com/facebook/flow/issues/2464
* http://sitr.us/2015/05/31/advanced-features-in-flow.html
