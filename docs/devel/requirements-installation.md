# Kubernetes Dashboard 依赖环境安装详细指南
 
 本文将详细阐述安装必备软件的过程。本教程需要一个Ubuntu环境（虚拟机也可以），但你不必精通于linux。如果你使用的是windows操作系统，那就先装个虚拟机吧，Virtual Box 或 VMWare都可以，推荐用Virtual Box.如果你不想使用虚拟机，也可以从[dashboard前后端分开部署教程](backend-frontend-separate.md)开始看起

 本文中的linux命令都跟在`$`符号后面，复制的时候只要复制`$`后面的字符即可。
 
## 更新和检查ubuntu

首先，更新Linux ：.

```shell
$ sudo apt-get update
$ sudo apt-get upgrade
```

#### 检查linux版本的方法

检查Linux发行版本:

```shell
$ uname -r
```
得到类似`3.2.0-23-generic`的结果 

也可以使用下面的命令：

```shell
$ lsb_release -a
```

得到类似下面的结果：

```log
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 12.04.5 LTS
Release:        12.04
Codename:       precise
```

这两个命令在你去github上提问的时候可能会用到。


## 安装实用工具

下面安装一些实用程序，这些程序在接下来的安装过程中将会用到：

```shell
$ sudo apt-get install curl
$ sudo apt-get install git
$ sudo apt-get install python
```

假如你用的是 fedora (install npm3)，你还需执行：

```shell
sudo dnf install -y fedora-repos-rawhide
sudo dnf install -y --enablerepo rawhide nodejs libuv --best --allowerasing
```

#### 检查一下安装是否成功

```shell
$ curl --version
$ git --version
$ python --version
```

These instructions were last tested with curl `7.22.0`, and git `1.7.9.5`.

## 安装 Vagrant

