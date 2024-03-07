Mono 使用 [平台调用 (P/Invoke)](https://docs.microsoft.com/zh-cn/dotnet/standard/native-interop/pinvoke) 技术与 puerts 交互  

# CS 向 JSEngine 注册类型
```plantuml
skinparam handwritten true

 -> TypeRegister.cs: RegisterType
TypeRegister.cs -> JSEngine.cpp:RegisterClass
note right: 注册构造函数和析构函数
JSEngine.cpp -> TypeRegister.cs:typeId
TypeRegister.cs -> JSEngine.cpp:RegisterFunction
note right: 注册函数
TypeRegister.cs -> JSEngine.cpp:RegisterProperty
note right: 注册属性的 getter 和 setter 函数
```

# JS 获取已注册的 CS 类型
```plantuml
skinparam handwritten true

JsEnv.cs -> JSEngine.cpp: SetGlobalFunction("__tgjsLoadType", JsEnv.LoadType)
csharp.mjs -> JsEnv.cs: LoadType
JsEnv.cs -> TypeRegister:GetTypeId
JsEnv.cs -> Puerts.cpp:ReturnClass
Puerts.cpp -> JSEngine.cpp:GetClassConstructor
Puerts.cpp -> v8.FunctionCallbackInfo:GetReturnValue().Set
```

# BlittableCopy
**平台调用**可以帮我们完成一些托管值类型到基本的非托管类型的转换(Marshaling)
```cs
        public static extern void ReturnNumber(IntPtr isolate, IntPtr info, double number);

#if PUERTS_GENERAL && !PUERTS_GENERAL_OSX
        [DllImport(DLLNAME, CallingConvention = CallingConvention.Cdecl, EntryPoint = "ReturnString")]
        public static extern void __ReturnString(IntPtr isolate, IntPtr info, byte[] str);
#else
        [DllImport(DLLNAME, CallingConvention = CallingConvention.Cdecl, EntryPoint = "ReturnString")]
        public static extern void __ReturnString(IntPtr isolate, IntPtr info, string str);
#endif
        [DllImport(DLLNAME, CallingConvention = CallingConvention.Cdecl)]
        public static extern void ReturnBigInt(IntPtr isolate, IntPtr info, long number);

        [DllImport(DLLNAME, CallingConvention = CallingConvention.Cdecl)]
        public static extern void ReturnBoolean(IntPtr isolate, IntPtr info, bool b);

        [DllImport(DLLNAME, CallingConvention = CallingConvention.Cdecl)]
        public static extern void ReturnDate(IntPtr isolate, IntPtr info, double date);
```
对于结构体，puerts 并没有再定义一套对应的非托管类型。与 Class 一样，传递到非托管代码继而传递到 JS 的是结构体的指针。  
但结构体是值类型，在栈上分配，与 JS 在堆上分配的 object 具有不同的生命周期。  
puerts 的一种方案是将结构体拷贝到堆上，那么，为了存储不同的结构体类型，需要将结构体**装箱**到 object。  
为了避免装箱，puerts 给出了另一种方案，直接将结构体拷贝到 JS ，由 JS 管理其生命周期，即 **BlittableCopy**。  