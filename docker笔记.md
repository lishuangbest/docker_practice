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

3. 使用 inspect 命令查看详细信息，命令格式
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

查看某容器的输出，可以使用下面的命令
```
docker logs 177
```

## 停止容器

相关命令：pause/unpause、stop 和 prune 子命令

1. 暂停容器

使用命令 docker [container] pause CONTAINER [CONTAINER...] 命令暂停一个运行中的容器。

```
docker ps
docker pause 177
docker ps
```
查看容器状态转换为（Paused）

2. 终止容器

使用 docker [container] stop 来终止一个运行中的容器。该命令的格式为 docker [container] stop [-t|--time[=10]] [CONTAINER...]

该命令会首先向容器发送 SIGTERM 信号，等待一段超时时间后（默认为10秒），在发送 SIGKILL 信号来终止容器：
```
docker stop 177
```
此时，执行 docker container prune 命令，自动清除所有处于停止状态的容器。还可以通过 docker [container] kill 直接发送 SIGKILL 信号来强行终止容器。处于终止状态的容器，可以通过 docker [container] start 命令重新启动。docker [container] restart 先将一个运行态的容器先终止，再重新启动。

## 进入容器

使用 -d 参数，容器启动进入后台，用户无法看到容器中的信息，也无法进行操作。可以使用 attach 或 exec 命令进入容器操作。

1. attach 命令

Docker 自带的命令，格式：
```
docker [container] attach [--detach-keys[=[]]] [--no-stdin] [--sig-proxy[=true]] CONTAINER
```

这个命令支持三个主要选项：

- detach-keys[=[]]：指定退出 attach 模式的快捷键序列，默认是 CTRL-p CTRL-q；
- --no-stdin=true|false：是否关闭标准输入，默认是保持打开；
- --sig-proxy=true|false：是否代理收到的系统信号给应用进程，默认为 true。

使用示例：
```
docker run -itd ubuntu
docker attach 8263
```
但是，当多个窗口同时 attach 到同一个容器的时候，所有的窗口都会同步显示；当某个窗口因命令阻塞时，其他窗口也无法执行操作了。

2. exec 命令

exec 命令是从 Docker 的1.3.0 版本起的，可以再运行中容器内直接执行任意命令。格式：
```
docker [container] exec [-d|--detach] [--detach-keys[=[]]] [-i|--interactive] [--privileged] [-t|--tty] [-u|--user[-USER]] CONTAINER COMMAND [ARG...] 
```

较为重要的参数有：

- -d，--detach：在容器中后台执行命令；
- --detach-keys=""：指定将容器切回后台的按键；
- -e，--env=[]：指定环境变量列表；
- -i，--interactive=true|false：打开标准输入接受用户输入命令，默认值为 false；
- --privileged=true|false：是否给执行命令以高权限，默认值为 false；
- -t，--tty=true|false：分配伪终端，默认值为 false；
- -u，--user=""：执行命令的用户名或 ID。

例如进入到刚创建的容器中，并启动一个bash：
```
docker exec -it 8263 /bin/bash
```
会打开一个新的 bash 终端，不影响其他用户的前提下，用户可以与容器进行交互。

## 删除容器

可以使用 docker [container] rm 命令删除处于终止或退出状态的容器，格式
```
docker [container] rm [-f|--force] [-l|--link] [-v|--volumes] CONTAINER [CONTAINER...]
```
支持的选项：

- -f，--force=false：是否强行终止并删除一个运行中的容器；
- -l，--link=false：删除容器的连接，但保留容器；
- -v，--volumes=false：删除容器挂载的数据卷。

例如查看全部容器，选择一个删除
```
docker ps -a
docker rm 8263
```
在使用 docker rm 命令时，默认只能删除已经处于终止或退出状态的容器，并不能删除还处于运行状态的容器。如果要直接删除一个运行中的容器可以添加 -f 参数，Docker 会先发送 SIGKILL 信号给容器，终止其中的应用，然后强行删除：
```
docker rm -f 015
```

## 导入和导出容器

1. 导出容器

导出容器是指，将一个已经创建的容器导出到一个文件，不管此时这个容器是否处于运行状态。可以使用 docker [container] export 命令，该命令格式为：
```
docker [container] export [-o|--output[=""]] CONTAINER
```
其中可以通过 -o 选项指定导出的 tar 文件名，也可以直接通过重定向来实现。
```
docker export -o hello.tar 8263
或
docker export 8263 > heheh.tar
```
导出的 tar 文件传输到其他机器上就可以通过导入命令直接导入到系统中，实现容器的迁移。

