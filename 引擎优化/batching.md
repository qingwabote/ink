# 静态合批（Static Batching）
把一个节点下的所有子节点的 MeshRenderer 的 mesh 合并成一个大 mesh, 合并后自然无法修改原有的子节点（节点已经不存在了），所以称为静态合批。
可以每帧重新合并实现动态合批，这也是 2D 中的合并手段。考虑到 3D 模型巨大的顶点数量在合并时带来的 CPU 开销，cocos 3.6.2 已移除了 3D 下的每帧合并功能（合并 VB 合批）

# 动态合批（Dynamic Batching）

## 实例化（GPU Instancing）
如果将原本使用 **uniform** 传输的数据（比如世界变换矩阵）也存入**顶点属性缓冲**，这样当然可以实现合批，但为每个顶点写入一份数据造成顶点属性缓冲中大量的重复。

Draw call 可以内部循环自身，并且每次循环额外从指定的顶点缓冲中读一个数据，每次循环绘制一个“实例”。

OpenGL 中给指定顶点属性的 divisor 设置成一个非 0 的数（大于 1 有什么意义吗？），shader 将不再通过索引获取该顶点属性的值，而是从该属性绑定的顶点缓冲中获取，每隔 divisor 个实例获取一次，这样就得到了一个“实例缓冲”，然后用 **drawElementsInstanced** 代替 drawElements。

*"glVertexAttribDivisor modifies the rate at which generic vertex attributes advance when rendering multiple instances of primitives in a single draw call (see glDrawArraysInstanced and glDrawElementsInstanced). If divisor is zero, the attribute at slot index advances once per vertex. If divisor is non-zero, the attribute advances once per divisor instances of the set(s) of vertices being rendered. An attribute is referred to as instanced if its GL_VERTEX_ATTRIB_ARRAY_DIVISOR value is non-zero." <https://www.khronos.org/registry/OpenGL-Refpages/es3.0/html/glVertexAttribDivisor.xhtml>*

*"I have personally never used divisor values larger than 1, but it's perfectly valid. Instead of using a value for each instance, you can use the same value for n instances. But IMHO, by far the most useful values are 0 and 1:  
0, the default, specifies that the attribute behaves as usual, meaning that the value is fetched based on the vertex index.  
1 specifies that you have one attribute value per instance, and the value is fetched based on the instance id (gl_InstanceID)." <https://stackoverflow.com/questions/31398169/how-attribute-divisor-works-with-indexed-drawing>*

### Cocos 的实现
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
CPU 每一帧进行“实例缓冲合并”，Cocos 通过直接对比 indexBuffer 来分组，这意味着 Mesh 必须相同，Material 相同
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

# 2D
在 Cocos 中，2D 对象没有模型，以 Canvas（RenderRoot2D）为入口收集数据并自动执行合批。

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