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

# Namespace vs Module
ES Module 解决了符号冲突的问题，但如果想省略前后缀，就得用 Namespace 或等价写法
```ts
export createVec4(...)
export copyVec4(...)
export addVec4(...)

//

export namespace vec4 {
    export create(...)
    export copy(...)
    export add(...)
}
// or
export const vec4 = {
    create(...)
    copy(...)
    add(...)
}
```
*Namespace prohibits **tree-shaking***