2. 导入容器

导出的 tar 文件可以使用 docker [container] import 命令导入变成镜像，格式：
```
docker import [-c|--change[=[]]] [-m|--message[=MESSAGE]] file|URL| - [REPOSITORY[:TAG]]
```
-c，--change=[] 选项在导入的同时执行对容器进行修改的Dockerfile指令。示例：
```
docker import hello.tar hello:v1.0
```

这个命令和 docker load 命令导入镜像文件类似，事实上，使用 docker load 导入镜像存出文件到本地镜像库，和 docker [container] import 导入一个容器快照到本地镜像仓库都可以使用，区别是：容器快照将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存出文件将保存完整记录，体积更大。此外，从容器快照文件导入时可以重新指定标签等元数据。

## 查看容器

inspect、top 和 stats 子命令。

1. 查看容器详情

使用 docker [container] inspect [OPTIONS] CONTAINER [CONTAINER...] 子命令。返回的容器具体信息包括容器 ID，创建时间，路径，状态，镜像，配置等信息在内的各项详细信息，以 json 格式返回。
```
docker inspect 174
```

2. 查看容器内进程

使用 docker [container] top [OPTIONS] CONTAINER [CONTAINER...] 子命令。类似 Linux 的 top 命令，会打印出容器内的进程信息，包括 PID、用户、时间、命令等。

```
docker top 17451
```

3. 查看统计信息

查看统计信息可以使用 docker [container] stats [OPTIONS] [CONTAINER...] 子命令，会显示 CPU、内存、存出、网络等使用情况的统计信息。选项包括：
- a，-all：输出所有容器统计信息，默认仅在运行中；
- -format string：格式化输出信息；
- -no-stream：不持续输出，默认会自动更新持续实时结果；
- -no-trunc：不截断输出信息。

```
docker stats 17451
docker stats -a
```

## 其他容器命令

包括 cp、diff、port 和 update 子命令

1. 复制文件

cp 命令支持在容器和主机之间复制文件。命令格式：

```
docker [container] cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
```
支持的选项包括：
- -a，-archive：打包模式，复制文件会带有原始的 uid/gid 信息
- -L，-follow-link：跟随软连接。当原路径为软连接时，默认只复制链接信息，使用该选项会复制链接的目标内容。

复制本地文件 hello.tar 到 17451 容器的 /tmp 路径下

```
docker cp hello.tar 17451:/tmp/
```

2. 查看变更

diff 命令查看容器内文件系统的变更。命令格式为：
```
docker [container] diff CONTAINER
```
查看 17451 的数据修改变更：
```
docker diff 17451
```

3. 查看端口映射

port 命令可以查看容器的端口映射情况。命令格式：
```
docker [container] port CONTAINER [PRIVATE_PORT[/PROTO]]
```
查看 17451 容器的端口映射情况：
```
docker port 17451
```

4. 更新配置

update 命令可以更新容器的一些运行时配置，主要是一些资源限制份额。命令格式为：
```
docker [container] update [OPTIONS] CONTAINER [CONTAINER...]
```

支持的选项：

- -blkio-weight uint16：更新块 IO 限制，10~1000，默认值为 0，代表着无限制；
- -cpu-period int：限制 CPU 调度器 CFS（Completely Fair Scheduler）使用时间，单位为微秒，最小1000；
- -cpu-quota int：限制 CPU 调度器 CFS 配额，单位为微秒，最小 1000；
- -cpu-rt-period int：限制 CPU 调度器的实时周期，单位为微秒；
- -cpu-rt-runtime int：限制 CPU 调度器的实时运行时，单位为微秒；
- -c，-cpu-shares int：限制 CPU 使用份额；
- -cpus decimal：限制 CPU 个数；
- -cpuset-cpus string：允许使用的 CPU 核，如 0-3，0，1；
- -cpuset-mems string：允许使用的内存块，如 0-3，0，1；
- -kernel-memory bytes：限制使用的内核内存；
- -memory-reservation bytes：内存软限制；
- -memory-swap bytes：内存加上缓存区的限制，-1 表示为对缓冲区无限制；
- -restart string：容器退出后的重启策略。

