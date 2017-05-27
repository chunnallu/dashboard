# 新手教程

本教程将告诉大家如何搭建起开发环境

## 项目架构

Kubernetes Dashboard 项目包含两个主要模块——前端（frontend）和后端（backend）。前端是一个运行在浏览器的单页web应用，它用标准HTTP的HTTP请求从后端获取所有业务数据。后端实现了界面业务逻辑，并且通过Kubernetes API来获取集群数据。

## 准备工作



要做如下的工作，建议跟着这篇教程来安装环境： [detailed steps on how to install these requirements](requirements-installation.md).

1、确保已经安装了下面的软件，并且这些软件的路径已经添加到 `$PATH` 环境变量中:

* Docker (1.10+)
* go (1.7+)
* nodejs (5.1.1+)
* npm (3+)
* java (7+)
* gulp (3.9+)
* python

2、接着把项目克隆到本地并且安装npm依赖

注意：如果你是以root权限来运行`npm install`的话，则要加上`--unsafe-perm` 参数

 ```shell
 # npm i --unsafe-perm
 ```

## 启动 Kubernetes 集群

For development it is recommended to run a local Kubernetes cluster. For your convenience, a
task is provided that checks out the latest stable version, and runs it inside a Docker container.
Run the following command:

```shell
$ gulp local-up-cluster
```

This will build and start a lightweight local cluster, consisting of a master and a single node.
All processes run locally, in Docker container. The local cluster should behave like a real
cluster, however, plugins like heapster are not installed. To shut it down, type the following
command that kills all running Docker containers:

```shell
$ docker kill $(docker ps -aq)
```

From time to time you might want to use to a real Kubernetes cluster (e.g. GCE, Vagrant) instead
of the local one. The most convenient way is to create a proxy. Run the following command instead
of the gulp task from above:

```shell
$ kubectl proxy --port=8080
```

kubectl will handle authentication with Kubernetes and create an API proxy with the address
`localhost:8080`. Therefore, no changes in the configuration are required.

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

It is easy to compile and run Dashboard. Open a new tab in your terminal and type:

```shell
$ gulp serve
```

Open a browser and access the UI under `localhost:9090`. A lot of things happened underneath.
Let's scratch on the surface a bit.

Compilation:
* Stylesheets are implemented with SASS and compiled to CSS with libsass
* JavaScript is implemented in ES6. It is compiled with Babel for development and the
  Google-Closure-Compiler for production.
* Go is used for the implementation of the backend. The source code gets compiled into the
  single binary 'dashboard'


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
