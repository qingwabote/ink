# Data race & Race condition
Data race 源于多个线程对同一变量的非原子的读写（至少有一个写），也许用“同一内存地址的非原子读写”更准确，因为分别访问同一变量 vector a 的不同位置 a[0] 和 a[1] 不存在 data race  
Race condition 典型的例子是两个线程对同一个变量做加 1  
*[A data race is a type of race condition & Not all regard data races as a subset of race conditions](https://en.wikipedia.org/wiki/Race_condition)*

# Cache controller
```plantuml
skinparam handwritten true
object coreA
object coreB

object cacheA
object cacheB

object memory

coreA --> cacheA: write to cache
cacheA --> memory: flush cache,\n make it available

coreB <-- cacheB: read from cache
cacheB <-- memory: invalidate cache,\n make it visible
```

# Instruction reordering

# Atomic operation & Memory fence
```cpp
// thread 1
data = 3;
flag = 1;

// thread 2
while(flag != 1) {};
print(data);
```
目前两个变量的读写顺序未知，比如
```cpp
load  data
store data
store flag
load  flag
```
插入两个**内存屏障**
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
循序可能变成这样
```cpp
store data
--- store ---;
store flag
load  flag
--- load ---;
load  data
```
或者...总之，屏障 load 保证了对 flag 的读在 data 之前，对 data 的写在 flag 之前

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
意味着：  
允许对 flag 的 store，提前  
允许对 flag 的 load，延后  
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
如此，CPU 仅需保证对这两个变量的读和写不会交错

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