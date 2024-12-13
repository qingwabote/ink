GPU 设备是一个**状态机**，顶点数据是**输入**，而使用什么输入格式，什么着色器，什么全局变量，怎样剔除面，如何做深度测试、颜色混合等等是其**状态**（这些状态的一部分对应的就是游戏引擎中的**材质**），帧缓冲是其**输出**。**Draw Call** 是状态机启动的命令，它将启动流水线，按照这些状态，加工输入的顶点数据，完成一次帧缓冲的写入。

不难看出，如果将相同材质模型的**顶点数据合并(Dynamic Batching)**，就可以一次 Draw Call 绘制多个模型。但合并顶点数据也要消耗 CPU 算力，比如坐标变换。

3D 模型顶点数量巨大，通常设置变换矩阵为全局变量以在着色器中完成顶点变换实现模型的移动：模型A → 变换A → draw, 模型B → 变换B → draw. 这里即使相同的模型，相同的材质，也要消耗多个 Draw Call. 如果将变换矩阵写入到每个顶点的数据中，空间、算力又吃不消。硬件的 **GPU Instancing** 技术为此而生，Draw call 内部循环自身，每次循环额外从[特殊的顶点缓冲](https://www.khronos.org/opengl/wiki/Vertex_Specification#Instanced_arrays)中读取一次数据（或者使用 GLSL 内置变量 gl_InstanceID 去 uniform buffer(size limitation), texture 或任何其它位置索引数据），比如变换矩阵，实现每次循环绘制模型的一个“实例”。

*我没有见到使用 GPU Instancing 渲染 2D 图形（比如 Sprite） 的引擎，全是 Dynamic Batching, 我想知道原因？*

*[Do not use more than one instanced vertex buffer per draw call?](https://developer.arm.com/documentation/101897/0302/Vertex-shading/Instanced-vertex-buffers)*

## CPU → GPU
CPU 需要从用户模式切换到内核模式与 GPU 通讯，这是耗时的，因此图形的 Runtime/Driver 将 Draw Call 以及它依赖的那些前置命令预先**编码**到 Command Buffer 再一起发送给 GPU，在此过程中还可以对命令进行**验证**，这些都消耗 CPU 算力。优化时所说的 “减少 Draw Call” 里的 Draw Call 指的因该是 “Draw Call 以及它依赖的那些前置命令”，我甚至尝试过在保证其它命令不变的条件下只将 Draw Call 拆分成多个，CPU 性能下降并不显著。 


*[Optimizing draw calls](https://docs.unity3d.com/cn/2023.2/Manual/optimizing-draw-calls.html)*

*[glVertexAttribDivisor modifies the rate at which generic vertex attributes advance when rendering multiple instances of primitives in a single draw call (see glDrawArraysInstanced and glDrawElementsInstanced). If divisor is zero, the attribute at slot index advances once per vertex. If divisor is non-zero, the attribute advances once per divisor instances of the set(s) of vertices being rendered. An attribute is referred to as instanced if its GL_VERTEX_ATTRIB_ARRAY_DIVISOR value is non-zero.](https://www.khronos.org/registry/OpenGL-Refpages/es3.0/html/glVertexAttribDivisor.xhtml)*

*[I have personally never used divisor values larger than 1, but it's perfectly valid. Instead of using a value for each instance, you can use the same value for n instances. But IMHO, by far the most useful values are 0 and 1: 0, the default, specifies that the attribute behaves as usual, meaning that the value is fetched based on the vertex index. 1 specifies that you have one attribute value per instance, and the value is fetched based on the instance id (gl_InstanceID).](https://stackoverflow.com/questions/31398169/how-attribute-divisor-works-with-indexed-drawing)*

## Cocos GPU Instancing
editor\assets\chunks\cc-local-batch.chunk
```glsl
#if USE_INSTANCING
  in vec4 a_matWorld0;
  in vec4 a_matWorld1;
  in vec4 a_matWorld2;
  #if USE_LIGHTMAP
    in vec4 a_lightingMapUVParam;
  #endif
#elif USE_BATCHING
  in float a_dyn_batch_id;
  #define BATCHING_COUNT 10
  #pragma builtin(local)
  layout(set = 2, binding = 0) uniform CCLocalBatched {
    highp mat4 cc_matWorlds[BATCHING_COUNT];
  };
#else
  #include <cc-local>
#endif
```

每个顶点属性最多 4 个分量, 所以这里把 mat4 matWorld 拆成了 vec4×3, 这里齐次坐标是常量，所以省略了 1 个  
editor\assets\chunks\builtin\functionalities\world-transform.chunk
```glsl
void CCGetWorldMatrix(out mat4 matWorld)
{
  #if USE_INSTANCING
    matWorld = mat4(
      vec4(a_matWorld0.xyz, 0.0),
      vec4(a_matWorld1.xyz, 0.0),
      vec4(a_matWorld2.xyz, 0.0),
      vec4(a_matWorld0.w, a_matWorld1.w, a_matWorld2.w, 1.0)
    );
  #else
    matWorld = cc_matWorld;
  #endif
}
```
CPU 每一帧进行“实例缓冲合并”，Cocos 通过直接对比 indexBuffer 来分组
```ts
    public merge (subModel: SubModel, attrs: IInstancedAttributeBlock, passIdx: number, shaderImplant: Shader | null = null) {
        ...
        for (let i = 0; i < this.instances.length; ++i) {
            const instance = this.instances[i];
            if (instance.ia.indexBuffer !== sourceIA.indexBuffer || instance.count >= MAX_CAPACITY) { continue; }
            ...
            instance.data.set(attrs.buffer, instance.stride * instance.count++);
            this.hasPendingModels = true;
            return;
        }

        // Create a new instance
        const vb = this._device.createBuffer(new BufferInfo(
            BufferUsageBit.VERTEX | BufferUsageBit.TRANSFER_DST,
            MemoryUsageBit.HOST | MemoryUsageBit.DEVICE,
            stride * INITIAL_CAPACITY,
            stride,
        ));
        ...
    }
```
*cocos\core\pipeline\instanced-buffer.ts*

2D 对象没有模型，以 Canvas（RenderRoot2D）为入口每帧收集数据合并顶点。


## 合图
将图片纹理合并使合批不被打断。
### 静态合图（自动图集）
### 动态合图
利用图形引擎部分更新纹理的功能（glTexSubImage2D），将多个图像上传到一个纹理中，来实现动态图集（不支持压缩纹理）。  
```ts
public _checkPackable () {
    const dynamicAtlas = dynamicAtlasManager;
    if (!dynamicAtlas) return;
    const texture = this._texture;

    if (!(texture instanceof Texture2D) || texture.isCompressed) {
        this._packable = false;
        return;
    }

    const w = this.width;
    const h = this.height;
    if (!texture.image
        || w > dynamicAtlas.maxFrameSize || h > dynamicAtlas.maxFrameSize) {
        this._packable = false;
        return;
    }

    if (texture.image && texture.image instanceof HTMLCanvasElement) {
        this._packable = true;
    }
}
```
*cocos\2d\assets\sprite-frame.ts*

# Draw Call 性能消耗原因
https://www.reddit.com/r/vulkan/comments/g5bh31/why_are_drawcalls_setpass_calls_so_expensive/

https://gwb.tencent.com/community/detail/113040