## 一切软件系统都是遗留系统
项目由原来的曹操项目升级引擎而来（cocos2dx 2.2.6 ~ 3.15），替换了许多自行实现的 C++ 模块，如多线程 socket 实现，移植了多数 C++ 实现至 Lua，如与原生语言通信，建立了 Android 多平台解决方案，使用容器技术简化服务器部署。提倡使用Lua扩展，luasocket、lpack、zlib、cjson等，尽量减少对 cocos 引擎的修改，为未来升级引擎提供便利。读 Excel 导出的字节流项目里有两种实现，使用 lpack 的 Lua 实现和自己写的 Lua C 扩展，后者速度数倍于前者，所以在 Lua 和 C 上做了妥协。可以方便的使用部署在自己 mac 上的服务器进行开发得益于容器的使用，容器不可荒废且应更近一步，比如未来可以使用 docker 集群部署线上服务器，彻底统一开发环境。伴随引擎升级，项目使用了新的 API，如使用 cc.Label 代替旧的CCLabel，使用 lua table 代替 CCArray 或 vector，升级时我已移除许多 C++ 实现，但项目里已使用太多，于是我保留了接口而替换为 Lua 实现，见 src/deprecated，不推荐继续使用此文件夹下的 API 写新的功能。

## Git
项目使用开源的 gitlab 托管服务器，架设在.21（ssh -p 2290 root@192.168.1.21）上，管理员 root 。用户验证使用 ssh 公匙，密码与http功能未开启。

## 自动化热更新
使用 cocos 提供的 AssetsManagerEx。逻辑是 C++ 实现，意图是防止热更本身被误更。
内网热更服务已由 assets_server 仓 macmini 分支架设在.23（ssh -p2390 fan0114@jaegergame.asuscomm.com）的/Users/fan0114/skin/assets_server 中。执行其 manifest.sh 会自动发布 res_zh_hans 与 src 仓 master 分支的最新内容。外网更新为避免将客户端仓放置于外网服务器，改用 assets_server 仓 master 分支的 manifest.sh 。版本号会被自动记录在 version 文件里以实现自增。打新包时请跑一遍更新以获得更高版本号的 project.manifest 文件，新包的版本号低于线上版本号是危险的，而高出几个版本是无碍的。

## 打包时自动加密
Lua 加密在 Android Studio gradle 与 Xcode Run Script 中已配置，均使用 cocos XXtea 加密。key 与 sign 在Android Studio 下 gradle.properties 与 Xcode Run Script 中。在 build release 包时加密会自动进行。
没有做图片加密，如需，Texture Packer的加密方案 cocos 中有支持，推荐使用。

## Lua 与原生语言的通信
Lua 与原生语言的通信依赖 cocos LuaJavaBridge和LuaObjcBridge，接入支付与 SDK 依赖它们。LuaJavaBridge.getStaticField 是自己添加的，用于获取 Adndoid Studio 项目当前的 product flavor。Google 与 Apple 支付请参考曹操项目的实现。

## Android 与 IOS多平台
在 Android 使用 Android Studio productFlavors，在 Apple（MacOS 与 IOS）使用 Xcode targets。
项目目前 Andoid 已接入 aone 平台，IOS 则需要参考旧的曹操项目。

## Android 后门
解决“母包”带来的调试困难，Andoid 下应用会优先读取 /sdcard/cocos3/res和/sdcard/cocos3/src 下的文件，所以调试时可以通过修改 Android 手机 sd 卡文件实现在不能重新打包的情况下更新资源和代码，项目里提供自动同步文件到手机的 shell 脚本可供使用，调试完成后请清理 sd 卡下这些文件以免与后来安装的项目冲突。

## 战斗
战斗核心由 BattleManager 中简易的状态机实现，BattleManager:update 每帧按照 actionType 更新状态。playSkillAnimations 可能是最常修改的方法。

## 新手引导与剧情
新手引导与剧情是 guide.lua 实现、guide.json 配置的事件驱动模型，由 LEVEL（升级）、PASS（过关）、TASK_MAIN（领取主线任务）等一个或多个事件触发，它们既是事件，由 EventCenter 触发，又是状态，通过各自的 _validateXXX 进行验证。通过蒙黑并修改 node global zorder 的方式实现高亮。通过对 node 位置和尺寸的计算拦截用户点击。推荐使用 http://www.jsoneditoronline.org/ 作为 Json 编辑器。

## ogg播放
ogg 播放在 apple 设备上使用自己集成的 Fmod，cocos 在 apple 上不提供 ogg 播放功能。

## 服务器部署
内网服务器由 docker-compose 管理，阅读其 yml 即可了解服务器架构。外网服务器仍由 supervisor 管理，qetrunk/run/develop/run.conf 是其现行的配置文件。各个服务的 IP 端口等现行配置见 qetrunk/conf/docker。
需要手动初始化数据库（Mysql 与 Redis）：
- tools/initDB.py 0,worldID1,worldID2...
- tools/serverHunxiaInit.py worldID
- tools/serverJingjiInit.py worldID
- tools/serverPlunderInit.py platform




