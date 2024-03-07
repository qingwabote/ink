# 2D,3D,UI
Unity 中 Canvas 为 UI 提供了一个根节点，它可以在 screen space 或 world space. UI 节点的 layer 只在 world space 下有意义。Canvas 与 Canvas 之间的渲染顺序通过属性 SortingLayer 设置。

Cocos(v3.7.1) 中 Canvas 只在 screen space 中, 如果你尝试修改它的坐标（它的坐标在面板里被锁死了，你可以移动它的父节点试试）是没有意义的。不过，仍然可以创建多个 Canvas 用于 [2D 和 3D 的混排](https://forum.cocos.org/t/2d-3d-2d-3d/85020/4?u=qingwabote)。[Canvas 与 Canvas 之间的渲染顺序只能由分配不同的 Camera 实现](https://forum.cocos.org/t/topic/119447/8)。