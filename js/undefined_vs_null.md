在 typescript 中 undefined 存在一种用法是无法被 null 替代的：
```ts
class Bar {
    public foo?:number = undefined;
}

const bar = new Bar;
bar.foo = undefined; // 类型 “undefined” 不能分配给“exactOptionalPropertyTypes: true”的类型 “number”。请考虑将 “undefined” 添加到目标类型。。ts(2412)
```