# 新手教程

本教程将告诉大家如何搭建起开发环境

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

但其实这句简单的命令做了很多事，包括：

编译:
* 用SASS语法写的样式表会编译成CSS代码，通过libsass库
* 用ES6写的JavaScript代码，在开发环境中将会使用Babel编译器编译成ES5,而在产品环境中，则使用Google-Closure-Compiler来编译。
* Go语言编写的后端，会被编译成一个二进制包，名为'dashboard'


Execution:
* Frontend is served by BrowserSync. It enables features like live reloading when
  HTML/CSS/JS change and even synchronize scrolls, clicks and form inputs across multiple devices.
* Backend is served by the 'dashboard' binary.

File watchers listen for source code changes (CSS, JS, GO) and automatically recompile.
All changes are instantly reflected, e.g. by automatic process restarts or browser refreshes.
The build artifacts are created in a hidden folder (`.tmp`).

After successful execution of `gulp local-up-cluster` and `gulp serve`, the following processes
should be running (respective ports are given in parentheses):

BrowserSync (9090)  ---> Dashboard backend (9091)  ---> Kubernetes API server (8080)


## 构建Dashboard(生产环境)

The Dashboard project can be built for production by using the following task:

```shell
$ gulp build
```

The code is compiled, compressed and debug support removed. The artifacts can be found
in the `dist` folder.

In order to serve Dashboard from the `dist` folder, use the following task:

```shell
$ gulp serve:prod
```

Open a browser and access the UI under `localhost:9090.` The following processes should
be running (respective ports are given in parentheses):


Dashboard backend (9090)  ---> Kubernetes API server (8080)



In order to package everything into a ready-to-run Docker image, use the following task:

```shell
$ gulp docker-image:head
```

You might notice that the Docker image is very small and requires only a few MB. Only
Dashboard assets are added to a scratch image. This is possible, because the `dashboard`
binary has no external dependencies. Awesome!

## 运行测试

Unit tests should be executed after every source code change. The following task makes this
a breeze by automatically executing the unit tests after every save action.

```shell
$ gulp test:watch
```

The full test suite includes static code analysis, unit tests and integration tests.
It can be executed with:

```shell
$ gulp check
```

By the way, the test can run alone, such as integration and javascript-format test (it can help you focus primary problem).

```shell
$ gulp integration-test:prod
$ gulp check-javascript-format
```

## Committing changes to your fork

Before committing any changes, please link/copy the pre-commit hook into your .git directory. This will keep you from accidentally committing non formatted code.

The hook requires gofmt to be in your PATH.

```shell
cd <dashboard_home>/.git/hooks/
ln -s ../../hooks/pre-commit .
```

Then you can commit your changes and push them to your fork:

```shell
git commit
git push -f origin my-feature
```

## 在容器中构建Dashboard

It's possible to run `gulp` and all the dependencies inside a development container. To do this,
just replace `gulp [some arg]` commands with `build/run-gulp-in-docker.sh [some arg]`. If you
do this, the only dependency is `docker`, and required commands such as `npm install`
will be run automatically.

## Contribute

Wish to contribute? Great, start [here](../../.github/CONTRIBUTING.md).
