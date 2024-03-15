# [RAII](https://zh.cppreference.com/w/cpp/language/raii)
## [构造函数中对虚函数的调用(cpp)](https://google.github.io/styleguide/cppguide.html#Doing_Work_in_Constructors)
## 构造函数中对模板方法的调用(JS)
在 ES6 构造函数中调用被子类重写的方法时，该方法访问的子类属性尚未初始化，**在派生的类中，在你可以使用 'this' 之前，必须先调用 super()**

如果我们写 init() 并把它包装在静态的 create 方法中，又会陷入 [js 静态方法应不应该被继承的争议之中](https://github.com/microsoft/TypeScript/issues/4628). 我目前的方案像这样：
``` ts
export class Pass {
    static Pass(state: PassState, type = 'default') {
        const pass = new Pass(state, type);
        pass.initialize();
        return pass;
    }
    ...
}

export class PassInstance extends Pass {
    static PassInstance(raw: Pass) {
        const instance = new PassInstance(raw);
        instance.initialize();
        return instance;
    }
    ...
}

const pass = Pass.Pass(...);
const instance = PassInstance.PassInstance(...);
```

# 二段构造
“其实我们设计二段构造时首先考虑其优势而非兼容cocos2d-iphone. 初始化时会遇到图片资源不存在等异常，而C++构造函数无返回值，只能用try-catch来处理异常，启用try-catch会使编译后二进制文件大不少，故需要init返回bool值。Symbian, Bada SDK，objc的alloc + init也都是二阶段构造” —— 王哲。