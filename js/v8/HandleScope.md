# EscapableHandleScope
"handle_scope.Escape(handle)" 通过把指定 handle 放入父作用域，使其从本作用域的销毁中逃脱。做个实验验证这一点：
```cpp
    v8::HandleScope handle_scope(isolate);
    {
        v8::EscapableHandleScope handle_scope(isolate);

        auto foo = v8::String::NewFromUtf8Literal(isolate, "foo");
        // handle_scope.Escape(foo); // Use this, the WeakCallback will not be triggered

        v8::Persistent<v8::String> *ref =
            new v8::Persistent<v8::String>(isolate, foo);
        ref->SetWeak<v8::Persistent<v8::String>>(
            ref,
            [](const v8::WeakCallbackInfo<v8::Persistent<v8::String>> &data)
            {
                data.GetParameter()->Reset();
                delete data.GetParameter();
            },
            v8::WeakCallbackType::kParameter);
    }
    // v8::V8::SetFlagsFromString("--expose-gc-as=__gc__");
    auto gc = sugar::v8_object_get<v8::Function>(context, context->Global(), "__gc__");
    gc->Call(context, context->Global(), 0, nullptr);
```
*[WeakCallbackApi](https://raw.githubusercontent.com/nodejs/node/main/deps/v8/test/cctest/test-api.cc)*

# 只能“返回”一个值