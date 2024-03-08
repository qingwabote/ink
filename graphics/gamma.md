支持 sRGB 的图形引擎会自动完成线性与 sRGB 的互转。它假定在 shader 中使用的是线性空间，shader 中采样 sRGB 格式的纹理时自动将 sRGB 转换到线性，shader 输出给 sRGB 格式的纹理的值被自动转换到 sRGB.

但是，WebGL 无法指定 swapchain 的格式，我们可能得做比如自行创建 sRGB 的 framebuffer 然后在拷贝到 swapchain 的额外操作。所以 cocos 里没有使用 sRGB 格式, 而是在 shader 中做了 gamma 校正。

# Vulkan
[confusion_about_srgb_image_formats_and_gamma](https://www.reddit.com/r/vulkan/comments/vwn03l/confusion_about_srgb_image_formats_and_gamma/)

# WebGL
[there is currently no way to specify whether or not the default framebuffer will be linear or sRGB. In the case of Chrome v67 it seems that the default framebuffer is linear, so you have to perform the conversion manually in your fragment shader](https://stackoverflow.com/questions/51032480/how-do-you-implement-a-gamma-correct-workflow-in-webgl2)