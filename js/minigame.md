# 微信小游戏原生 JS 模块

## CMD 模块

微信小游戏目前**没有**提供对**原生 ES 模块**的支持。

_如果在微信开发者工具中关闭“将 JS 编译成 ES5”，原生 ES 模块将引发错误"SyntaxError: Unexpected token 'export'", "Cannot use import statement outside a module"_

微信开发者工具编译 js 源码时无差别注入 CMD API，即便源码已经是 CMD 模块。结果像这样

```js
define("foo.js", function (require, module, exports) {
  define("foo.js", function (require, module, exports) {
    console.log("foo");
  });
});
```

据此猜测工具编译的过程是将 js 源码无差别编译为 **CommonJS**，然后将结果首尾注入 CMD API.

_[CMD](https://github.com/cmdjs/specification/blob/master/draft/module.md)_

## npm-package or import-maps

我猜测微信对 npm-package 的支持是基于 import-maps 实现的，但并没有暴露 import-maps 给开发者使用。

## TLA(Top-level await) 与 import-maps
基于微信小游戏原生 JS 模块实现的限制，若要使用 TLA 或 import-maps, 可以借助 [SystemJs](https://github.com/qingwabote/zero/blob/master/minigame/README.md)

# WebGL2
WebGL2 在微信[基础款版本](https://developers.weixin.qq.com/minigame/dev/guide/runtime/client-lib/version.html) 2.24.0 中提供了支持

[高性能模式](https://developers.weixin.qq.com/minigame/dev/guide/performance/perf-high-performance.html)在 [Safari iOS15](https://caniuse.com/webgl2) 中提供了 WebGL2 支持，猜测高性能模式跑在 WKWebView 中(为了使用 JIT) 

[高性能+模式](https://developers.weixin.qq.com/minigame/dev/guide/performance/perf-high-performance-plus.html) [将原本的高性能框架中的webGL渲染，通过跨进程通讯技术，交由了微信客户端的原生渲染实现](http://www.gamelook.com.cn/2024/01/536242)，iOS 版本降到 14, 基础库版本 3.3.2. 跨进程通讯产生的带宽会大大增加场景加载的时间，期间 [XHR 次数(文档没有解释什么是 XHR)](https://gitee.com/wechat-minigame/minigame-unity-webgl-transform/blob/main/Design/PowerPerf-iOS.md) 会飙升到上千

# 引擎插件
[目前一个小游戏APPID，仅支持引用一个引擎插件。游戏引擎目前支持未经修改的 Cocos、白鹭、LayaAir 1.0、LayaAir 2.0 引擎版本。](https://developers.weixin.qq.com/community/develop/doc/0000225f290c50132d791596756400)没有找到普通开发者提交引擎插件的途径。[如果使用引擎插件功能，包的总大小会算上线上插件里的引擎代码。](https://segmentfault.com/a/1190000022066660)

# 更新机制（子包）
体验版只有一个版本，所以 loadSubpackage 直接加载新版本子包，如果在游戏过程中有新版本发布，触发加载时别指望还能加载到旧版本（已经不存在了）

线上版，旧版本加载不到新版本子包（我只试过一次，发布新版本后，发现自己操作失误提前加载了子包，随即刷新游戏，因为第一次冷启动不一定会应用更新，等了二十分钟后加载子包，万幸结果仍是旧版本的）