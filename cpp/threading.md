# Data race & Race condition
Data race 源于多个线程对同一变量的非原子的读写（至少有一个写），也许用“同一内存地址的非原子读写”更准确，因为分别访问同一变量 vector a 的不同位置 a[0] 和 a[1] 不存在 data race  
Race condition 典型的例子是两个线程对同一个变量做加 1  
*[A data race is a type of race condition & Not all regard data races as a subset of race conditions](https://en.wikipedia.org/wiki/Race_condition)*

# Atomic operation & Memory fence
试想，线程 1 管理数据 data, 通过 flag(flag = 1)通知线程 2 可以读取 data 了
```cpp
data = 0;
flag = 0;

// thread 1
data = 3;
flag = 1;

// thread 2
while(flag != 1) {};
print(data);
```
目前没有约定两个变量的读写顺序，可能线程 2 读出 flag = 1 时 data = 3 还未写入

现在插入两个**内存屏障**
```cpp
// thread 1
data = 3;
--- store ---;
flag = 1;

// thread 2
while(flag != 1) {};
--- load ---;
print(data);
```
屏障 store 阻塞了 cpu 的写操作，直到清空 cache(完成前面的写入)  
屏障 load 阻塞了 cpu 的读操作，直到清空 invalidate queue(使之前的写入对 cpu 可见)

如果换成两个**单向屏障**
```cpp
// thread 1
data = 3;
--- store-release ---;
flag = 1;

// thread 2
while(flag != 1) {};
--- load-acquire ---;
print(data);
```
store-release 只挡住后面的指令  
load-acquire 只挡住前面的指令

循序可能变成这样
```cpp
store flag
store data
--- store-release ---;
--- load-acquire ---;
load  data
load  flag
```
或是
```cpp
--- load-acquire ---;
load  data
load  flag
store flag
store data
--- store-release ---;
```
如此，CPU 仅需保证对这两个变量的读和写不会交错即可，我猜测实现方式是允许清空 cache 延后，允许清空 invalidate queue 提前

# C++
The standalone acquire fence is not "acquire operation"
```cpp
#include <atomic>
int standalone_acquire_load(std::atomic<int> &var){
    int ret = var.load(std::memory_order_relaxed);
    std::atomic_thread_fence(std::memory_order_acquire);
    return ret;
}

int normal_acquire_load(std::atomic<int> &var){
    int ret = var.load(std::memory_order_acquire);
    return ret;
}
```
在 ARM 下编译为
```asm 
standalone_acquire_load(std::atomic<int>&):
        ldr     r0, [r0]  // plain load
        dmb     ish       // FULL BARRIER //和使用 memory_order_seq_cst 没区别
        bx      lr  

normal_acquire_load(std::atomic<int>&):
        lda     r0, [r0]  // acquire load
        bx      lr  
```
*https://stackoverflow.com/questions/61710818/c11-standalone-memory-barriers-loadload-storestore-loadstore-storeload*

[Acquire and Release Fences Don't Work the Way You'd Expect](https://preshing.com/20131125/acquire-and-release-fences-dont-work-the-way-youd-expect/)