# 访问 Docker 仓库

## Docker Hub 公共镜像市场

1. 登录

命令 docker login 输入用户名、密码登录。本地用户文件夹自动创建 .docker/config.json 文件保存用户的认证信息。
```
docker login

```

2. 基本操作

用户即使不登陆也可以通过 docker search 命令查找官方仓库中的镜像，并利用 docker [image] pull 命令将它下载。

官方提供的镜像一般是由单个单词命名的，比如 ubuntu、centos 这样的镜像，也称为根镜像。另一类由用户创建并维护的镜像，命名格式为「用户名/镜像名」，使用用户名称作为前缀，表明是某个用户下的某个仓库。

用户登录后可以通过 docker push 命令将本地镜像推送到 Docker Hub。

3. 自动创建

自动创建（Automated Builds）是 Docker Hub 提供的自动化服务，自动跟随项目代码的变更重新构建镜像。比如，用户构建了某应用镜像，如果应用发布新版本，则用户需要手动更新镜像，自动创建就是用户通过 Docker Hub 指定跟踪一个目标网站（目前支持 GitHub 或 BitBucket）上的项目，一旦项目发生新的提交，则自动执行创建。

要配置自动创建，包括如下步骤：

1. 创建并登录 Docker Hub，以及目标网站如 Github；
2. 在目标网站中允许 Docker Hub 访问服务；
3. 在 Docker Hub 中配置一个「自动创建」类型的项目；
4. 选取一个目标网站中的项目（需要含 Dockerfile）和分支；
5. 指定 Dockerfile 的位置，并提交创建。

之后，可以在 Docker Hub 的「自动创建」页面中跟踪每次创建的状态。

## 第三方镜像市场

下载镜像

```
docker pull index.tenxcloud.com/<namespace>/<repository>:<tag>
```

比如下载 node:latest 

```
docker pull index.tenxcloud.com/docker_library/node:latest
```
这是使用时速云仓库下载的命令，下载后可以将镜像标签修改为和官方一致的标签。

## 搭建本地私有仓库

1. 使用 registry 镜像创建私有仓库

通过官方提供的 registry 镜像简单搭建一套本地私有仓库环境：

```
docker run -d -p 5000:5000 registry:2
```

会自动下载并启动一个 registry 容器，创建本地的私有仓库服务。默认情况仓库会被放在创建容器的 /var/lib/registry 目录下。可以通过 -v 参数来将镜像文件存放在本地的指定仓库路径。例如下面的例子将上传的镜像放到 /opt/data/registry 目录
```
docker run -d -p 5000:5000 -v /opt/data/registry:/var/lib/registry registry:2
```
此时，在本地将启动一个私有仓库服务，监听端口为 5000。

2. 管理私有仓库

见 《Docker 技术入门与实战 第 3 版》第五章 5.3 搭建本地私有仓库。

# Docker 数据管理

容器中的管理数据主要有两种方式：

- 数据卷（Data Volumes）：容器内数据直接映射到本地主句环境；
- 数据卷容器（Data Volume Containers）：使用特定容器维护数据卷。

## 数据卷

数据卷（Data Volumes）是一个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似于 Linux 中的 mount 行为。

数据卷可以提供很多有用的特性：

- 数据卷可以在容器之间共享和重用，容器间传递数据将变得高效与方便；
- 对数据卷内数据的修改会立马生效，无论是容器内操作还是本地操作；
- 对数据卷的更新不会影响镜像，解耦开应用的数据；
- 卷会一直存在，直到没有容器使用，可以安全地卸载它。

1. 创建数据卷

volume 命令管理数据卷，快速在本地创建数据卷：

```
docker volume create -d local test
```

此时，查看 /var/lib/docker/volumes 路径下，会发现所创建的数据卷位置：

```
ls -l /var/lib/docker/volumes
```

除了 create 子命令外，docker volume 还支持 inspect（查看详细信息）、ls（列出已有数据卷）、prune（清理无用数据卷）、rm（删除数据卷）等。

2. 绑定数据卷

除了使用 volume 命令来管理数据卷外，还可以创建容器时将主机本地的任意路径挂载到容器内作为数据卷，这种形式创建的数据卷称为绑定数据卷。

在用 docker [container] run 命令的时候，可以使用 -mount 选项来使用数据卷。

-mount 选项支持三种类型的数据卷

