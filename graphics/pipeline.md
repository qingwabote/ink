```plantuml
!option handwritten true

:PipelineState DescriptorSet InputAssembler;
:draw;
partition GPU {
    :Input assembler;<<#lightGreen>>
    :Vertex shader;<<#orange>>
    :Tessellation;<<#orange>>
    :Geometry shader;<<#orange>>
    :Rasterization;<<#lightGreen>>
    :Fragment shader;<<#orange>>
    :Color blending;<<#lightGreen>>
}
:Framebuffer;
```
*Stages with a green color are known as fixed-function stages.  
Stages with an orange color on the other hand are programmable.  
<https://vulkan-tutorial.com/Drawing_a_triangle/Graphics_pipeline_basics/Introduction>*

## Cocos 视角
```plantuml
!option handwritten true

class InputAssembler #lightGreen

MeshRenderData o-- InputAssembler
Renderable2D o-- IAssembler
IAssembler o-- RenderData
IAssembler o-- MeshRenderData
Renderable2D o-- Batcher2D
Batcher2D o-- StaticVBAccessor
StaticVBAccessor o-- MeshBuffer
MeshBuffer o-- InputAssembler

MeshRenderer o-- Mesh
MeshRenderer o-- Model
Model o-- SubModel
SubModel o-- InputAssembler

note left of IAssembler : 无状态的模块\n用来组装 render data
note left of RenderData : 对应不可变顶点\n例如 sprite
note bottom of MeshRenderData : 对应可变顶点\n例如 spine
note right of StaticVBAccessor : 对应 RenderData
note right of MeshBuffer : 收集相同格式的顶点
note bottom of InputAssembler: 这里的 InputAssembler 与 vulkan api 里的 InputAssemblyState 不是一个概念。\nInputAssembler 包含顶点缓冲，顶点索引缓冲、顶点属性信息... 在 webgl 中正好对应一个 VAO
```
### **材质**是渲染管线一部分状态的资源表现形式
```plantuml
!option handwritten true

class RasterizerState #lightGreen
class DepthStencilState #lightGreen
class BlendState #lightGreen
class Shader #orange

Material o-- Pass

Pass o-- RasterizerState
Pass o-- DepthStencilState
Pass o-- BlendState
Pass o-- PipelineLayout
Pass o-- DescriptorSet
Pass o-- Shader

RasterizerState --o PipelineState
DepthStencilState --o PipelineState
BlendState --o PipelineState
PipelineLayout --o PipelineState
Shader --o PipelineState
```
*[材质正是对管线某一个状态的完整描述](https://www.cxybb.com/article/6346289/110018037>)；[我们通常用材质文件来描述上述PSO状态数据，Shader数据和贴图数据](https://my.oschina.net/HMSCore/blog/5067171)*

## main loop
```plantuml
!option handwritten true

Root -> Batcher2D: update
Batcher2D -> Renderable2D: updateAssembler
Renderable2D -> IAssembler: updateRenderData
Renderable2D -> Batcher2D: commitComp
Batcher2D -> IAssembler: fillBuffers
IAssembler -> MeshBuffer: "update vData, iData"

Root -> Batcher2D: uploadBuffers
Batcher2D -> MeshBuffer: uploadBuffers
Batcher2D -> LocalDescriptorSet: updateLocal ubo

Root -> RenderScene: update
RenderScene -> Model: updateUBOs

Root -> RenderPipeline: render
RenderPipeline -> RenderStage: render
RenderStage -> CommandBuffer: draw
```

