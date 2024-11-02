支持 sRGB 的图形引擎可以隐式完成线性与 sRGB 的互转。它**假定**（非常隐晦）在 shader 中使用的是线性空间，采样 sRGB 纹理的值被转换到线性空间，[输出到 sRGB 纹理的值被转换到 sRGB 空间](https://registry.khronos.org/vulkan/specs/1.3-extensions/html/vkspec.html#textures-output-format-conversion).

WebGL 无法选择 swapchain 的格式，cocos 里没有使用 sRGB 纹理, 而是在 shader 中做了 gamma 校正。

# Vulkan
[confusion_about_srgb_image_formats_and_gamma](https://www.reddit.com/r/vulkan/comments/vwn03l/confusion_about_srgb_image_formats_and_gamma/)

# WebGL
[there is currently no way to specify whether or not the default framebuffer will be linear or sRGB. In the case of Chrome v67 it seems that the default framebuffer is linear, so you have to perform the conversion manually in your fragment shader](https://stackoverflow.com/questions/51032480/how-do-you-implement-a-gamma-correct-workflow-in-webgl2)