- volume：普通数据卷，映射到主机 /var/lib/docker/volumes 路径下；
- bind：绑定数据卷，映射到主机指定路径下；
- tmfs：临时数据卷，只存才于内存中。

下面使用 training/webap 镜像创建一个 Web 容器

```
docker run -d -P --name web --mount type=bind,source=/webapp,destination=/opt/webapp training=webapp python app.py
```

上述命令等同于使用旧的 -v 标记可以在容器内创建一个数据卷：

```
docker run -d -P --name web -v /webapp:/opt/webapp training/webapp python app.py
```

这个功能在进行应用测试的时候十分方便，比如用户可以放置一些程序或数据到本地目录中实时进行更新，然后在容器内运行和使用。

另外，本地目录的路径必须是绝对路径，容器内路径可以为相对路径。如果目录不存在，Docker 会自动创建。

Docker 挂在数据卷的默认权限是读写（rw），用户也可以通过 ro 指定为只读：

```
docker run -d -P --name web -v /webapp:/opt/webapp:ro training/webapp python app.py
```

加了 :ro 后，容器内对所挂载数据卷内的数据就无法修改了。如果直接挂载一个文件到容器，使用文件编辑工具，包括 vi 或 sed --in-place 的时候，可能会造成文件 inode 的改变。从 Docker 1.1.0 起，这会导致报错误信息。所以推荐的方式是直接挂载文件所在的目录到容器内。

## 数据卷容器

如果用户需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器。数据卷容器也是一个容器，但是它的目的是专门提供数据卷给其他容器挂载。

首先创建一个数据卷容器 dbdata，并在其中创建一个数据卷挂载到 /dbdata：

```
docker run -it -v /dbdata --name dbdata ubuntu
```

然后可以在其他容器中使用 -volumes-form 来挂在 dbdata 容器中的数据卷，例如创建 db1 和 db2 两个容器，并从 dbdata 容器挂载数据卷：

```
docker run -it --volumes-from dbdata --name db1 ubuntu
docker run -it --volumes-from dbdata --name db2 ubuntu
```

此时，容器 db1 和 db2 都挂载同一个数据卷到相同的 /dbdata 目录，三个容器任何一方在该目录下的写入，其他容器看不到。比如，在 dbdata 容器中创建一个 test 文件：

```
cd /dbdata
touch test
ls
```

在 db1 容器内查看它：

```
docker run -it --volumes-from dbdata --name db1 ubuntu
ls dbdata/
```

可以多次使用 --volumes-from 参数来从多个容器挂载多个数据卷，还可以从其他已经挂在了容器卷的容器来挂载数据卷：

```
docker run -d --name db3 --volumes-from db1 traning/postgres
```

注意：使用 --volumes-from 参数所挂载数据卷的容器自身并不需要保持在运行状态。

如果删除了挂载的容器（包括 dbdata、db1 和 db2），数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时显式使用 docker run -v 命令来指定同时删除关联的容器。

## 利用数据卷容器来迁移数据

可以利用数据卷容器对其中的数据卷进行备份、回复，以实现数据的迁移。

1. 备份

使用下面的命令来备份 dbdata 数据卷容器内的数据卷：

```
docker run --volumes-from dbdata -v $(pwd):/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata
```

首先利用 ubuntu 镜像创建了一个容器 worker。使用 --volumes-from dbdata 参数来让 worker 容器挂载 dbdata 容器的数据卷（即 dbdata 数据卷）;使用 -v $(pwd):/backup 参数来挂载本地的当前目录到 worker 容器的 /backup 目录。

worker 容器启动后，使用 tar cvf /backup/backup.tar /dbdata 命令将 /dbdata 下内容备份为容器内的 /backup/backup.tar，即宿主主机当前目录下的 backup.tar。

2. 恢复

如果要恢复数据到一个容器，可以按照下面的操作。首先创建一个带有数据卷的容器 dbdata2：

```
docker run -v /dbdata --name dbdata2 ubuntu /bun/bash
```

然后创建另一个新容器，挂载 dbdata2 的容器，并使用 untar 解压备份文件到所挂载的容器卷中：

```
docker run --volumes-from dbdata2 -v $(pwd):/backup --name busybox ubuntu tar xvf
```

# 端口映射与容器互联

## 端口映射实现容器访问

1. 从外部访问容器应用

