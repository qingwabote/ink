[Vulkan 简介](https://zhuanlan.zhihu.com/p/165141740)

[Vulkan 学习笔记](https://github.com/GavinKG/ILearnVulkanFromScratch-CN)

# Vulkan Loader
在 windows 上为 vulkan-1.dll，Vulkan Runtime 说的应该也是它。  
*[Runtime - Runtime Installer. Installs the Vulkan Loader on your system. Most users do NOT need to install the runtime installer. The preferred method for obtaining the runtime installer is from your IHVs driver package update.](https://vulkan.lunarg.com/sdk/home?fbclid=IwAR3uPe0tJMTAdnaDAELcT-wI44vKlvv1hEfzokwyLIQuOAgOI6D7qh_HjnA)* 

# Threaded Command Buffer Generation
```plantuml
skinparam handwritten true

state "Thread/CPU 1 (Busy)" as Thread1 {
    state "Update Work" as UW1
    state "Write Command Buffers" as WCB1
}

state "Thread/CPU 2 (Busy)" as Thread2 {
    state "Update Work" as UW2
    state "Write Command Buffers" as WCB2
}

state "Thread/CPU 3 (Busy)" as Thread3 {
    state "Update Work" as UW3
    state "Write Command Buffers" as WCB3
}

state "Thread/CPU 4 (Busy)" as Thread4 {
    state "Game Work" as GW
    state "Thread Coordination" as TC
    state "Submit to Queue" as STQ
    state "Swapping" as S
}

state "GPU (Busy - Good…)" as GPU

Thread1 --> Thread4
Thread2 --> Thread4
Thread3 --> Thread4
Thread4 -right-> GPU
```
*<https://developer.nvidia.com/sites/default/files/akamai/gameworks/blog/munich/mschott_vulkan_multi_threading.pdf>*

# Synchronization
## Pipeline Barriers
*[Global memory barriers vs buffer/image memory barrier](https://www.reddit.com/r/vulkan/comments/v2mswb/global_memory_barriers_vs_bufferimage_memory/?utm_source=share&utm_medium=web2x&context=3)*
## Subpass Dependencies
实际上是 Subpass Stage Dependency, 即上个 Subpass 的某个 Stage 与下个 Subpass 的某个 Stage 的依赖关系  
*[Subpass Dependencies: What are those and why do I need them?](https://www.reddit.com/r/vulkan/comments/s80reu/subpass_dependencies_what_are_those_and_why_do_i/)*  

[The external subpass dependency is basically just a vkCmdPipelineBarrier injected for you by the driver](http://themaister.net/blog/2019/08/14/yet-another-blog-explaining-vulkan-synchronization/)

*[游戏引擎随笔 0x07：现代图形 API 的同步 - 丛越的文章 - 知乎](https://zhuanlan.zhihu.com/p/100162469)
[关于 Vulkan Tutorial 中同步问题的解释](https://zhuanlan.zhihu.com/p/350483554)   
[draw calls will respect the results of previous draw calls, within the same subpass (between subpasses is governed by explicit subpass dependencies)](https://stackoverflow.com/questions/56849788/synchronization-between-drawcalls-in-vulkan)*

## Cache control
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
*[To be clear, we can say that “making memory available” is all about flushing caches, and “making memory visible” is invalidating caches. This should make it more obvious what is going on](http://themaister.net/blog/2019/08/14/yet-another-blog-explaining-vulkan-synchronization/)*

## Memory layout

## Image layouts
[Image layouts are likely (though they don't have to be) used for internal transparent compression of images when in use by the GPU. This is NOT a lossy block compressed format, it is an internal format that is used by the GPU to save bandwidth! It is unlikely there will be a "standard" compression format that can be exposed to the CPU. The reason you need to transition your images from one layout to another is some hardware may only be able to access the compressed data from certain hardware blocks.](https://www.reddit.com/r/vulkan/comments/48cvzq/comment/d0j92ac/?utm_source=share&utm_medium=web2x&context=3)

## Access scopes 
https://registry.khronos.org/vulkan/specs/1.3-extensions/html/vkspec.html#synchronization-dependencies-access-scopes

# Cross platform
[Handling differences between Vulkan and OpenGL coordinate system](https://www.khronos.org/news/permalink/handling-differences-between-vulkan-and-opengl-coordinate-system)

[weird problem will cullface](https://forums.developer.nvidia.com/t/weird-problem-will-cullface/41525/4)

[It is the GLSL-to-SPIR-V compiler that creates these offsets&strides by applying the OpenGL std140 rules](https://computergraphics.stackexchange.com/questions/8699/clarifying-vulkan-glsl-std140)