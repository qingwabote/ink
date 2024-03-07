[Emplace a lambda which captured a unique_ptr into container like queue](https://codereview.stackexchange.com/questions/257663/emplace-a-lambda-which-captured-a-unique-ptr-into-container-like-queue)

unique_ptr capture with std::move is ok, but then, putting the lambda function into container meets a compile error

[Because std::function is copyable, the standard requires that callables used to construct it also be copyable](https://stackoverflow.com/questions/25330716/move-only-version-of-stdfunction)

[Lambda with dynamic storage duration](https://stackoverflow.com/questions/37924996/lambda-with-dynamic-storage-duration)

[UniqueFunction](https://github.com/qingwabote/zero/blob/master/native/core/UniqueFunction.hpp)