# 新手教程

本教程将告诉大家如何搭建起开发环境。本教程将会引导大家在ubuntu单机上运行起开发环境，对于很多机器性能不好的开发者，或许希望可以将后端或kubernetes集群放到服务器，将前端放到本机上开发，这也是可以的，可以看[dashboard前后端分开部署教程](backend-frontend-separate.md)

## 项目架构

Kubernetes Dashboard 项目包含两个主要模块——前端（frontend）和后端（backend）。

前端是一个运行在浏览器的单页WEB应用，主要使用了angular框架，它用标准的HTTP请求从后端获取所有业务数据。
后端接受前端的请求，从Kubernetes API Server中获取集群的数据。后端是使用go语言编写的。

默认情况下，所有这些部件（前端、后端、kubernetes集群）都是安装在同一台linux上的，当然，也都是有办法把各个部件分开的。

建议先不要急于开始，把本文简略读一遍再行动也不迟。

## 准备工作

建议跟着这篇教程来安装环境： [Kubernetes Dashboard 依赖环境安装详细指南](requirements-installation.md).

安装完后检查：

1、确保已经安装了下面的软件，并且这些软件的路径已经添加到 `$PATH` 环境变量中:

* Docker (1.10+)
* go (1.7+)
* nodejs (5.1.1+)
* npm (3+)
* java (7+)
* gulp (3.9+)
* python

2、如果你是以root权限来运行`npm install`

如果你是以root权限来运行`npm install`的话，则确保一定加了`--unsafe-perm` 参数

 ```shell
 # npm i --unsafe-perm
 ```
 否则的话，就算`npm install`没报错，其实也没安装成功。

## 启动 Kubernetes 集群

开发时，建议使用本地kubernetes集群，简单、方便、要求低。Dashboard本身提供了一个一键启动本地kubernetes集群的gulp命令：

```shell
$ gulp local-up-cluster
```
仅仅这一句就可以了，这会启动一个轻量的本地集群（类似minikuber），包含一个master节点和一个node节点。这个集群和真实的集群别无二致，只不过很多插件都没装，像heapster插件。另外，这个集群的API Server只能本地访问。

如果要关闭这个集群，可以使用下面的命令杀死docker容器中所有的进程：
```shell
$ docker kill $(docker ps -aq)
```

随着时间推移，你可能希望使用真的Kubernetes集群来代替这个本地集群，最方便的办法是使用代理，用下面的命令替代上面的gulp命令：

```shell
$ kubectl proxy --port=8080
```

kubectl 会处理与Kubernetes集群的认证并且创建一个API代理，代理地址为`localhost:8080`，因此，其它什么都不用改，就可以切换到真正的kubernete集群上了。

除此之外，还有另外一种方法：
Another way to connect to real cluster while developing dashboard is to override default values used
by our build pipeline. In order to do that we have introduced two environment variables
`KUBE_DASHBOARD_APISERVER_HOST` and `KUBE_DASHBOARD_KUBECONFIG` that will be used over default ones when
defined. Before running our gulp tasks just do:

```shell
$ export KUBE_DASHBOARD_APISERVER_HOST="http://<APISERVER_IP>:<APISERVER_PORT>"
# or
$ export KUBE_DASHBOARD_KUBECONFIG="<KUBECONFIG_FILE_PATH>"
```

**NOTE: Environment variable `KUBE_DASHBOARD_KUBECONFIG` has higher priority than `KUBE_DASHBOARD_APISERVER_HOST`.**

## 以开发模式启动dashboard

要编译和启动dashboard非常容易，只需要运行：

```shell
$ gulp serve
```
然后打开浏览器，输入`localhost:9090`就可以看到了。

这句会运行一个gulp定义的任务，任务的定义放在`build/serve.js`文件中。任务运行起来很简单，但其实做了很多事，包括：

**编译**:
* 通过libsass库将用SASS语法写的样式表会编译成CSS代码
* 用ES6写的JavaScript代码，在开发环境中将会使用Babel编译器编译成ES5,而在生产环境中，则使用Google-Closure-Compiler来编译。
* Go语言编写的后端，会被编译成一个二进制包，名为'dashboard'

**执行**:
* 通过BrowserSync启动前端，BrowserSync是一个浏览器同步测试框架，它可以在多个设备、多个浏览器之间同步热更新代码、同步滚动、点击和表单输入等。多说无益，前往[browserSync中文网](http://www.browsersync.cn/)看看就能有个深刻的印象了！
* 运行前面生成的`dashboard`二进制包来启动后端

此外，所有相关文件将会被打包到`.tmp`文件夹下，gulp会监控源码(CSS, JS, GO)的更新，并自动重编译，所以，对源码的更改都会立即生效。当然，有时候也会出现修改了文件，却迟迟没见重新编译，这时候可能要重新运行一次`gulp serve`。

成功执行`gulp local-up-cluster` 和 `gulp serve`之后，下列程序就会运行起来（括号中是默认使用的端口）：

BrowserSync (9090)  ---> Dashboard backend (9091)  ---> Kubernetes API server (8080)


## 构建Dashboard(生产环境)

**打包成产品**

运行如下命令可以将dashboard打包成一个产品:

```shell
$ gulp build
```
这将会编译、压缩代码，删除测试信息，并将处理后的代码放到`dist`文件夹下。

你可以使用如下命令从`dist`文件夹启动dashboard:
```shell
$ gulp serve:prod
```
这将会启动如下进程：

Dashboard backend (9090)  ---> Kubernetes API server (8080)

然后打开浏览器访问`localhost:9090.`就可以看到dashboard的UI界面了。

**打包成docker镜像**

要打包成docker镜像可以使用如下命令:

```shell
$ gulp docker-image:head
```

> You might notice that the Docker image is very small and requires only a few MB. Only
Dashboard assets are added to a scratch image. This is possible, because the `dashboard`
binary has no external dependencies. Awesome!

## 运行测试

每次修改完源码后都应运行单元测试，下面这个命令可以检测代码的修改自动运行单元测试：

```shell
$ gulp test:watch
```

而完整的测试，包括静态代码检查、单元测试和集成测试则可以运行下面的命令来执行：

```shell
$ gulp check
```

此外，各种测试也可以分开执行。比如，要进行集成测试和JavaScript代码检查（它能帮你找出一些低级错误），你可以单独运行下面两个命令：

```shell
$ gulp integration-test:prod
$ gulp check-javascript-format
```

## 提交代码

在提交代码前，请链接或拷贝（link/copy）pre-commit 钩子到你的`.git`目录，这能避免你意外提交未格式化的代码。

这个钩子要求 gofmt 在 PATH 路径中：

```shell
cd <dashboard_home>/.git/hooks/
ln -s ../../hooks/pre-commit .
```

然后你便可以提交代码到代码仓库中了：

```shell
git commit
git push -f origin my-feature
```

## 在容器中开发Dashboard

在容器中进行开发也是可以的，你完全可以在容器运行`gulp`任务和安装所有的依赖。要做到这点，首先要安装好`docker`,然后将之前的`gulp [some arg]`命令相应的替换成`build/run-gulp-in-docker.sh [some arg]`即可。至于像`npm install`这种搭建开发环境的命令，将会被自动运行。

> 容器本身的目标，就是要提供一个标准的运行环境，所以，使用容器来运行，或许能省去很多环境相关的问题。缺点便是，调试起来相对麻烦。
