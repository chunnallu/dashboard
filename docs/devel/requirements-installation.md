# 安装 Kubernetes Dashboard 依赖环境

These instructions are an elaboration on how to install the requirements listed on the [Getting Started page](getting-started.md). This document assumes you have a Linux machine (or VM), and that you have a brand new Ubuntu Linux environment setup, but does not assume familiarity with Linux. If you don't have a Linux environment and you're using Windows, you may want to read instructions on how to setup a Linux VM on Windows first.

Before you begin please make sure you can connect to your Linux machine and login. Command line instructions for Linux will be shown starting with `$`; you should only type the text following the `$`.

## 系统设置

Based on instructions from: https://docs.docker.com/engine/installation/linux/ubuntulinux/

This will update and upgrade Linux.

```shell
$ sudo apt-get update
$ sudo apt-get upgrade
```

### Check

```shell
$ uname -r
```

You should get `3.2.0-23-generic` or something similar depending on what the current version is.


```shell
$ lsb_release -a
```

^ You should get a response like:

```log
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 12.04.5 LTS
Release:        12.04
Codename:       precise
```


## 安装实用工具

Install some programs that we'll need later on, and verify that they're there.

```shell
$ sudo apt-get install curl
$ sudo apt-get install git
$ sudo apt-get install python
```

if you are fedora (install npm3)

```shell
sudo dnf install -y fedora-repos-rawhide
sudo dnf install -y --enablerepo rawhide nodejs libuv --best --allowerasing
//TODO 自行安装python,依照经验在 npm install 的时候会用到
```

### 检查一下

```shell
$ curl --version
$ git --version
$ python --version
```

These instructions were last tested with curl `7.22.0`, and git `1.7.9.5`.

## 安装 Vagrant

```shell
$ sudo apt-get install vagrant
$ vagrant --version
$ export KUBERNETES_PROVIDER=vagrant
$ echo "export KUBERNETES_PROVIDER=vagrant" >> ~/.profile
```

These instructions are using vagrant version 1.0.1.

## 安装 Docker

Based on instructions from: https://docs.docker.com/engine/installation/linux/ubuntulinux/

### Setup

```shell
$ sudo apt-get install apt-transport-https ca-certificates
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

Create a docker.list file with one command:

```shell
$ sudo bash -c 'echo "deb https://apt.dockerproject.org/repo ubuntu-precise main" > /etc/apt/sources.list.d/docker.list'
```

```shell
$ sudo apt-get update
$ sudo apt-get purge lxc-docker
$ apt-cache policy docker-engine
```

### Only needed for Ubuntu Precise 12.04

```shell
$ sudo apt-get update
$ sudo apt-get install linux-image-generic-lts-trusty
$ sudo reboot
```

### Do the Docker install

```shell
$ sudo apt-get update
$ sudo apt-get install docker-engine
$ sudo service docker start
$ sudo docker run hello-world
```

You should receive a message that includes: `This message shows that your installation appears to be working correctly`.

### Configure Docker for your user

Based on instructions from https://docs.docker.com/engine/installation/linux/ubuntulinux/#create-a-docker-group

The example below uses "username" as a placeholder. Please substitute with the user you are logged in as, which can be seen by using `$ id`.
If you are running Linux in a VM using Vagrant, your username will be "vagrant".

```shell
$ sudo groupadd docker
$ sudo usermod -aG docker username
$ sudo reboot
```

#### 检查一下

```shell
$ docker run hello-world
```

You should get the same message as above, that includes: `This message shows that your installation appears to be working correctly`.

For an additional check you can run these commands:

`$ status docker` --> should say  "docker start/running, process [some number]"

`$ docker ps` --> should show a table of information (or at least headers)


## 安装 Go

The instructions below are for install a specific version of Go (1.8 for linux amd64). If you want the latest Go version or have a different system, then you can get the latest download URL from https://golang.org/dl/

```shell
$ wget https://storage.googleapis.com/golang/go1.8.linux-amd64.tar.gz
$ sudo tar -C /usr/local -xzf go1.8.linux-amd64.tar.gz
$ export PATH=$PATH:/usr/local/go/bin
$ echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
```

### 检查一下

```shell
$ go version
$ echo $PATH
```

The Go version should return something like `go version go1.8 linux/amd64`. Note that if you already had Go installed, ensure that `GO15VENDOREXPERIMENT` is unset.

## 安装 Node 和 NPM

For some reason doing `sudo apt-get install nodejs` gives a much older version, so instead we will get the more recent version:

```shell
$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
$ sudo apt-get install -y nodejs
```

### Check

```shell
$ node -v
$ npm -v
```

The last time these instructions were updated, this returned `v6.3.1` and `3.10.3` respectively, but later versions will probably also work.

## 安装 Java 运行时

```shell
$ sudo apt-get install openjdk-8-jre
```

if you are fedora

```shell
$ sudo dnf install java-1.8.0-openjdk
```

### 检查一下

```shell
$ java -version
```

Should return `java version "1.7.0_101"`.

## 用 npm 全局安装 Gulp

```shell
$ sudo npm install --global gulp-cli
$ sudo npm install --global gulp
```

### 检查一下

```shell
$ gulp -v
```

Should return `CLI version 3.9.1` and `Local version 3.9.1`.


## 获取 kubernetes

Download the command line tool _kubectl_.

```shell
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl   
$ chmod +x ./kubectl   
$ sudo mv ./kubectl /usr/local/bin/kubectl   
```

Clone the Dashboard and Kubernetes code from the GitHub repos. *This could take a while.*

```shell
$ git clone https://github.com/kubernetes/dashboard.git
$ git clone https://github.com/kubernetes/kubernetes.git
```

## 安装 npm 依赖

```shell
$ cd ~/dashboard
$ npm install
```

这会读取项目根目录下的`package.json` 文件，安装其中所有的依赖，这需要花费一段时间，请耐心等待。


## 启动 Kubernetes 集群

Run the script included with the dashboard that checks out the latest Kubernetes and runs it in a Docker container.

```shell
$ cd ~/dashboard 
   
# Start cluster at first, because the dashboard need apiserver-host for requesting.
$ gulp local-up-cluster   
  
# Run script to build docker image with name "kubernetes-dashboard-build-image ", and then start a container.
# The parameter "serve" is necessary, otherwise, there will be a gulp task error.
$ sudo build/run-gulp-in-docker.sh serve
```
If you need append ENV variables to container, you should edit the Dockerfile.

```Dockerfile
ENV http_proxy="http://username:passowrd@10.0.58.88:8080/"
ENV https_proxy="http://username:password@10.0.58.88:8080/"
```

If you need to stop the cluster you can run `$ docker kill $(docker ps -aq)`, 
and the dashboard container is stopped also.


### 检查一下

Open up another terminal to your machine, and try to access the dashboard.

```shell
$ curl http://localhost:9090
```

This should return the HTML for the dashboard.

### Continue

Now you may [continue with the Getting Started guide](getting-started.md) to learn more about developing with the Kubernetes Dashboard.

# Troubleshooting

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
