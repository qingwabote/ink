# Interleaved vertex buffer
```js
vertexAttribPointer(index, size, type, normalized, stride, offset)
```
WebGL 通过设置 stride 为非 0 的值来暗示 buffer 是 Interleaved 而不是 tightly packed.  
*[If stride is 0, the attribute is assumed to be tightly packed, that is, the attributes are not interleaved but each attribute is in a separate block, and the next vertex' attribute follows immediately after the current vertex](https://developer.mozilla.org/en-US/docs/Web/API/WebGLRenderingContext/vertexAttribPointer)*

同上，图形 API 中并没有显式的 Interleaved buffer 的概念，offset 也是相对于整个 buffer 而不是 buffer 中的 vertex attribute.

对应的 API 在 Vulkan 中为 VkVertexInputAttributeDescription. 但是 offset 在 Vulkan 中有特殊的限制：

**VUID-VkVertexInputAttributeDescription-offset-00622**  
offset must be less than or equal to VkPhysicalDeviceLimits::maxVertexInputAttributeOffset

**VUID-VkVertexInputAttributeDescription-vertexAttributeAccessBeyondStride-04457**  
If the VK_KHR_portability_subset extension is enabled, and VkPhysicalDevicePortabilitySubsetFeaturesKHR::vertexAttributeAccessBeyondStride is VK_FALSE, the sum of offset plus the size of the vertex attribute data described by format must not be greater than stride in the VkVertexInputBindingDescription referenced in binding

而 gltf 的 offset 超出了 VkPhysicalDeviceLimits::maxVertexInputAttributeOffset 的限制 *<https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#_accessor_byteoffset>*

# 资源绑定
## OpenGL
vertexAttribPointer(index, ...) 将 bufer 绑定到 attribute index(location)
```plantuml
skinparam handwritten true

map VertexBuffers {
 pos =>
 col => 
}

object VertexShader {
 layout(location = 0) in vec4 a_position
 layout(location = 1) in vec4 a_color
}

VertexBuffers::pos -> VertexShader::a_position
VertexBuffers::col -> VertexShader::a_color
```
vertexAttribPointer 可以通过 VAO 实现复用，但它与 buffer 耦合，更换 buffer 就得创建新的 VAO。后来被拆成了 VertexAttribFormat 、VertexAttribBinding 和 BindVertexBuffer，VertexAttribBinding 将 attribute 绑定到一个 binding point 上避免与 buffer 直接绑定，运行时使用 BindVertexBuffer 与 VertexArrayVertexBuffer 实现 VAO 的复用（这一点是我的猜测，没有实验过）。

[glVertexAttribPointer 后来拆成了 glVertexAttribFormat 和 glBindVertexBuffer (VkVertexInputAttributeDescription and vkCmdBindVertexBuffers in Vulkan)，概念就清晰了。不过 WebGL 中没有实现。](https://stackoverflow.com/questions/37972229/glvertexattribpointer-and-glvertexattribformat-whats-the-difference)

[While the ArrayBuffer can be filled with both integers and floats, the attributes will always be converted to a float when they are sent to the vertex shader. If you need to use integers in your vertex shader code, you can either cast the float back to an integer in the vertex shader (e.g. (int) floatNumber), or use gl.vertexAttribIPointer() (en-US) from WebGL2.](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGLRenderingContext/vertexAttribPointer#integer_attributes) 由此看来，shader 读取 buffer 时做了数据类型转换，这就是为什么骨骼动画的顶点数组中关节(JOINTS_0) vec4 可以是 unsigned byte(8bits) 或 unsigned short(16bits) 的，而不影响 shader 中使用 uvec4(32bits) 接收数据。

*[understanding glVertexAttribPointer](https://stackoverflow.com/questions/24876647/understanding-glvertexattribpointer)*

## Vulkan
Vulkan 中我们先定义 attribute (VkVertexInputAttributeDescription) 与 **binding point** (VkVertexInputBindingDescription), 这样 pipeline 就确定了。然后在运行时将 buffer 绑定到 **binding point** (vkCmdBindVertexBuffers).

*[Vertex shader input variables are bound to buffers via an **indirect** binding](https://registry.khronos.org/vulkan/specs/1.3-extensions/html/vkspec.html#fxvertex-attrib)*

*[What is the purpose of `binding` from `VkVertexInputBindingDescription`?](https://stackoverflow.com/questions/40450342/what-is-the-purpose-of-binding-from-vkvertexinputbindingdescription)*