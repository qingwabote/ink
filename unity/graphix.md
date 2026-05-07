**Entities Graphics** 渲染库依赖 WebGL 不支持的 Compute Shader，我模仿它的接口实现了兼容 WebGL 的渲染库并命名为 **Graphix**
> https://github.com/qingwabote/graphix

## GPU Instancing
Entities Graphics 使用 **BatchRendererGroup(BRG)** 但 Graphix 选择了 **Graphics.RenderMeshInstanced**, 这只是为了降低开发难度

## MaterialPropertyAttribute
Graphix 兼容 MaterialPropertyAttribute

## EntitiesGraphicsSystem
Graphix 兼容 EntitiesGraphicsSystem, 用于运行时注册 Material 和 Mesh.

## Batching

## Scene View Mode
Entities Graphics 控制是否在 Scene View 中显示的实现原理尚不明确
> https://docs.unity3d.com/Packages/com.unity.entities@1.4/manual/editor-authoring-runtime.html

在编辑器中观察到 Entities 会在 Scene View Mode 为 Authoring Data 时为 Entity 添加 **EditorRenderData** 并赋值 SceneCullingMask 为 **SceneCullingMasks.GameViewObjects** 告知 Entities Graphics 仅在 Game View 中显示

因为尚不清楚为什么官方要通过 EditorRenderData 实现在组件级上的控制，Graphix 没有遵循这种做法，而是仅通过 **LiveConversionEditorSettings.LiveConversionMode** 在全局上控制