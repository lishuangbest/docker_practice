# Docker 学习笔记
## 命令
### 获取镜像

pull 命令

- docker [image] pull NAME[:TAG]：其中 NAME 是镜像仓库名称，TAG 是镜像标签，描述一个镜像需要包括「名称 + 标签」信息。若不显式指定 TAG，则默认会选择 latest 标签，这回下载仓库中最新版本的镜像。

严格的讲，镜像的仓库名称中还应该添加仓库地址（即 registry，注册服务器）作为前缀，只是默认使用的是官方 Docker Hub 服务，该前缀可以忽略。
比如
```
docker pull ubuntu:18.04
相当于
docker pull registry.hub.docker.com/ubuntu:18.04
```

如果从非官方的仓库下载，需要在仓库名称前指定完整的仓库地址，比如
```
docker pull hub.c.163.com/public/ubuntu:18.04
```

子命令支持的选项：
- -a, --all-tags=true|false：是否获取仓库中的所有镜像，默认为否；
- --disable-content-trust：取消镜像的内容校验，默认为真。

指定镜像代理服务加速镜像获取，在启动配置中加入 `--registry-mirror=proxy_URL`。

使用 ubuntu 的 bash 应用打印「Hello World」：
```
docker run ubuntu:18.04 bash
echo "Hello World"
exit
```

## 查看镜像信息

1. 使用 images 命令列出镜像

```
docker images
或
docker image ls
```

镜像 id 相同说明两个 id 指向相同的镜像，只是具有不同的标签。

images 子命令主要支持以下选项：
- -a, --all=true | false: 列出所有（包括临时文件）镜像文件，默认为否；
- --digests=true | false: 列出镜像的数字摘要值，默认为否；
- -f, --filter=[]：过滤列出的镜像，如 dangling=true 只显示没有被使用的镜像；也可指定带有特定标注的镜像等；
- --format="TEMPLATE"：控制输出格式，如 .ID 代表 ID 信息，.Repository 代表仓库信息等；
- --no-trunc=true|false：对输出结果中太长的部分是否进行截断，如镜像的 ID 信息，默认为是；
- -q， --quiet=true|false：仅输出 ID 信息，默认为否。

更多子命令选项可通过 man docker-images 来查看。

2. 使用 tag 命令添加镜像标签，例如
```
docker tag ubuntu:12.04 myubuntu:12
```
实际上它们指向的是同一个镜像文件，只是别名不同，docker tag 命令添加的标签实际上起到了类似链接的作用。

3. 使用 inspect 命令查看详细信息
命令格式
```
docker [image] inspect NAME:TAG
```
比如输入
```
docker inspect myubuntu:12
```
会返回一个JSON格式的消息，如果我们只要其中一项内容时，可以使用 -f 来指定，例如，获取镜像的 RootFS
```
docker inspect -f {{".RootFS"}} myubuntu:12
```
4. 使用 history 命令查看镜像历史

格式

```
docker history ubuntu:12.04
```

过长的命令被自动截断，可以使用 --no-trunc 选项来输出完整命令。

## 搜寻镜像

主要使用 search 子命令。语法
```
docker search [option] keyword
```
支持的命令选项包括
- -f，--filter filter：过滤输出内容
- --format string：格式化输出内容
- --limit int：限制输出结果个数，默认为25个
- --no-trunc：不截断输出结果
例如，搜索官方提供的待 nginx 关键字的镜像，如下
```
docker search --filter=is-official=true nginx
```
搜索所有收藏数超过 4 的关键字包括 tensorflow 的镜像
```
docker search --filter=stars=4 tensorflow
```
结果包含镜像名字，描述，收藏数，是否官方创建，是否自动创建等，默认输出结构按照收藏数排序

