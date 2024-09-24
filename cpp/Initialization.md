```cpp
struct Foo {
    int a;
};

Foo f;
```
If no **initializer** is specified for an object, the object is **default-initialized**.  

**Member initialization,** Non-static data members may be initialized in one of two ways:   
- In the **member initializer list** of the constructor.  
- Through a **default member initializer**, which is a brace or equals initializer included in the member declaration and is used if the member is omitted from the member initializer list of a constructor. If a member has a default member initializer and also appears in the member initialization list in a constructor, the default member initializer is ignored for that constructor.

*[Before the compound statement that forms the function body of the constructor begins executing, initialization of all direct bases, virtual bases, and non-static data members is finished.](https://en.cppreference.com/w/cpp/language/constructor)* 
 
**Default-initialization** calls **default constructor(Implicitly-declared)**