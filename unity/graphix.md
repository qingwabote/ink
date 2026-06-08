**Entities Graphics** 渲染库依赖 WebGL 不支持的 Compute Shader，我模仿它的接口实现了兼容 WebGL 的渲染库并命名为 **Graphix**
> https://github.com/qingwabote/graphix

## Skinning
顶点变换在 Vertex Shader 中执行，同时支持 GPU Instancing 和烘焙
| 版本 | 链接 |
|------|------|
| 实时 | https://displaying.pages.dev/skinning-256-unity-ecs-instanced/ |
| 实时 + SIMD | https://displaying.pages.dev/skinning-256-unity-ecs-instanced-simd/ |
| 烘焙 | https://displaying.pages.dev/skinning-256-unity-ecs-instanced-baked/ |

## Batching
Entities Graphics 基于 **BatchRendererGroup(BRG)**
> [BRG WebGL Demo](https://displaying.pages.dev/brg-shooter/)

BRG 使用 float3x4 压缩矩阵，节省带宽

Graphix 基于 Graphics.RenderMeshInstanced 依赖 **MaterialPropertyBlock** 上传自定义实例属性。MaterialPropertyBlock 没有提供非托管数组的接口，为了避免从 burst 到托管数组的拷贝，Graphix Pin 了托管数组在 burst 下直接通过指针读写并建了托管数组池循环使用。输出数组时为了解决 MaterialPropertyBlock.SetFloatArray 不能自定义长度的问题，Graphix 借 List 的壳传递数组，hack 了 List 的 size

RenderMeshInstanced 对 worldToObject matrix 的计算成本似乎并不会随着 **assumeuniformscaling** 开启而被省略
```c
// Put worldToObject array to a separate CB if UNITY_ASSUME_UNIFORM_SCALING is defined. Most of the time it will not be used.
#ifdef UNITY_ASSUME_UNIFORM_SCALING
    #define UNITY_WORLDTOOBJECTARRAY_CB 1
#else
    #define UNITY_WORLDTOOBJECTARRAY_CB 0
#endif
```
> com.unity.render-pipelines.core\ShaderLibrary\UnityInstancing.hlsl


## MaterialPropertyAttribute
Graphix 兼容 MaterialPropertyAttribute

## EntitiesGraphicsSystem
Graphix 兼容 EntitiesGraphicsSystem, 用于运行时注册 Material 和 Mesh.

## Scene View Mode
Entities Graphics 控制是否在 Scene View 中显示的实现原理尚不明确
> https://docs.unity3d.com/Packages/com.unity.entities@1.4/manual/editor-authoring-runtime.html

在编辑器中观察到 Entities 会在 Scene View Mode 为 Authoring Data 时为 Entity 添加 **EditorRenderData** 并赋值 SceneCullingMask 为 **SceneCullingMasks.GameViewObjects** 告知 Entities Graphics 仅在 Game View 中显示

因为尚不清楚为什么官方要通过 EditorRenderData 实现在组件级上的控制，Graphix 没有遵循这种做法，而是仅通过 **LiveConversionEditorSettings.LiveConversionMode** 在全局上控制