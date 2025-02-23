Implicit initialization
- objects with automatic storage duration are initialized to indeterminate values (which may be trap representations). *听起来像是没做初始化的委婉说法？*
- objects with static and thread-local storage duration are empty-initialized. *处于数据安全原因，操作系统将分配给 .bss 段的内存（可能来自于其它应用）初始化为零，所以未定义的全局变量的值是零，相反，对于 automatic/dynamic storage duration 栈或堆上的变量就没有隐式初始化，因为这是应用内的，自己处理（显式初始化）即可*


```cpp
struct Foo {
    int a;
};

Foo f;
```
If no **initializer** is specified for an object, the object is **default-initialized**.  

**Member initialization:** Non-static data members may be initialized in one of two ways:   
- In the **member initializer list** of the constructor.  
- Through a **default member initializer**, which is a brace or equals initializer included in the member declaration and is used if the member is omitted from the member initializer list of a constructor. If a member has a default member initializer and also appears in the member initialization list in a constructor, the default member initializer is ignored for that constructor.

*[Before the compound statement that forms the function body of the constructor begins executing, initialization of all direct bases, virtual bases, and non-static data members is finished.](https://en.cppreference.com/w/cpp/language/constructor)* 
 
**Default-initialization** calls **default constructor(Implicitly-declared if not defined)**