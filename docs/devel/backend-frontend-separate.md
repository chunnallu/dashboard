
# dashboard前后端分开部署教程

# 后端搭建

## 搭建后端环境

#### 安装项目环境

第一次运行，请先[安装项目环境](requirements-installation.md)

#### 启动docker服务
```text
sudo service docker start
```

#### 启动kubernetes集群

```text
sudo gulp local-up-cluster
```
默认情况下，这将启动一个只能本地访问的kubernetes集群，你可以使用：
```
curl 127.0.0.1:8080
```
访问Kubernetss API Server。

如果你想要能够远程访问Kubernetss API Server，你可以使用：
```
curl https://raw.githubusercontent.com/fest-research/iot-addon/master/assets/hyperkube/hyperkube.sh | sudo sh
```
代替`sudo gulp local-up-cluster`,参考[the api server can't access remotely](https://github.com/kubernetes/dashboard/issues/2012)

####  允许所有的HTTP连接

默认情况下，后端是只允许本地http连接的，我们需要修改成允许所有的http连接

```text
cd ~/dashboard
vi src/app/backend/dashboard.go
```
把：
```text
 argInsecureBindAddress = pflag.IP("insecure-bind-address", net.IPv4(127, 0, 0, 1), "The IP address on which to serve the --port (set to 0.0.0.0 for all interfaces).")
```
改成：
```text
argInsecureBindAddress = pflag.IP("insecure-bind-address", net.IPv4(0, 0, 0, 0), "The IP address on which to serve the --port (set to 0.0.0.0 for all interfaces).")
```


####  启动服务器
```text
sudo "PATH=$PATH" gulp serve
```
## 搭建后端环境可能会遇到的问题

#### 运行`gulp local-up-cluster`一直`waitting for a heapster...`

打开build/conf.js文件，将：
```text
 heapsterServerHost: gulpUtil.env.heapsterServerHost !== undefined ? gulpUtil.env.heapsterServerHost : '',
```
改成:
```text
 heapsterServerHost: gulpUtil.env.heapsterServerHost !== undefined ? gulpUtil.env.heapsterServerHost : '127.0.0.1:8082',
```

####  找不到go路径
报错信息如下：
``` text
Error: Go is not on the path. Please pass the PATH variable when you run the gulp task with "PATH=$PATH" or install go if you have not yet.
```
首先，要确定你已经安装了go,运行:
```text
go version
```
如果有返回go的版本，则安装成功了，否则，先[安装go](requirements-installation.md#install-go).

如果go确实已经安装，则可以在运行命令时传入PATH变量：
```text
sudo "PATH=$PATH" <COMMAND>
```


# 前端搭建

前端运行在windows上，通过api调用后端的服务。所以在windows上也只需要安装前端的依赖。

## 搭建前端环境

#### 安装实用工具

1、安装[git](https://git-scm.com/downloads)

2、安装[curl](https://curl.haxx.se/)

这两个工具自行安装，比较简单。安装完之后：

1、点开右键，如果有“git bash”和“git gui”两个选项，git就安装成功了

2、打开命令行，运行：
```text
curl www.baidu.com
```
如果有返回就是成功了

#### 安装node
  
  下载[nodejs安装包](https://nodejs.org/zh-cn/download/),并安装
  
  > 提示：如果你需要在多个node版本之间切换，可以使用[nvm windows](https://github.com/coreybutler/nvm-windows)工具,切换了node版本之后要运行一次`node rebuild`
  
#### 安装windows开发环境

  以管理员身份运行：
  ```text
   npm install -g windows-build-tools
  ```
  这是微软提供的一键安装windows开发环境的工具，请确保这行命令运行成功，详情参考[Configuring your Windows development environment](https://github.com/Microsoft/nodejs-guidelines/blob/master/windows-environment.md)

#### 安装java 及 xtbgenerator
1、国际化需要用到java环境，请自行安装java
2、在项目根目录下，右键打开git bash，运行：
```text
mkdir ./.tools/
cd ./.tools/; git clone https://github.com/kuzmisin/xtbgenerator; cd xtbgenerator; git checkout d6a6c9ed0833f461508351a80bc36854bc5509b2
```

#### 修改国际化脚本在windows下的bug

1、打开`build/i18n.js`,

找到：
```js
let messageVarPrefix = filePath.toUpperCase().split('/').join('_').replace('.HTML', '');
```
替换成：

```js
let messageVarPrefix = filePath.toUpperCase().replace(/\\/g,"\/").split('/').join('_').replace('.HTML', '');
```
这是因为在windows下路径并不是以`/`来分割的

#### 删除package.json中的postinstall脚本
 postinstall.sh是`npm install` 的一个钩子脚本，它在`npm install`命令运行完之后执行，进行bower依赖的安装和go路径的设置，这里我们将手动执行。
 
 打开package.json，将：
 ```text
"postinstall": "build/postinstall.sh"
```
删去

#### 安装前端依赖
在项目路劲下，运行：
```text
npm  install
npm install -g bower
bower install
```
#### 修改browserSync代理配置
打开build/serve.js文件，把：
```text
let proxyMiddlewareOptions =
        url.parse(`http://localhost:${conf.backend.devServerPort}${apiRoute}`);
```
改成：
```text
let proxyMiddlewareOptions =
        url.parse(`http://<YOU BACKEND SERVER IP>:${conf.backend.devServerPort}${apiRoute}`);
```
记得把`<YOU BACKEND SERVER IP>`替换成你后端服务器的ip

#### 删除后端监控脚本
打开build/serve.js文件，把：
```text
gulp.watch(path.join(conf.paths.backendSrc, '**/*.go'), ['spawn-backend']);
```
这句删掉

#### 添加前端监控脚本

打开build/serve.js文件，在末尾添加：
```js
/**
 * 只编译和监控前端代码
 */
gulp.task('serve:frontend', ['watch'], serveDevelopmentMode);
```

#### 运行起前端监控和开发服务器
```text
gulp serve:forntend
```

## 搭建前端环境可能遇到的问题

#### npm 错误

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
这有可能是版本node版本的问题，可以安装[nvm-windows](https://github.com/coreybutler/nvm-windows)来切换node版本。

> 注意：nvm-windows安装路径不能有空格

（5）如果报`Failed at the kubernetes-dashboard@1.6.1 postinstall script 'build/postinstall.sh`错误，是因为postinstall.sh不能再windows下执行的问题
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

#### gulp错误
（1）运行带`:prod`标签的任务，如`gulp serve:prod`报：
  ```
Error: Input is not proper UTF-8, indicate encoding !
Bytes: 0xBC 0xAF 0xC8 0xBA
```
最新版本已经解决这这个问题，如果你用的是较旧版本（2017.7之前）,你可以打开`build/i18n.js`，将：
```
fileExists(translationBundle)

```
改成：
```text
fileExists.sync(translationBundle)
```
既可以