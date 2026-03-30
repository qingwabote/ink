# Unity 6
即适配新版本的 Emscripten

## Webassembly 2023
随着 WebAssembly.Table 的开启，DYNCALLS 被关闭了，微信和抖音的 JS 库仍然使用旧的 dynCall_viii
- 要么重新打开 webGLEmscriptenArgs: -s DYNCALLS=1
- 或者干脆用基于 Table 的[实现](https://github.com/qingwabote/bastard/blob/main/Plugins/DYNCALLS.jspre)

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