## 删除和清理镜像
rm 和 prune 子命令
1. 使用标签删除镜像
使用 docker rmi 或 dockers image rm 命令删除，命令格式
```
docker rmi IMAGE [IMAGE...]
```
其中IMAGE可以为标签或ID。
支持的选项包括：
- -f，-force：强制删除镜像，即使又容器依赖它；
- -no-prune：不要清理未带标签的父镜像。
删除 myubuntu:12 镜像
```
docker rmi myubuntu:12
```
当同一镜像拥有多个标签的时候，dockers rmi 命令只是删除了该镜像多个标签中的指定标签而已，并不影响镜像文件。
2. 使用镜像id删除镜像
当 docker rmi 命令后跟镜像 ID 时，会先尝试删除所有指向该镜像的标签，然后删除该镜像文件本身。需要注意的是，当有镜像创建的容器存在时，镜像文件默认无法被删除。
先使用 docker ps -a 命令查看本机上存在的所有容器
```
docker ps -a
```
我们存在有 ubuntu:12.04 创建的退出状态的容器。这时我们尝试删除该镜像
```
docker rmi ubuntu:12.04
```
会提示有容器存在，无法删除，若想强制删除，可以通过添加 -f 参数
```
docker rmi -f ubuntu:12.04
```
并不推荐使用 -f 参数来删除一个存在容器依赖的镜像。正确的步骤是先删除依赖该镜像的所有容器，再来删除镜像
```
docker rm 54361e
```
然后通过 ID 删除镜像，此时会正常打印出删除的各层信息
```
docker rmi 5b117edd
```
3. 清理镜像
使用 Docker 一段时间后，系统中可能会遗留一些临时的镜像文件，以及一些没有被使用的镜像，可以通过 docker image prune 命令来进行清理。

支持的选项包括：
- -a，-all：删除所有无用镜像，不光是临时镜像；
- -filter filter：只清理符合给定过滤器的镜像；
- -f，-force：强制删除镜像，而不进行提示确认。

使用实例
```
docker image prune -f
```

## 创建镜像
创建镜像的方法主要有三种，基于已有镜像的容器创建、基于本地模板导入、基于 Dockerfile 创建。
1. 基于已有容器创建
使用 docker [container] commit 命令
```
docker [container] commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```
主要选项包括：
- -a，--author=""：作者信息；
- -c，--change=[]：提交的时候执行 Dockerfile 指令，包括 CMD|ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|VOLUME|WORKDIR 等；
- -m，--message=""：提交消息；
- -p，--pause=true：提交时暂停容器运行。

下面演示如何使用该命令创建一个新镜像。

首先，启动一个镜像，并在其中进行修改操作。例如，创建一个 test 文件，之后退出，代码如下：
```
docker run -it ubuntu:latest /bin/bash
touch test
exit
```
记住容器的 ID，比如我刚刚创建的容器的 ID 为：45a2238d01d0，可以通过 commit 命令提交为一个新的镜像。提交时可以使用 ID 或名称来指定容器：
```
docker [container] commit -m "Added a new file" -a "li shuang" 45a2238d01d0 test:0.1
```
如果顺利的话会返回新镜像的 ID，通过 docker images 就可以看到新镜像了。
2. 基于本地模板导入
用户也可以直接从一个操作系统模板文件导入一个镜像，主要使用docker [container] import 命令。命令格式

```
docker [image] import [OPTIONS] file|URL|-[REPOSITORY[:TAG]]
```
要直接导入一个镜像，可以使用 OpenVZ 提供的模板来创建，或者用其他已到处的镜像模板来创建。例如，下载好 ubuntu-18.04 的模板压缩包，之后使用以下命令导入即可

```
cat ubuntu-18.04-x86_64-minimal.tar.gz | docker import - ubuntu:18.04
```
然后查看新导入的镜像
```
docker iamges
```
3. 基于 Dockerfile 创建
这是最常见的方式。Dockile 是一个文本文件，利用给定的指令描述基于某个父镜像创建新镜像的过程。下面给出 Dockerfile 的一个简单示例，基于 debian:stretch-slim 镜像安装 Python 3 环境，构成一个新的 python:3 镜像：
```
FROM debian:stretch-slim

LABEL version="1.0" maintainer="docker user <docker_user@github>"

RUN apt-get update && \
    apt-get install -y python3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```
创建镜像的过程可以使用 docker [image] build 命令，编译成功后本地将多出一个 python: 3 镜像
```
docker [image] build -t python:3 .
```

## 存出和载入镜像
Docker 镜像的 save 和 load 子命令。用户可以使用 docker [iamge] save 和 docker [image] load 命令来存出和载入镜像。

