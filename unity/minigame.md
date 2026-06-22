## Unity 6

### WXSDK
安装 WXSDK 后，原生 Web 构建失败，需手动加 webGLEmscriptenArgs: -s ERROR_ON_UNDEFINED_SYMBOLS=0

### Webassembly 2023
随着 WebAssembly.Table 的开启，DYNCALLS 被关闭了，微信和抖音的 JS 库仍然使用旧的 dynCall_viii
- 要么重新打开 webGLEmscriptenArgs: -s DYNCALLS=1
- 或者干脆用基于 Table 的[实现](https://github.com/qingwabote/flavor/blob/main/core/dyncalls.jspre)

### EmscriptenGLX
EmscriptenGLX 中存在通过 EM_JS 调用的 Module API(Module._malloc ...), Unity 不再导出 malloc 和 free 了，使用 EXPORTED_FUNCTIONS 又会覆盖 Unity 的黑箱参数，hack emscripten 的 postamble.js 手动添加也许是最优解
```js
{{{ exportRuntime() }}}

// hack
Module["_malloc"] = _malloc;
Module["_free"] = _free;
Module["lengthBytesUTF8"] = lengthBytesUTF8;
Module["stringToUTF8"] = stringToUTF8;

var calledRun;
```

## Addressables
Unity 构建 WebGL 时将所有资源文件打包到 .data 文件中，引擎初始化解包时重新构建 Emscripten 内存文件系统(MEMFS), AssetBundle 不包含在其中，那么读取本地的 AssetBundle 就成了问题

### File System
> [File System](/unity/flavor.md#file-system)

### AssetBundleProvider
可以通过自定义 [AssetBundleProvider](https://github.com/qingwabote/flavor/blob/main/core/AA/AssetBundleProvider.cs) 把 AssetBundle 与小游戏分包结合起来
> [子包更新机制](/js/minigame.md#子包更新机制)

### SubScene
使用 Addressables 加载 Scene 时并没有按预期的处理 SubScene，事实上 Entities 与 Addressables 互不兼容  
可以在 Addressables 加载 Scene 之前下载 RemoteContentCatalogBuildUtility.BuildContent 产出的 Entities 资源到 /StreamingAssets/, Entities 默认从那里读取

## 更多
[体积优化](/unity/size-optimization.md)