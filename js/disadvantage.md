## 没有可封装数据和相关功能的值类型

**可封装数据和相关功能的类型**只有在堆上分配内存的引用类型 Object 这一种, 为了减少内存分配次数和 GC 压力我们需要尽可能的复用 Object, 这带来了新问题，在 Object 的传递过程中要储存它则必须建立拷贝，因为它是一个被复用的临时对象。在 C# 中可以使用[结构体](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/struct)解决此问题