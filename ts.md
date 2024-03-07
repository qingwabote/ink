# Readonly is a joke

```ts
interface Foo {
    name: string;
}
type ReadOnlyFoo = Readonly<Foo>;

let readOnlyFoo: ReadOnlyFoo = { name: "foo" };

let foo: Foo = readOnlyFoo; // should show a error
foo.name = ""; // value is changed with no error either
```

*[add enforceReadonlyAssignability flag for smoother transition to true readonly modifiers](https://github.com/Microsoft/TypeScript/issues/13002)*