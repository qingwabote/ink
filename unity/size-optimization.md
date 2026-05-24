## Code

### Web Stripping Tool package for Unity 6
https://docs.unity3d.com/Packages/com.unity.web.stripping-tool@1.3/manual/index.html

### IL2CPP Code Generation - Optimize for code size
IL2CPP Code Generation 设置为 Optimize for code size 会开启 **Full Generic Sharing**

### Web Platform Settings - Code Optimization - Disk Size with LTO
LTO 能在 Disk Size 的基础上进一步减少体积

### UI Toolkit
**UGUI** 检测到 UIElements 包，就会产生引用，即便是运行时，并不像注释所说的 "it can be stripped from a project build if unused".
```
"versionDefines": [
    {
        "name": "com.unity.modules.uielements",
        "expression": "1.0.0",
        "define": "PACKAGE_UITOOLKIT"
    }
],
```
```c#
        // This code is disabled unless the com.unity.modules.uielements module is present.
        // The UIElements module is always present in the Editor but it can be stripped from a project build if unused.
#if PACKAGE_UITOOLKIT
        [SerializeField, HideInInspector] private UIToolkitInteroperabilityBridge m_UIToolkitInterop = new ();

        /// <summary>
        /// Use this property to initialize UI Toolkit interoperability with uGUI events.
        /// </summary>
        internal UIToolkitInteroperabilityBridge uiToolkitInterop => m_UIToolkitInterop;
#endif
```
去掉这个对 UIElements 的 versionDefine 即可。
> https://github.com/qingwabote/com.unity.ugui/commit/fc2e293c8a15656d6369f376f301746e38b9ce18

**Input System** 则直接依赖 UIElements. 好在对 UIElements 的代码引用大都在 #if UNITY_EDITOR 下, 处理掉几个特例删掉 dependencies 即可
```
  "dependencies": {
    "com.unity.modules.uielements": "1.0.0"
  }
```
> https://github.com/qingwabote/com.unity.inputsystem/commit/9c8eee7c83696c9d5d6325d785c61ddcbe21e049

### Entities baking
Unity 并不会在构建时自动排除 [BakingType], Baker 与 Baking System, 更不会自动排除 Authoring Component. 由于 Authoring Component 是 MonoBehaviour 甚至无法放入 editor 程序集，只能用宏 UNITY_EDITOR 加以排除。官方的 entities 相关 package 在这点上也是混乱的

## Data

### IL2CPP global-metadata.dat
Web Stripping Tool 并不会将 data 文件里的类型的元数据一并剔除

### Addressables & Entities
Addressables 与 Entities 互不兼容。Entities 不会从 Addressables bundle 拉取资源，而是自己储存，所以Entities 里不要显示引用 Addressables bundle 的资源依赖“自动”管理

### Use larger block sizes for ASTC compressed textures
[When ASTC is selected in build settings (or Player Settings for Unity versions that have it) the ASTC block size depends on the “Compression” dropdown in the Default platform tab of the TextureImporter UI (Normal=6x6, Low=8x8, High=4x4)](https://discussions.unity.com/t/what-astc-compression-used-in-build-settings-corresponds-to-the-one-used-in-texture-import-settings/869993/2)

### 移除 SkyBox
将每个场景 Lighting → Environment 的 SkyboxMaterial 置为 None 并从 AlwaysIncludedShaders 中移除诸如 Hidden/CubeBlur 的几个 shader

### Splash Screen Unity Logo
去掉勾选 Show Splash Screen 并不能代表会同时去掉 Show Unity Logo
> https://discussions.unity.com/t/splash-screen-logo-texture-2-7mb-included-in-build-even-when-disabled-unity-pro/1681099