Vagrant是一个快速搭建编程环境的工具，有开发者形象称其为：[程序员的虚拟机](http://wp.fungo.me/linux/vagrant-for-programmer-ch1.html)。比如，你要在Java 1.6和Java 1.8编程环境中来回切换，你可能就需要用到它了

运行下面的命令安装Vagrant：

```shell
$ sudo apt-get install vagrant
$ vagrant --version
$ export KUBERNETES_PROVIDER=vagrant
$ echo "export KUBERNETES_PROVIDER=vagrant" >> ~/.profile
```

这里用的是Vagrant 1.0.1.

## 安装 Docker

安装docker过程基于: https://docs.docker.com/engine/installation/linux/ubuntulinux/。

> 随着docker版本升级，新的docker安装的过程或许不再是这样，这时你可以丢弃下面的安装过程，参照官方最新的教程来安装。总之我们的目的，就是在设备上安装一个docker

#### 预备工作

运行下面的命令：

```shell
$ sudo apt-get install apt-transport-https ca-certificates
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
$ sudo bash -c 'echo "deb https://apt.dockerproject.org/repo ubuntu-precise main" > /etc/apt/sources.list.d/docker.list'
$ sudo apt-get update
$ sudo apt-get purge lxc-docker
$ apt-cache policy docker-engine
```

对于 Ubuntu Precise 12.04 版本的用户，还需执行：

```shell
$ sudo apt-get update
$ sudo apt-get install linux-image-generic-lts-trusty
$ sudo reboot
```

#### 开始安装docker

```shell
$ sudo apt-get update
$ sudo apt-get install docker-engine
$ sudo service docker start
$ sudo docker run hello-world
```

运行结束后，将会显示一个包含 `This message shows that your installation appears to be working correctly`.的打印信息，说明你安装成功了。

#### 配置docker用户

下面的配置过程基于：:https://docs.docker.com/engine/installation/linux/ubuntulinux/#create-a-docker-group

注意：在下面的命令中，`username`只是一个占位符，需要替换成你当前登录ubuntu的用户名，可以用`id`命令来查看，比如，如果你是在虚拟机上使用Vagrant来搭建环境，你的用户名应该是vagrant

```shell
$ sudo groupadd docker
$ sudo usermod -aG docker username
$ sudo reboot
```

#### 检查一下docker是否安装成功

```shell
$ docker run hello-world
```

如果你还是看到包含`This message shows that your installation appears to be working correctly`的信息，那就是安装成功了

下面两个命令也可以用来检测docker的运行状态:

`$ status docker` --> 运行这个应该会返回  "docker start/running, process [some number]"

`$ docker ps` --> 运行这个会返回一个docker容器列表(如果没有容器，就只显示一个表头)


## 安装 Go

下面的命令会安装Go 1.8 for linux amd64 ，并把Go添加到PATH变量中，如果你要安装其它版本，可以到到[Go官方下载地址](https://golang.org/dl/)下载。

```shell
$ wget https://storage.googleapis.com/golang/go1.8.linux-amd64.tar.gz
$ sudo tar -C /usr/local -xzf go1.8.linux-amd64.tar.gz
$ export PATH=$PATH:/usr/local/go/bin
$ echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
```

#### 检查一下Go是否安装成功

```shell
$ go version
$ echo $PATH
```
这会打印出类似`go version go1.8 linux/amd64`的信息。另外，如果你之前早已安装过Go,确保你的环境变量里没有`GO15VENDOREXPERIMENT`，有的话就把它删去。

## 安装 Node 和 NPM

由于某些原因，执行`sudo apt-get install nodejs`会安装相对较低版本的Nodejs，所以我使用下面的命令安装较新版本：

```shell
$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
$ sudo apt-get install -y nodejs
```

#### 检查Node和NPM是否安装成功

```shell
$ node -v
$ npm -v
```

最好一次更新此文档时，上述命令返回`v6.3.1` 和 `3.10.3`，但是更高的版本也可能可以正常工作。（言下之意，为了确保能够完全正常工作，还是安装v6.3.1好）

## 安装 Java 运行时

```shell
$ sudo apt-get install openjdk-8-jre
```

假如你的linux是 fedora 系统：

```shell
$ sudo dnf install java-1.8.0-openjdk
```

#### 检查 java 运行时是否安装成功

```shell
$ java -version
```

这会返回类似`java version "1.7.0_101"`.

## 用 npm 全局安装 Gulp

```shell
$ npm install --global gulp-cli
$ npm install --global gulp
```
如果直接运行`npm install`提示`npm install access denied`，这应该是权限不够，官方早已有解决方案[Fixing npm permissions](https://docs.npmjs.com/getting-started/fixing-npm-permissions),建议按照官方的教程修复。如果实在不能修复，也可以使用sudo来执行：
```shell
$ sudo npm install --global gulp-cli
$ sudo npm install --global gulp
```

#### 检查 gulp是否安装成功

```shell
$ gulp -v
```

这会返回类似`CLI version 3.9.1` 和 `Local version 3.9.1`.


## 获取 kubernetes
下载kubernetes命令行工具kubectl:
```shell
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl   
$ chmod +x ./kubectl   
$ sudo mv ./kubectl /usr/local/bin/kubectl   
```

从Github中克隆下Dashboard和kubernetes的源码. *这需要一段时间，特别是没有翻-墙的话*

```shell
$ git clone https://github.com/kubernetes/dashboard.git
$ git clone https://github.com/kubernetes/kubernetes.git
```

## 安装 npm 依赖

```shell
$ cd ~/dashboard
$ npm install
```

这会读取项目根目录下的`package.json` 文件，安装其中所有的依赖，这需要花费一段时间，请耐心等待，在国内的话，如果安装不成功，则最好挂上代理翻墙。


## 启动 Kubernetes 集群
下面的命令下载最新的kubernetes，并在本地docker容器上运行起来：

```shell
$ cd ~/dashboard 
   
# Start cluster at first, because the dashboard need apiserver-host for requesting.
# 首先需要启动集群，因为dashboard需要从kubernetes api-server请求数据
$ gulp local-up-cluster   
  
# Run script to build docker image with name "kubernetes-dashboard-build-image ", and then start a container.
# The parameter "serve" is necessary, otherwise, there will be a gulp task error.
# 这个命令会将项目打包到"kubernetes-dashboard-build-image "镜像，然后以这个镜像启动一个容器
# 注意serve 参数是必须的
#
$ sudo build/run-gulp-in-docker.sh serve
```

如果你需要添加环境变量到容器中，则需要修改一下Dockerfile 文件

```Dockerfile
ENV http_proxy="http://username:passowrd@10.0.58.88:8080/"
ENV https_proxy="http://username:password@10.0.58.88:8080/"
```

要停止集群的话，可以运行 `$ docker kill $(docker ps -aq)`, 这样dashboard容器也会停止


#### 检查dashboard项目是否部署成功

打开另一个命令行，访问dashboard：

```shell
$ curl http://localhost:9090
```


#### 下一步

至此，所有环境依赖已经安装完成，下一步就是回到新手教程继续你未完成的工作[continue with the Getting Started guide](getting-started.md) to learn more about developing with the Kubernetes Dashboard.

# 问题排除

## Docker

If you're having trouble with the `gulp local-up-cluster` step, you may want to investigate the docker containers.

* `docker ps -a` lists all docker containers
* `docker inspect name_of_container | grep "Error"` will look through the details of a docker container and display any errors.

If you have an error like "linux mounts: Path /var/lib/kubelet is mounted on / but it is not a shared mount." you should try `sudo mount --bind /var/lib/kubelet /var/lib/kubelet` followed by `sudo mount --make-shared /var/lib/kubelet`. ([source](https://github.com/kubernetes/kubernetes/issues/4869#issuecomment-193640483))

## Go

假如报 "Go is not on the path."这样的错误 ，如下:
```text
$ sudo gulp serve
......

[23:48:52] 'backend' errored after 323 ms
[23:48:52] Error: Go is not on the path. Please pass the PATH variable when you run the gulp task with "PATH=$PATH" or install go if you have not yet.
    at /home/lcl/dashboard/build/gocommand.js:114:23
    at ChildProcess.exithandler (child_process.js:211:5)
    at emitTwo (events.js:106:13)
    at ChildProcess.emit (events.js:191:7)
    at maybeClose (internal/child_process.js:886:16)
    at Socket.<anonymous> (internal/child_process.js:342:11)
    at emitOne (events.js:96:13)
    at Socket.emit (events.js:188:7)
    at Pipe._handle.close [as _onclose] (net.js:497:12)

```
你可以添加`"PATH=$PATH"`参数来运行该命令：
```text
sudo "PATH=$PATH" gulp serve
```


## Helpful Linux Tips

* `env` will show your environment variables. One common error is not having every directory needed in your PATH.

Using *vim* to edit files may be helpful for beginners.

* `sudo apt-get install vim` will get *vim*
* `sudo vim /path/to/folder/filename` will open the file you want to edit.
* <kbd>i</kbd> = insert
* <kbd>Esc</kbd> = stops inserting
* `:x` = exits and saves
