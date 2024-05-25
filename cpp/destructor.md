## 默认析构函数并不等于自定义一个空的析构函数
bar.hpp
```hpp
class Foo;
class Bar {
	std::unique_ptr<Foo> foo;
};
```
bar.cpp
```cpp
class Foo {}
```
以上，虽然在 bar.cpp 中存在 Foo 的定义，但在编译器生成 Bar 的默认析构函数时 Foo 的定义不存在，报错 C2027

此时 Bar 的析构函数不能省略，且必须定义在 bar.cpp 中，直接定义在 bar.hpp 中同样不行
