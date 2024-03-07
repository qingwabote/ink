```plantuml
skinparam handwritten true

:PipelineState DescriptorSet InputAssembler;
:draw;
partition GPU {
    #lightGreen:Input assembler;
    #orange:Vertex shader;
    #orange:Tessellation;
    #orange:Geometry shader;
    #lightGreen:Rasterization;
    #orange:Fragment shader;
    #lightGreen:Color blending;
}
:Framebuffer;
```
*Stages with a green color are known as fixed-function stages.  
Stages with an orange color on the other hand are programmable.  
<https://vulkan-tutorial.com/Drawing_a_triangle/Graphics_pipeline_basics/Introduction>*

*[猴子也能看懂的渲染管线](https://zhuanlan.zhihu.com/p/137780634)*

# cocos 视角
```plantuml
skinparam handwritten true

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
## 可以看出**材质**实际上是渲染管线状态的资源表现形式
```plantuml
skinparam handwritten true

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
*后来找到了一样的想法，“材质正是对管线某一个状态的完整描述 <https://www.cxybb.com/article/6346289/110018037>”；“我们通常用材质文件来描述上述PSO状态数据，Shader数据和贴图数据 <https://my.oschina.net/HMSCore/blog/5067171>”*

# Main Loop
```plantuml
skinparam handwritten true

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

深入理解CocosCreator 3D渲染管线 https://forum.cocos.org/t/topic/101841

Cocos Creator 3D源码简析 https://forum.cocos.org/t/cocos-creator-3d/99630

Cocos Creator 3.0源码漫游 https://forum.cocos.org/t/topic/102023

麒麟子带你快速进入Cocos Creator的3D世界 https://forum.cocos.org/t/topic/124256

# GPU专栏(四) 基于块的渲染(Tile Based Rendering)
https://www.cnblogs.com/Arnold-Zhang/p/15514499.html
# TBR和TBDR
https://zhuanlan.zhihu.com/p/429519726
# 自定义渲染
https://docs.cocos.com/creator/manual/zh/advanced-topics/custom-render.html?q=#%E8%87%AA%E5%AE%9A%E4%B9%89%E6%B8%B2%E6%9F%93

