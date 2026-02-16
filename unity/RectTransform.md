# RectTransform 完全(不)理解

## Pivot
建模的时候坐标原点“放在” cube 的几何中心还是 soldier 的脚下，那 Image 和 TMP 呢？

UI 在运行时根据 Pivot 生成顶点，虽然也可以给 cube 添加 RectTransform，但是 cube 的 mesh 不是运行时的生成的，修改 Pivot 没有意义。添加一个组件但没有预期的效果，如果 Unity 限定 RectTransform 仅用于 UI，那么 3D TMP 呢，我现在可以理解我为什么不理解了

## Anchors