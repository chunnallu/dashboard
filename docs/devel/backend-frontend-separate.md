# 后端搭建

## 搭建后端环境

第一次运行，请先[安装项目环境](requirements-installation.md)

- 启动docker服务
```text
sudo service docker start
```

- 启动kubernetes集群

```text
sudo gulp local-up-cluster
```

- 启动前后端服务器
```text
sudo "PATH=$PATH" gulp serve
```
### 搭建后端环境可能会遇到的问题

1、运行`gulp local-up-cluster`一直`waitting for a heapster...`

打开build/conf.js文件，将：
```text
 heapsterServerHost: gulpUtil.env.heapsterServerHost !== undefined ? gulpUtil.env.heapsterServerHost : '',
```
改成:
```text
 heapsterServerHost: gulpUtil.env.heapsterServerHost !== undefined ? gulpUtil.env.heapsterServerHost : '127.0.0.1:8082',
```

# 前端搭建

前端运行在windows上，通过api调用后端的服务。所以在windows上也只需要安装前端的依赖。

## 搭建前端环境

- 安装node
  
  下载[nodejs安装包](https://nodejs.org/zh-cn/download/),并安装
  
- 安装python

  下载[python 2安装包](https://www.python.org/downloads/release/python-2713/)，并安装

- 以管理员身份运行cmd，并切换到项目路径

- 安装前端依赖

```text
npm  install
npm install -g bower
bower install
```
- 运行起前端开发服务器
```text
gulp serve:forntend
```

### 搭建前端环境可能遇到的问题

1、运行`npm install`报错

（1）首先，必须以管理员身份运行`npm install`
（2）如果报“MSBUILD : error MSB4132: 无法识别工具版本“2.0”。可用的工具版本为 "4.0" ”错误，如下：
```text
MSBUILD : error MSB4132: 无法识别工具版本“2.0”。可用的工具版本为 "4.0"。
gyp ERR! build error
gyp ERR! stack Error: `C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe` failed with exit code: 1
gyp ERR! stack     at ChildProcess.onExit (C:\Program Files (x86)\nodejs\node_modules\npm\node_modules\node-gyp\lib\build.js:276:23)
gyp ERR! stack     at emitTwo (events.js:106:13)
gyp ERR! stack     at ChildProcess.emit (events.js:191:7)
gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:215:12)
gyp ERR! System Windows_NT 10.0.15063
gyp ERR! command "C:\\Program Files (x86)\\nodejs\\node.exe" "C:\\Program Files (x86)\\nodejs\\node_modules\\npm\\node_modules\\node-gyp\\bin\\node-gyp.js" "rebuild"
gyp ERR! cwd E:\work\dashboard\node_modules\libxmljs-mt
gyp ERR! node -v v6.10.3
gyp ERR! node-gyp -v v3.4.0
gyp ERR! not ok
```
这有可能是.net版本太高造成的，需要降低.net版本。

```text
打开【控制面板】——【程序】——【启用或关闭windows功能】——勾选低版本的.net 
```
(3) 如果报错“MSBUILD : error MSB3428: 未能加载 Visual C++ 组件“VCBuild.exe”，如下：

```text
MSBUILD : error MSB3428: 未能加载 Visual C++ 组件“VCBuild.exe”。要解决此问题，1) 安装 .NET Framework 2.0 SDK；2) 安装 Microsoft Visual Stu
dio 2005；或 3) 如果将该组件安装到了其他位置，请将其位置添加到系统路径中。 [E:\work\dashboard\node_modules\libxmljs-mt\build\binding.sln]
gyp ERR! build error
gyp ERR! stack Error: `C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe` failed with exit code: 1
gyp ERR! stack     at ChildProcess.onExit (C:\Program Files (x86)\nodejs\node_modules\npm\node_modules\node-gyp\lib\build.js:276:23)
gyp ERR! stack     at emitTwo (events.js:106:13)
gyp ERR! stack     at ChildProcess.emit (events.js:191:7)
gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:215:12)
gyp ERR! System Windows_NT 10.0.15063
gyp ERR! command "C:\\Program Files (x86)\\nodejs\\node.exe" "C:\\Program Files (x86)\\nodejs\\node_modules\\npm\\node_modules\\node-gyp\\bin\\node-gyp.js" "rebuild"
gyp ERR! cwd E:\work\dashboard\node_modules\libxmljs-mt
gyp ERR! node -v v6.10.3
gyp ERR! node-gyp -v v3.4.0
gyp ERR! not ok
```
这是开发环境的一些依赖没有安装，在windows下开发nodejs应用，需要做一些特别的准备，可以参考[windows 开发环境搭建指南](https://github.com/Microsoft/nodejs-guidelines/blob/master/windows-environment.md#compiling-native-addon-modules),或者，直接以管理员身份运行：
```text
npm install -g windows-build-tools
```
一般来说，这都是安装node-sass时候出现的错误(node-gyp需要这些环境)，你也可以到node-gyp项目上查看[node-gyp安装指南](https://github.com/nodejs/node-gyp)

（4）如果报“npm ERR! Windows_NT 10.0.14393 npm ERR! argv ”错误，如下：
```text
npm ERR! Windows_NT 10.0.14393 
npm ERR! argv "C:\\Program Files\\nodejs\\node.exe" "C:\\Users\\YK\\AppData\\Roaming\\npm\\node_modules\\npm\\bin\\npm-cli.js" "update" "-g" "ionic" 
npm ERR! node v6.10.2 
npm ERR! npm v3.10.10 
```
这有可能是版本node版本太高的问题，可以安装[nvm-windows](https://github.com/coreybutler/nvm-windows)来切换node版本。

> 注意：nvm-windows安装路径不能有空格

（5）
```text
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@^1.0.0 (node_modules\chokidar\node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.1: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})
npm WARN babel-loader@7.0.0 requires a peer of webpack@2 but none was installed.
npm WARN browserify-shim@2.0.10 requires a peer of browserify@>= 2.3.0 < 4 but none was installed.
npm ERR! Windows_NT 10.0.15063
npm ERR! argv "C:\\nodejs\\node.exe" "C:\\nodejs\\node_modules\\npm\\bin\\npm-cli.js" "install"
npm ERR! node v6.10.3
npm ERR! npm  v3.10.10
npm ERR! code ELIFECYCLE
npm ERR! kubernetes-dashboard@1.6.1 postinstall: `build/postinstall.sh`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the kubernetes-dashboard@1.6.1 postinstall script 'build/postinstall.sh'.
npm ERR! Make sure you have the latest version of node.js and npm installed.
npm ERR! If you do, this is most likely a problem with the kubernetes-dashboard package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     build/postinstall.sh
npm ERR! You can get information on how to open an issue for this project with:
npm ERR!     npm bugs kubernetes-dashboard
npm ERR! Or if that isn't available, you can get their info via:
npm ERR!     npm owner ls kubernetes-dashboard
npm ERR! There is likely additional logging output above.

npm ERR! Please include the following file with any support request:
npm ERR!     E:\work\dashboard\npm-debug.log
```
删除package.json的postinstall属性


# 项目修改记录
- conf文件，修改了heapsterServerHost
- 把项目中的localhost改成了主机ip，为了向外提供服务
- 删除了package.json中的postinstall
- 添加了一个serve:frontend的任务，可以独立出来，避免影响到升级