1. 存出镜像
docker [image] save 命令，支持参数 -o、-output string 参数，导出镜像到指定的文件中。

例如，导出本地的 ubuntu:latest 镜像文件为 ubuntu_latest.tar，如下
```
docker image save -o ubuntu_latest.tar ubuntu:latest
```
然后，我们就可以将 ubuntu_latest.tar 文件分享给其他人，其他人也可以使用我们保存的镜像。

2. 载入镜像
docker [iamge] load 将导出的 tar 文件再导入本地镜像库，支持 -i、-input string选项，从指定文件中读入镜像内容。
```
docker load -i ubuntu_latest.tar
或
docker load < ubuntu_latest.tar
```
## 上传镜像

push 子命令。docker [iamge] push 命令上传镜像到仓库，默认上传到 Docker Hub 官方仓库（需要登录）。命令格式为 
```
docker [image] push NAME[:TAG] | [REGISTRY_HOST[:REGISTRY_PORT]/]NAME[:TAG]
```
例如，用户user上传本地的 test:latest 镜像，可以先添加新的标签 user/test:latest，然后用 docker [image] push 命令上传镜像：
```
docker tag test:latest user/test:latest
docker push user/test:latest
```

# 操作 Docker 容器

## 创建容器

1. 新建容器
可以使用 docker [container] create 命令新建一个容器，例如：
```
docker create -it ubuntu:latest
```
使用上条指令创建新建的容器处于停止状态，可以使用 docker [container] start 命令启动。
选项包含几大类：与容器运行模式相关、与容器环境配置相关、与容器资源限制和安全保护相关。太多，不写，见《Docker 技术入门与实战 第三版》40页。

2. 启动容器
docker [container] start 命令启动。
```
docker start d41ec
```

3. 新建并启动容器
命令 docker [container] run 等价于先执行 docker [container] create 命令，再执行 docker [container] start 命令。
例如下面命令输出一个「Hello World」，之后容器自动终止
```
docker run ubuntu /bin/echo 'Hello World'
```
这跟在本地执行/bin/echo 「Hello World」几乎感觉不到区别。

当使用 run 命令，Docker 在后台运行的标准操作包括：
- 检查本地是否存在指定的镜像，不存在就从公有仓库下载；
- 利用镜像创建一个容器，并启动该容器；
- 分配一个文件系统给容器，并在只读的镜像层外面挂载一层可读写层；
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去；
- 从网桥的地址池配置一个 IP 地址给容器；
- 执行用户指定的应用程序；
- 执行完毕后容器被自动终止。

下面的命令，启动一个 bash 终端，允许用户进行交互：
```
docker run -it ubuntu:latest /bin/bash
```
其中，-t 选项让 Docker 分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上，-i 则让容器的标准输入保持打开。更多的命令选项可以通过 man docker-run 命令查看。

在交互模式下，用户可以通过所创建的终端来输入命令，例如
```
pwd
ls
ps
```
通过 ps 命令查看进程，可以看到，只运行了 bash 应用，并没有运行其他无关的进程。

用户可以按 Ctrl+d 或输入 exit 命令退出容器，可以使用 docker container wait CONTAINER [CONTAINER...] 子命令来等待容器退出，并打印退出的返回结果。

docker [container] run 命令常见错误码
- 125：Docker daemon 执行出错，例如指定了不支持的 Docker 命令参数；
- 126：所指定命令无法执行，例如权限出错；
- 127：容器内命令无法找到。
命令执行后出错，会默认返回命令的退出错误码。

4. 守护态运行
更多时候，需要让 Docker 容器在后台以守护态（Daemonized）形式运行。此时，可以通过添加 -d 参数来实现。
例如，下面的命令会在后台运行容器：
```
docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```
容器启动后会返回一个 id，可以通过 docker ps 或 docker container ls 命令查看容器信息。
5. 查看容器输出

要查看容器的输出信息，通过 docker [container] logs 命令，该命令支持的选项

- -details：打印详细信息；
- -f，-follow：持续保持输出；
- -since string：输出从某个时间开始的日志；
- -tail string：输出最近的若干日志；
- -t，-timestamps：显示时间戳信息；
- -until string：输出某个时间之前的日志。