# 小游戏 CPU 性能
*小游戏指：Javascript/Webassembly + WebGL*

## 用实时 Skinning 作为参考
*为了凸显 CPU 瓶颈，这里没有展示烘焙*

Unity 原生由于 WebGL 不支持 Compute Shader, 它的 Skinning 是纯 CPU 运算，所以性能最差：  
https://qingwabote.github.io/fishes-pages/skinning-256-unity/  
开启 SIMD：  
https://qingwabote.github.io/fishes-pages/skinning-256-unity-simd/

Cocos 针对 WebGL 不支持 Compute Shader 的问题，选择了 Vertex Shader Skinning, 顶点变换在 Vertex Shader 中执行，所以比纯 CPU 运算性能好，缺陷是 Vertex Shader 每个 Pass 都执行一遍，对多 Pass 不友好，而且 Cocos 实时运算不走 GPU Instancing：  
https://qingwabote.github.io/fishes-pages/skinning-256-cocos/

我在 Unity ECS 下也实现了 Vertex Shader Skinning, 目前为止性能最好：
> https://github.com/qingwabote/graphix

https://qingwabote.github.io/fishes-pages/skinning-256-unity-ecs/  
开启 SIMD：  
https://qingwabote.github.io/fishes-pages/skinning-256-unity-ecs-simd/  
*因为实现支持 GPU Instancing 这里顺便给出：*  
*https://qingwabote.github.io/fishes-pages/skinning-256-unity-ecs-instanced/*  
*开启 SIMD：*  
*https://qingwabote.github.io/fishes-pages/skinning-256-unity-ecs-instanced-simd/*

Webassembly 优于 Javascript, SIMD 对于重向量运算有明显性能提升

## Unity 微信小游戏案例
>[转自《异星幸存者》游戏圈帖子](https://game.weixin.qq.com/cgi-bin/h5/static/circle/detail.html?liteapp=liteapp%3A%2F%2Fwxalite842f9e8076010458697522e7db33761b%3Fpath%3Dpages%252Fdetail%252Findex&wechat_pkgid=circle_detail&tid=hxFdSkCdyG1v6MC79qDvLA#wechat_redirect)

iPhone XR 1792x828, A12 Bionic  
FPS 60  
敌人数量(Enemies) 670
<img src="https://mmgame.qpic.cn/image/5207e3b83bfd35036db51e04fb9c272f6a2dc43ac872f876437c242e01122470/0" referrerpolicy="no-referrer">

iPhone SE 2 1334x750, A13 Bionic  
FPS 60  
敌人数量(Enemies) 661
<img src="https://mmgame.qpic.cn/image/26d96348c1b75ebf740695f7c28f093db948ff0b69101bb3264c977735e25acd/0" referrerpolicy="no-referrer">

小米8 2248x1080, 骁龙 845, Adreno (TM) 630  
FPS 33  
敌人数量(Enemies) 673
<img src="https://mmgame.qpic.cn/image/349b62178cd59e6e4551a0f5696edbcaeb279aec4a5b5ec9779507c81d05abef/0" referrerpolicy="no-referrer">

vivo Z5 2340x1080, 骁龙712, Adreno (TM) 616  
FPS 41  
敌人数量(Enemies) 681
<img src="https://mmgame.qpic.cn/image/624e92330f7b13bb31f932d034dbb667f92993b2abc8fbd02ffa861b99867a2c/0" referrerpolicy="no-referrer">

*游戏基于 Unity Entities, 用 [Graphix](https://github.com/qingwabote/graphix) 渲染, 使用了物理引擎（但敌人之间的避障走 RVO）*

> 截图来自微信云测试。

## 如果感兴趣
qingwabote@126.com

*本人正在寻找工作机会坐标**北京***
