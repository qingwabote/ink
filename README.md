# 小游戏 CPU 性能
> 小游戏指 Javascript/Webassembly + WebGL

## 用实时 Skinning 作为参考

Unity 原生由于 WebGL 不支持 Compute Shader, 它的 Skinning 是纯 CPU 运算，所以性能最差：  
https://qingwabote.github.io/fishes-pages/skinning-256-unity/  
开启 SIMD：  
https://qingwabote.github.io/fishes-pages/skinning-256-unity-simd/

Cocos 针对 WebGL 不支持 Compute Shader 的问题，选择了 Vertex Shader Skinning, 顶点变换在 Vertex Shader 中执行，所以比纯 CPU 运算性能好，缺陷是 Vertex Shader 每个 Pass 都执行一遍，对多 Pass 不友好，而且实时运算不走 GPU Instancing：  
https://qingwabote.github.io/fishes-pages/skinning-256-cocos/

我在 Unity ECS 下也实现了 Vertex Shader Skinning, 并且支持 GPU Instancing, 目前为止性能最好：  
https://qingwabote.github.io/fishes-pages/skinning-256-unity-ecs/  
开启 SIMD：  
https://qingwabote.github.io/fishes-pages/skinning-256-unity-ecs-simd/

Webassembly 优于 Javascript, SIMD 对于重向量运算有明显性能提升

## Unity ECS 小游戏性能案例
>[转自《异星幸存者》游戏圈帖子](https://game.weixin.qq.com/cgi-bin/h5/static/circle/detail.html?liteapp=liteapp%3A%2F%2Fwxalite842f9e8076010458697522e7db33761b%3Fpath%3Dpages%252Fdetail%252Findex&wechat_pkgid=circle_detail&tid=hxFdSkCdyG1v6MC79qDvLA#wechat_redirect)

iPhone XR 1792x828, A12 Bionic  
FPS 60  
敌人数量(Enemies) 670
![](https://mmgame.qpic.cn/image/5207e3b83bfd35036db51e04fb9c272f6a2dc43ac872f876437c242e01122470/0)

iPhone SE 2 1334x750, A13 Bionic  
FPS 60  
敌人数量(Enemies) 661
![](https://mmgame.qpic.cn/image/26d96348c1b75ebf740695f7c28f093db948ff0b69101bb3264c977735e25acd/0)

小米8 2248x1080, 骁龙 845, Adreno (TM) 630  
FPS 33  
敌人数量(Enemies) 673
![](https://mmgame.qpic.cn/image/349b62178cd59e6e4551a0f5696edbcaeb279aec4a5b5ec9779507c81d05abef/0)

vivo Z5 2340x1080, 骁龙712, Adreno (TM) 616  
FPS 41  
敌人数量(Enemies) 681
![](https://mmgame.qpic.cn/image/624e92330f7b13bb31f932d034dbb667f92993b2abc8fbd02ffa861b99867a2c/0)
> 截图来自微信云测试