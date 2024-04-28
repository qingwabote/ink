在 typescript 中 undefined 存在一种无法被 null 替代的用法：
```ts
class Bar {
    public foo?:number;
}

const bar = new Bar;
bar.foo = undefined; // 报错 ts(2412) 类型 “undefined” 不能分配给“exactOptionalPropertyTypes: true”的类型 “number”。请考虑将 “undefined” 添加到目标类型。
```
当然你可以这样：
```ts
class Bar {
    public foo:number | undefined;
}

const bar = new Bar;
bar.foo = undefined; // 不会报错
```
或者用 null
```ts
class Bar {
    public foo:number | null = null; // 必须赋值。
                                     // “空不是无，空是一种存在，你得用空这种存在填满自己。”《三体》第16章 三体问题
}

const bar = new Bar;
bar.foo = null;
```