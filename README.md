# 小游戏 CPU 性能
*小游戏指：Javascript/Webassembly + WebGL*

## Javascript 的瓶颈
*为了凸显 CPU 瓶颈，这里只跑了实时 Skinning 的测试*

**Unity 原生**由于 WebGL 不支持 Compute Shader, 它的 Skinning 是纯 CPU 运算，所以性能最差  
> https://qingwabote.github.io/fishes-pages/skinning-256-unity/  
开启 SIMD：  
https://qingwabote.github.io/fishes-pages/skinning-256-unity-simd/

**Cocos** 针对 WebGL 不支持 Compute Shader 的问题，选择了 Vertex Shader Skinning, 顶点变换在 Vertex Shader 中执行，所以比纯 CPU 运算性能好，缺陷是 Vertex Shader 每个 Pass 都执行一遍，对多 Pass 不友好，而且 Cocos 实时运算不走 GPU Instancing  
> https://qingwabote.github.io/fishes-pages/skinning-256-cocos/

**Unity ECS** 下我在 [Graphix](unity/graphix.md) 中实现了 Vertex Shader Skinning, 把性能找补了回来，并且超越了 Cocos
> https://qingwabote.github.io/fishes-pages/skinning-256-unity-ecs/  
开启 SIMD：  
https://qingwabote.github.io/fishes-pages/skinning-256-unity-ecs-simd/  
*因为实现支持 GPU Instancing 这里顺便给出：*  
*https://qingwabote.github.io/fishes-pages/skinning-256-unity-ecs-instanced/*  
*开启 SIMD：*  
*https://qingwabote.github.io/fishes-pages/skinning-256-unity-ecs-instanced-simd/*

所以性能上 Unity(Webassembly) 优于 Javascript, 而且可以利用 SIMD 进一步提升

## 连续内存访问
Javascript 没有复合类型的值对象，很难实现在内存中连续访问一批对象，对缓存不友好。Unity C# 则可以利用值类型将对象放入连续的内存当中。
### Unity Entities
Entities 在 WebGL 下的最大兼容问题在于配套的 Entities Graphics 依赖 Compute Shader, 所以我自己实现了
Entities 的渲染库 [Graphix](unity/graphix.md)  
Entities 与 Addressables 互不兼容，若需要分包加载，参考 [Addressables](unity/minigame.md#addressables)

## Unity Minigame Showcase
<img src="https://mmgame.qpic.cn/image/95abb1c4bd6c789f35717eb40464e02a2d3f8be8868c4de3915bf09725b69390/0" referrerpolicy="no-referrer" width="128">

### 低端设备微信云测试

> 测试场景：敌人数量达到约 670 时低端设备的表现

#### iPhone XR
- 分辨率：1792 × 828
- SoC：A12 Bionic
- FPS：60
- 敌人数量：670

<img src="https://mmgame.qpic.cn/image/5207e3b83bfd35036db51e04fb9c272f6a2dc43ac872f876437c242e01122470/0" referrerpolicy="no-referrer">

---

#### iPhone SE (第 2 代)
- 分辨率：1334 × 750
- SoC：A13 Bionic
- FPS：60
- 敌人数量：661

<img src="https://mmgame.qpic.cn/image/26d96348c1b75ebf740695f7c28f093db948ff0b69101bb3264c977735e25acd/0" referrerpolicy="no-referrer">

---

#### 小米 8
- 分辨率：2248 × 1080
- SoC：骁龙 845
- GPU：Adreno 630
- FPS：33
- 敌人数量：673

<img src="https://mmgame.qpic.cn/image/349b62178cd59e6e4551a0f5696edbcaeb279aec4a5b5ec9779507c81d05abef/0" referrerpolicy="no-referrer">

---

#### vivo Z5
- 分辨率：2340 × 1080
- SoC：骁龙 712
- GPU：Adreno 616
- FPS：41
- 敌人数量：681

<img src="https://mmgame.qpic.cn/image/624e92330f7b13bb31f932d034dbb667f92993b2abc8fbd02ffa861b99867a2c/0" referrerpolicy="no-referrer">

---

#### 测试结果汇总

| 低端设备 | SoC | 分辨率 | FPS | 敌人数量 |
|--------|--------|--------|--------:|--------:|
| iPhone XR | A12 Bionic | 1792×828 | 60 | 670 |
| iPhone SE 2 | A13 Bionic | 1334×750 | 60 | 661 |
| 小米 8 | 骁龙 845 + Adreno 630 | 2248×1080 | 33 | 673 |
| vivo Z5 | 骁龙 712 + Adreno 616 | 2340×1080 | 41 | 681 |

#### 总大小 19503 KB
| Path                                      | Size      |
|:------------------------------------------|:----------|
| `/wasmcode/`                              | 5695.6 KB |
| `/StreamingAssets/aa/WebGL/moon`          | 3050.2 KB |
| `/StreamingAssets/ContentArchives`        | 2550.1 KB |
| `/StreamingAssets/aa/WebGL/duplicate`     | 2083.8 KB |
| `/data-package/`                          | 1892.1 KB |
| `/StreamingAssets/aa/WebGL/title`         | 1568.7 KB |
| `/StreamingAssets/EntityScenes`           | 1392.0 KB |
| `main`                                    | 1270.3 KB |

#### 系统与实现
| System            | Implementation                         |
|:------------------|:---------------------------------------|
| ECS               | Unity Entities                         |
| Rendering         | [Graphix](unity/graphix.md)            |
| Physics           | Unity Physics                          |
| Crowd Avoidance   | RVO（用于敌人之间的避障）                |
| Platform          | [Flavor](unity/flavor.md)              |

> [Unity 微信小游戏适配](unity/minigame.md)

## 如果感兴趣
qingwabote@126.com

*本人正在寻找工作机会坐标**北京***