启动容器时，不指定对应参数，在容器外无法通过网络来访问容器内的网络应用和服务。

在容器中运行一些网络应用，要让外部访问这些应用时，可以通过 -P 或 -p 参数来指定端口映射。当使用 -P 标记时，Docker 会随机映射一个 49000 ~ 49900 的端口到内部容器开放的网络端口：

```
docker run -d -P training/webapp python app.py
```

使用 -p 可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有 IP:HostPort:ContainerPort | IP::ContainerPort | HostPost:ContainerPort。


2. 映射所有接口地址

使用 HostPort:ContainerPort 格式本地的 5000 端口映射到容器的 5000 端口，可以执行以下命令：

```
docker run -d -p 5000:5000 training/webapp python app.py
```

此时默认会绑定本地所有接口上的所有地址。多次使用 -P 标记可以绑定多个端口。例如：

```
docker run -d -p 5000:5000 -p 3000:80 training/webapp python app.py
```

3. 映射到指定地址的端口

可以使用 IP:HostPort:ContainerPort 格式指定映射使用一个特定地址，比如 localhost 地址 127.0.0.1：

```
docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

4. 映射到指定地址的任意端口

使用 IP::ContainerPort 绑定 localhost 的任意端口到容器的 5000 端口，本地主机会自动分配一个端口：

```
docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

还可以使用 udp 标记来指定 udp 端口：

```
docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```

5. 查看映射端口配置

使用 docker port 来查看当前映射的端口配置，也可以查看到绑定的地址：

```
docker port eloquent_mendel 5000
```

容器有自己的内部网络和 IP 地址，使用 docker [container] inspect + 容器 ID 可以获取容器的具体信息。

## 互联机制实现便捷互访

容器的互联（linking）是一种让多个容器中的应用进行快速交互的方式。会在源和接收容器之间创建连接关系，接收容器可以通过容器名快速访问到源容器，而不用指定具体的 IP 地址。

1. 自定义容器命名

```
docker run -d -P --name web training/webapp python app.py
```

查看容器名字

```
docker [container] inspect -f "{{ .Name }}" aed84ee21bde
```

执行 docker [container] run 的时候添加 --rm 标记，则容器在终止后会立刻删除。--rm 和 -d 参数不能同时使用。

2. 容器互联

使用 --link 参数可以让容器之间安全地进行交互。

创建一个新的数据库容器：

```
docker run -d --name db training/postgres
```

创建一个新的 web 容器，并连结到 db 容器：

```
docker run -d -P --name web --link db:db training/webapp python app.py
```

此时，db 容器和 web 容器建立互联关系。

--link 参数的格式为 --link name:alias，其中 name 是要链接的容器的名称。alias 是别名。

通过 docker ps 命令查看容器的连接，可以看到自定义命名的容器 db 和 web，db 容器的 names 列有 db 也有 web/db，这表示 web 容器链接到 db 容器，web 容器将被允许访问 db 容器的信息。

Docker 相当于两个互联的容器之间创建了一个虚机通道，而且不用映射它们的端口到宿主主机上。在启动 db 容器的时候并没有使用 -p 和 -P 标记，从而避免了暴露数据库服务端口到外部网络上。

Docker 通过两种方式为容器公开连接信息：

- 更新环境变量；
- 更新 /etc/hosts 文件。

使用 env 命令来查看 web 容器的环境变量：

```
docker run --rm --name web2 --link db:db training/webapp env
```

其中 DB_ 开头的环境变量是供 web 容器连接 db 容器使用，前缀采用大写的连接别名。

除了环境变量，Docker 还添加 host 信息到父容器的 /etc/hosts 的文件。

```
docker run -t -i --rm --link db:db training/webapp /bin/bash

cat /etc/hosts
```

这里有 2 个 hosts 信息，第一个是 web 容器，web 容器用自己的 id 作为默认主机名，第二个是 db 容器的 IP 和主机名。可以在 web 容器中安装 ping 命令来测试跟 db 容器的连通。

```
apt-get install -yqq inetutils-ping
ping db
```

# 使用 Dockerfile 创建镜像

Dockerfile 是一个文本格式的配置文件，用户可以使用 Dockerfile 来快速创建自定义镜像。

Dockerfile 由命令语句组成，支持以 # 开头的注释行。一般而言，Dockerfile 主题内容分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。见《Docker 技术入门与实战 第 3 版》 8章，68页。

