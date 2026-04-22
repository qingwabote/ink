StreamingAssets 是 Unity 打包资源时留的逃生通道

## 未解之谜 vfs_streamingassets
Entities 构建 WebGL 时使用类似如下路径写入 .data 中
```
"/vfs_streamingassets/EntityScenes/e1b7c7b65bbd8d844afb1c863aab996c.entityheader"
```
问题在于微信小游戏读取时却按
```
"/StreamingAssets/EntityScenes/e1b7c7b65bbd8d844afb1c863aab996c.entityheader"
```
导致异常