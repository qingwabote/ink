# [Scenes overview](https://docs.unity3d.com/Packages/com.unity.entities@1.4/manual/conversion-scene-overview.html)
In the entity component system (ECS), scenes work differently. This is because Unity's core scene system is incompatible with ECS. There are the following types of scene concept to understand:
- **Authoring scenes**: An authoring scene is a scene that you can open and edit like any other scene, but is designed for baking to process. It contains GameObjects and MonoBehaviour components that Unity converts to ECS data at runtime.
- **Entity scenes**: An entity scene contains the ECS data that the baking process produces.
- **Subscenes**: A subscene is a reference to an authoring or entity scene. In the Unity Editor, you create a subscene to add authoring elements to. When the subscene is closed, it triggers the baking process for related entity scenes.

上文官方对 Authoring scenes, Entity scenes, Subscenes 的解释让我 confusing, 我将其重新组织给出我目前的理解：  
We create **Subscenes** as **Authoring scenes** that contains GameObjects and MonoBehaviour components, and then bake them into **Entity scenes**.