# [SWIG](https://www.swig.org/)

## Module
每个 Module 包含完整的 runtime, 独立运行，甚至"SWIG modules generated with different versions can peacefully coexist".

## 在 JS 绑定中需要解决的问题

### 总是返回新的 JS 对象
swig 并不维护其生成的 JS 对象。可以创建 map 将 JS 对象与原生对象（指针）关联，为了不破坏 GC，map 里可以存 JS 对象的弱引用。

但还是有一个逻辑漏洞：  
在 JS 端创建一个原生对象 A 和与之绑定的 JS 对象，然后赋值给另一个原生对象 B 作为属性，当你从 B 的此属性取回 A 时，即便我们使用 map 做了关联，一旦最初的那个 JS 对象被 GC，我们仍然得创建一个新 JS 对象用于返回。所以，要注意，你取的和存的可能不是同一个 JS 对象。

### 不支持 callback
可以使用 [Typemaps](https://www.swig.org/Doc4.1/Typemaps.html#Typemaps) 自己实现

### 对智能指针的支持并不完整
官方提供了 std_unique_ptr.i, 但没有 [std_shared_ptr.i](https://github.com/swig/swig/pull/1365), 我在 Zero 引擎中给出了一种[实现方案](https://github.com/qingwabote/zero/blob/master/native/main/swig/Lib/javascript/v8/std_shared_ptr.i)

### 目前不支持嵌套类

### 函数可选参数不支持以 undefined 作为实参
[How to define a parameter as optional, but if specified be ensured as not undefined](https://github.com/microsoft/TypeScript/issues/11988)