支持 sRGB 的图形引擎可以隐式完成线性与 sRGB 的互转。它**假定**（非常隐晦）在 shader 中使用的是线性空间，采样 sRGB 纹理的值被转换到线性空间，[输出到 sRGB 纹理的值被转换到 sRGB 空间](https://registry.khronos.org/vulkan/specs/1.3-extensions/html/vkspec.html#textures-output-format-conversion).

WebGL 默认的 framebuffer 不支持 sRGB，因而 cocos 干脆不使用 sRGB 纹理, 而是在 shader 中做了 sRGB 的编解码

这与 Unity 的 Gamma 工作流并不是一个概念，Unity 的 Gamma 不仅不使用 sRGB 纹理（忽略纹理的 sRGB（Color Texture） 选项），而且也没有在 shader 中处理，而是直接在 Gamma 空间计算光照。可能是伴随着 URP 才出现了 Linear 工作流，对 sRGB 纹理做了支持，[甚至在材质属性检查器中通过 Color Picker 输入的颜色也被认定为 sRGB, 传入 shader 前做了线性空间转换](https://docs.unity3d.com/2022.3/Documentation/Manual/SL-PropertiesInPrograms.html)，类似的行为也发生在 Light Color 上。 WebGL 下则先把颜色输出到 sRGB FBO 然后跑一个额外的 render pass 做 sRGB 到 Linear 的转换。

# Vulkan
[confusion_about_srgb_image_formats_and_gamma](https://www.reddit.com/r/vulkan/comments/vwn03l/confusion_about_srgb_image_formats_and_gamma/)

# WebGL
[there is currently no way to specify whether or not the default framebuffer will be linear or sRGB. In the case of Chrome v67 it seems that the default framebuffer is linear, so you have to perform the conversion manually in your fragment shader](https://stackoverflow.com/questions/51032480/how-do-you-implement-a-gamma-correct-workflow-in-webgl2)