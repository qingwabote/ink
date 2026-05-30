Unity 统一小游戏平台 API，抹平小游戏平台差异
> https://github.com/qingwabote/flavor

## File System
参考 Emscripten MEMFS 实现了一个 [MinigameFS](https://github.com/qingwabote/flavor/blob/main/core/fs.jspre) 挂载到 Emscripten 虚拟文件系统(VFS) 的 /StreamingAssets/ 下，把小游戏[原生文件系统](https://github.com/qingwabote/flavor/blob/main/wx/fs_op.jspre)集成到 VFS