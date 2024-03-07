```plantuml
skinparam handwritten true

Batcher2D -> Mask: updateAssembler
Mask -> Mask.Assembler: fillBuffers
Mask.Assembler -> StencilManager: pushMask
Mask.Assembler -> StencilManager: finishMergeBatches
Mask.Assembler -> StencilManager: clear
Mask.Assembler -> StencilManager: enterLevel
Mask.Assembler -> StencilManager: forceMergeBatches

Mask.Assembler -> StencilManager: enableMask

Batcher2D -> Mask: postUpdateAssembler
Mask -> Mask.PostAssembler: fillBuffers
Mask.PostAssembler -> StencilManager: exitMask
```
我猜测"StencilManager: clear"的原理是使用mask内建材质渲染一遍来覆盖模板缓冲对应的区域实现清除，但它如何知道该用多大的尺寸来渲染呢，按节点大小？节点大小每一帧都变化呢？