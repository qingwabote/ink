# Unity 6 微信小游戏适配
## Webassembly 2023
如果开启 WebAssembly.Table 就得手动开启 DYNCALLS(-s DYNCALLS=1), 因为微信的 js 库中依赖旧的 dynCall_xxx API
## EmscriptenGLX
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