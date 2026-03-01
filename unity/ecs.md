# 非托管
大量非托管类型的加入，使事情变的复杂

## 内存连续

## Burst
“Burst 是一个 IL 的 **LLVM** 前端”

简化版编译流程（每个阶段的产物） [C#] → [IL] → [Burst LLVM IR] → [Machine Code]

# 多态
**面相对象**下，对象在内存布局中存在虚函数表，调用虚函数存在**间接调用**开销，且不被 burst 支持 
## 模板方法
OOP 模板方法的本质是父类事先定义好函数的输入输出与执行顺序，子类仅实现函数自身。迁移到 ECS 每个函数对则应一个 system，每个 system 的输入输出由 component 实现，将 systems 按顺序调度，区别在于 OOP 可以使这些输入输出中间变量仅存在于栈上，处理多个对象，ECS 需要为每个对象单独储存中间变量，内存消耗更高
## 静态多态
更简洁的办法是不走 ECS，走 burst, 走泛型，即静态多态
## 重写
用**WriteGroup**部分代替面相对象中对父类方法的重写，这里父类方法对应的是使用 SystemAPI.Query 的 system. 使一特定类型与 SystemAPI.Query 中的某个类型建立 WriteGroup，带有这个特定类型的 entity 便可以逃离原有 SystemAPI.Query 的处理，不过就别指望能“调用父类方法”。我猜 WriteGroup 的设计哲学是：Query 时，发现类型属于某个 Group 便要确认 Group 的其它成员是否也在 Query 中，否则跳过。猜的很牵强，解释不了为什么非得 Write, Read 不行吗？

# Baking
然而 ECS 没有独立的编辑器，为了帮助你将 Unity 编辑器中的 **GameObject** 转化成 **Entity**，于是 Unity 引入了一个新系统
## Baker
仅为当前 GameObject 关联的 Entity 添加 Component，否则使用 **BakingSystem**
## BakingSystem
但是 Child 仅存在于运行时（由 ParentSystem 维护）

按理说 Baker 与 BakingSystem 是 Editor Only 的，但未见将它们放入 Editor 文件夹下的案例

如果你的 Unmanaged Component 包含集合并参与 Baking 为确保自身的 **Blittable**，你得学习使用 ECS 提供的集合类型：**Dynamic Buffer**(可存放 Entity) 或 **Blob Array**

# [Scenes overview](https://docs.unity3d.com/Packages/com.unity.entities@1.4/manual/conversion-scene-overview.html)
In the entity component system (ECS), scenes work differently. This is because Unity's core scene system is incompatible with ECS. There are the following types of scene concept to understand:
- **Authoring scenes**: An authoring scene is a scene that you can open and edit like any other scene, but is designed for baking to process. It contains GameObjects and MonoBehaviour components that Unity converts to ECS data at runtime.
- **Entity scenes**: An entity scene contains the ECS data that the baking process produces.
- **Subscenes**: A subscene is a reference to an authoring or entity scene. In the Unity Editor, you create a subscene to add authoring elements to. When the subscene is closed, it triggers the baking process for related entity scenes.

上文官方对 Authoring scenes, Entity scenes, Subscenes 的解释让我 confusing, 我目前的理解：  
We create **Subscenes** to reference **Authoring scenes** which contain GameObjects and MonoBehaviour components, and then bake them into **Entity scenes**.

# SystemAPI
DOTS_OUTPUT_SOURCEGEN_FILES

# Graphics
## RenderMeshArray
*https://docs.unity3d.com/Packages/com.unity.entities.graphics@6.5/changelog/CHANGELOG.html#120-pre12---2024-02-13*