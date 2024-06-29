---
layout: post
title: Linux Docker cheatsheet
---

因为最近要在我的阿里云 ECS 上部署[软工作业项目](https://github.com/wi1sonh/cloudmall)，遂借此记录Docker在Linux上的配置过程，以 Ubuntu 22.04 LTS 为例说明

## 安装

安装添加 Docker 仓库所需的前置软件包：
`sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release`
(似乎除apt-transport-https外ubuntu都已自带了...)

使用以下命令下载并导入 Docker 官方的 GPG 密钥：
`curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -`

将 Docker 的官方仓库添加到 Ubuntu 22.04 LTS 的软件源列表：
`sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"`

刷新软件包列表，以便系统识别新添加的 Docker 仓库：
`sudo apt update`

执行以下命令来安装最新版本的 Docker，包括 Docker 引擎及其相关组件:
`sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

> 安装的组件包括：
> `docker-ce`：Docker Engine。
> `docker-ce-cli`：用于与 Docker 守护进程通信的命令行工具。
> `containerd.io`：管理容器生命周期的容器运行时环境。
> `docker-buildx-plugin`：增强镜像构建功能的 Docker 扩展工具，特别是在多平台构建方面。
> `docker-compose-plugin`：通过单个 YAML 文件管理多容器 Docker 应用的配置管理插件。

配置用户组（可选）：
> 默认情况下，只有 root 用户或具有 sudo 权限的用户才能够执行 Docker 命令。如果不加sudo前缀直接运行docker命令，系统会报权限错误。我们可以将当前用户添加到docker组，以避免每次使用Docker时都需要使用sudo。命令如下：
> `sudo usermod -aG docker ${USER}`（注：重新登录才能使更改生效）

我们可以通过启动docker来验证我们是否成功安装。命令如下：
`sudo systemctl start docker`

重启docker
`sudo service docker restart`

使用以下命令检查 Docker 的运行状态：
`sudo systemctl is-active docker`

替换 DockerHub 国内镜像源：

1. **打开配置文件**：使用文本编辑器打开 Docker 的配置文件，如果没有就新建：`sudo vim /etc/docker/daemon.json`
2. **编辑配置文件**：在配置文件中添加或修改registry-mirrors，指定国内镜像源的 URL。以下是一些可用的国内镜像源地址，可以根据需要选择使用：
  DaoCloud：https://docker.m.daocloud.io
  百度云镜像站：https://mirror.baidubce.com
  网易云镜像站: http://hub-mirror.c.163.com
  南京大学镜像站: https://docker.nju.edu.cn

```c
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://mirror.baidubce.com",
    "http://hub-mirror.c.163.com",
    "https://docker.nju.edu.cn"
  ]
}
```

3. **重启 Docker 服务**：保存配置文件并重启 Docker 服务以应用更改。

```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
```

4. **验证配置**：重启 Docker 后，使用docker info命令来检查镜像源是否已经替换成功：

```sh
sudo docker info
```

运行 hello-world 测试容器，验证 Docker 是否安装成功并正常工作：(因为我们之前没有拉取过hello-world，所以运行命令后会出现本地没有该镜像，并且会自动拉取的操作)
`sudo docker run hello-world`

## 杂项

我们可以通过下面的命令来查看docker的版本
`sudo docker version`

上面我们拉取了hello-world的镜像，现在我们可以通过命令来查看镜像，命令如下：
`sudo docker images`

查看镜像列表 `docker images`

删除镜像(xxx指镜像前三个id字母，需要先停止容器运行，并删除容器) `docker rmi xxx`(可加`-f`强制删除)

查看Docker运行中的容器 `docker ps`

查看所有容器(包括未运行的) `docker ps -a`

停止容器运行 `docker stop f66`

删除容器(可加`-f`强制删除) `docker rm container_id`

重启容器 `docker restart <容器ID>`

本地传文件到容器：`docker cp 本地文件路径 ID全称:容器路径`

容器传文件到本地：`docker cp ID全称:容器文件路径 本地路径`

要从 Ubuntu 22.04 LTS 中卸载 Docker，可以按照以下步骤操作：

1. 使用以下命令卸载 Docker 及其相关组件：

```sh
sudo apt purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
```

2. 执行以下命令来删除 Docker 创建的目录：

```sh
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

docker中 启动所有的容器命令
`docker start $(docker ps -a | awk '{ print $1}' | tail -n +2)`

docker中 关闭所有的容器命令
`docker stop $(docker ps -a | awk '{ print $1}' | tail -n +2)`

docker中 删除所有的容器命令
`docker rm $(docker ps -a | awk '{ print $1}' | tail -n +2)`

docker中 删除所有的镜像
`docker rmi $(docker images | awk '{print $3}' |tail -n +2)`


## reference

1. [Docker官方文档](https://docs.docker.com/)
2. [Docker社区论坛](https://forums.docker.com/)
3. [https://www.sysgeek.cn/install-docker-ubuntu/](https://www.sysgeek.cn/install-docker-ubuntu/)
4. [https://developer.aliyun.com/article/1323800](https://developer.aliyun.com/article/1323800)
