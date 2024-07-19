# [How to build V8 on Windows and not go mad](https://medium.com/angular-in-depth/how-to-build-v8-on-windows-and-not-go-mad-6347c69aacd4)

Clone v8 库，选一个[版本](https://v8.dev/docs/version-numbers) tag 创建分支，然后执行 gclient sync(or gclient sync -D --force --reset) (this will reset related repo to corresponding commit).

## Set proxy if necessary:
for CMD
```cmd
set HTTP_PROXY=http://127.0.0.1:58591
set HTTPS_PROXY=http://127.0.0.1:58591
set ALL_PROXY=socks5://127.0.0.1:51837
```
for WSL (在 WSL 下编译的 Android)
```
# get IP
cat /etc/resolv.conf

export HTTP_PROXY=http://172.30.112.1:58591; export HTTPS_PROXY=http://172.30.112.1:58591; export ALL_PROXY=socks5://172.30.112.1:51837
```
for gsutil in depot_tools
```
# boto.txt
[Boto]
proxy=172.30.112.1
proxy_port=58591

export NO_AUTH_BOTO_CONFIG=../boto.txt
```

## Build
执行 python3 tools/dev/v8gen.py x64.release (确保这里 python3 是 depot_tools 里的), 此命令生成目录 out.gn. 
```
C:\Users\qingwabote\Documents\v8\v8>where python3
C:\depot_tools\python3.bat
C:\Users\qingwabote\AppData\Local\Microsoft\WindowsApps\python3.exe
```

执行 gn args out.gn\x64.release (文中 "gn out.gn\x64.release" gn 后少了一个参数 args).

[You must have the version 10.0.20348.0 Windows 10 SDK installed. This can be installed separately or by checking the appropriate box in the Visual Studio Installer](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md#Setting-up-Windows)

[On Windows, this has to be done in the Command Prompt (cmd.exe), as opposed to PowerShell or others.](https://v8.dev/docs/source-code) Windows 上最好全程使用 CMD，如果我使用 PowerShell，甚至 VSCode 集成的 Command Prompt，执行 gn 命令的时候就会报错：
```
C:\Users\qingwabote\Documents\v8\v8>gn args .\out\foo
Waiting for editor on "C:\Users\qingwabote\Documents\v8\v8\out\foo\args.gn"...
Generating files...
Traceback (most recent call last):
  File "C:/Users/qingwabote/Documents/v8/v8/build/toolchain/win/setup_toolchain.py", line 315, in <module>
    main()
  File "C:/Users/qingwabote/Documents/v8/v8/build/toolchain/win/setup_toolchain.py", line 293, in main
    f.write(env_block)
UnicodeEncodeError: 'gbk' codec can't encode character '\u03a2' in position 3043: illegal multibyte sequence
ERROR at //build/toolchain/win/toolchain.gni:500:24: Script returned non-zero exit code.
  win_toolchain_data = exec_script("//build/toolchain/win/setup_toolchain.py",
                       ^----------
Current dir: C:/Users/qingwabote/Documents/v8/v8/out/foo/
Command: C:/depot_tools/bootstrap-2@3_8_10_chromium_23_bin/python3/bin/python3.exe C:/Users/qingwabote/Documents/v8/v8/build/toolchain/win/setup_toolchain.py "C:\Program Files/Microsoft Visual Studio/2022/Community" "C:\Program Files (x86)\Windows Kits\10" "C:\Windows\System32;C:\Windows\SysWOW64;Arm64Unused" win x86 environment.x86
Returned 1.
See //build/toolchain/win/BUILD.gn:34:3: whence it was called.
  win_toolchains("x86") {
  ^-----------------------
See //BUILD.gn:1634:1: which caused the file to be included.
action("postmortem-metadata") {
^------------------------------
```

文中 "build it as a static library", build 出的 v8_base_without_compiler.lib(构建结果里并没有找到 v8_base.lib) 体积比较大
```
is_component_build = false
v8_static_library = true
```
我改用 "is_component_build=true"，构建结果里可以找到 v8 v8_libbase v8_libplatform 的 .dll 与 .lib，这就和 cocos 一致了。  
*https://www.chromium.org/developers/gn-build-configuration/#component-build*  
*https://forum.cocos.org/t/v8-50mb/94029/3?u=qingwabote*

my args.gn
```
target_cpu="x64"

is_debug=false
strip_debug_info=true

is_component_build=true

# 默认 true，是否使用 V8 buildtools/third_party 里自带的那份自定义的 libc++ 库
use_custom_libcxx = false

symbol_level=0

dcheck_always_on = false

# snapshot技术是V8为了提高Context的加载速度引入的优化，将V8启动后的内存布局和JS函数预编译好的二进制对象写到专门的文件shapshot_blob.bin和natives_blob.bin中，并在每一次初始化的时候直接加载，以减少重复编译消耗
v8_use_external_startup_data = false

v8_enable_i18n_support = false
v8_enable_gdbjit=false
v8_enable_test_features=false
v8_enable_pointer_compression=true

treat_warnings_as_errors=false
```
*如果 is_debug=true, 需要删掉 "v8_enable_test_features=false"*

最后 ninja -C out.gn/x64.release

## Android
v8 for android 不能在 windows 下构建，这里走 WSL。depot_tools 可以继续用 windows 上的，v8 源码我另起炉灶了，因为 windows 构建 v8 留下的 windows 的文件，不知道如何清理。
```
target_os = "android"
target_cpu = "arm64"
v8_target_cpu = "arm64"

is_debug=false
strip_debug_info=true

is_component_build=false

# 默认 true，是否使用 V8 buildtools/third_party 里自带的那份自定义的 libc++ 库
use_custom_libcxx = false

symbol_level=0

dcheck_always_on = false

# snapshot技术是V8为了提高Context的加载速度引入的优化，将V8启动后的内存布局和JS函数预编译好的二进制对象写到专门的文件shapshot_blob.bin和natives_blob.bin中，并在每一次初始化的时候直接加载，以减少重复编译消耗
v8_use_external_startup_data = false

v8_enable_i18n_support = false
v8_enable_gdbjit=false
v8_enable_test_features=false
v8_enable_pointer_compression=true

treat_warnings_as_errors=false

v8_static_library=true
v8_monolithic = true
```

参数 v8_monolith 不能省
```
ninja -C out.gn/arm.release v8_monolith
```

install-build-deps.sh 后来我尝试不执行这个命令，好像也没问题。
```
logan@DESKTOP-DR3S8UV:/mnt/c/Users/qingwabote/Documents/v8-wsl/v8$ ./build/install-build-deps.sh
ERROR: The only supported distros are
        Ubuntu 14.04 LTS (trusty with EoL April 2022)
        Ubuntu 16.04 LTS (xenial with EoL April 2024)
        Ubuntu 18.04 LTS (bionic with EoL April 2028)
        Ubuntu 20.04 LTS (focal with Eol April 2030)
        Ubuntu 20.10 (groovy)
        Debian 10 (buster) or later
```

sudo apt-get install pkg-config
```
logan@DESKTOP-DR3S8UV:/mnt/c/Users/qingwabote/Documents/v8/v8$ python3 tools/dev/v8gen.py -vv arm.release
################################################################################
/usr/bin/python3 -u tools/mb/mb.py gen -f infra/mb/mb_config.pyl -m developer_default -b arm.release out.gn/arm.release
  
  Writing """\
  dcheck_always_on = false
  is_debug = false
  target_cpu = "x86"
  v8_target_cpu = "arm"
  """ to /mnt/c/Users/qingwabote/Documents/v8/v8/out.gn/arm.release/args.gn.

  /mnt/c/Users/qingwabote/Documents/v8/v8/buildtools/linux64/gn gen out.gn/arm.release --check
    -> returned 1
  ERROR at //build/config/linux/pkg_config.gni:104:17: Script returned non-zero exit code.
      pkgresult = exec_script(pkg_config_script, args, "value")
                  ^----------
  Current dir: /mnt/c/Users/qingwabote/Documents/v8/v8/out.gn/arm.release/
  Command: python3 /mnt/c/Users/qingwabote/Documents/v8/v8/build/config/linux/pkg-config.py -s /mnt/c/Users/qingwabote/Documents/v8/v8/build/linux/debian_bullseye_i386-sysroot -a x86 glib-2.0 gmodule-2.0 gobject-2.0 gthread-2.0
  Returned 1.
  stderr:

  Traceback (most recent call last):
    File "/mnt/c/Users/qingwabote/Documents/v8/v8/build/config/linux/pkg-config.py", line 248, in <module>
      sys.exit(main())
    File "/mnt/c/Users/qingwabote/Documents/v8/v8/build/config/linux/pkg-config.py", line 143, in main
      prefix = GetPkgConfigPrefixToStrip(options, args)
    File "/mnt/c/Users/qingwabote/Documents/v8/v8/build/config/linux/pkg-config.py", line 81, in GetPkgConfigPrefixToStrip
      prefix = subprocess.check_output([options.pkg_config,
    File "/usr/lib/python3.10/subprocess.py", line 420, in check_output
      return run(*popenargs, stdout=PIPE, timeout=timeout, check=True,
    File "/usr/lib/python3.10/subprocess.py", line 501, in run
      with Popen(*popenargs, **kwargs) as process:
    File "/usr/lib/python3.10/subprocess.py", line 969, in __init__
      self._execute_child(args, executable, preexec_fn, close_fds,
    File "/usr/lib/python3.10/subprocess.py", line 1845, in _execute_child
      raise child_exception_type(errno_num, err_msg, err_filename)
  FileNotFoundError: [Errno 2] No such file or directory: 'pkg-config'
```

sudo apt install ninja-build
```
logan@DESKTOP-DR3S8UV:/mnt/c/Users/qingwabote/Documents/v8/v8$ ninja -C out.gn/arm.release d8
depot_tools/ninja.py: Could not find Ninja in the third_party of the current project, nor in your PATH.
Please take one of the following actions to install Ninja:
- If your project has DEPS, add a CIPD Ninja dependency to DEPS.
- Otherwise, add Ninja to your PATH *after* depot_tools.
```

*https://v8.dev/docs/cross-compile-arm*

# console.log 
API 存在但是不起作用，据说将来会被移除 [The console API is not removed from the global in this CL,
but it is planned to be removed in the later release](https://bugs.chromium.org/p/v8/issues/detail?id=11989)