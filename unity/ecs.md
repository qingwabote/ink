# 非托管
ECS 对非托管类型的大量使用，是 Unity 变复杂的根源。

## 内存连续

## Burst 
简化版编译流程 [C#] → [IL] → [Burst LLVM IR] → [Machine Code]

是不是可以说“Burst 是一个 C# 的 **LLVM** 前端”，它自然不能处理**托管类型**

# 多态
**面相对象**下，对象在内存布局中存在虚函数表，调用虚函数存在**间接调用**开销

# Baking
然而 ECS 没有独立的编辑器，为了帮助你将 Unity 编辑器中的 **GameObject** 转化成 **Entity**，于是 Unity 引入了一个更复杂的系统

如果你的 Unmanaged Component 包含集合并参与 Baking 为确保自身的 **Blittable**，你得学习使用 ECS 提供的集合类型：**Dynamic Buffer**(可存放 Entity) 或 **Blob Array**

# [Scenes overview](https://docs.unity3d.com/Packages/com.unity.entities@1.4/manual/conversion-scene-overview.html)
In the entity component system (ECS), scenes work differently. This is because Unity's core scene system is incompatible with ECS. There are the following types of scene concept to understand:
- **Authoring scenes**: An authoring scene is a scene that you can open and edit like any other scene, but is designed for baking to process. It contains GameObjects and MonoBehaviour components that Unity converts to ECS data at runtime.
- **Entity scenes**: An entity scene contains the ECS data that the baking process produces.
- **Subscenes**: A subscene is a reference to an authoring or entity scene. In the Unity Editor, you create a subscene to add authoring elements to. When the subscene is closed, it triggers the baking process for related entity scenes.

上文官方对 Authoring scenes, Entity scenes, Subscenes 的解释让我 confusing, 我将其重新组织给出我目前的理解：  
We create **Subscenes** as **Authoring scenes** that contains GameObjects and MonoBehaviour components, and then bake them into **Entity scenes**.

[Baked entities don’t have a Child buffer. That’s added at runtime](https://discussions.unity.com/t/need-help-with-accessing-child-entities-during-subscene-baking/930559)