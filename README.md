# Gitlab CI/CD + Docker 前端持续集成部署

基于 gitlab ci/cd + docker 进行前端自动化部署。

环境说明：基于阿里云的服务器，配置为 2 核 4G，带宽为 3M，系统是 CentOS 7.7 64 位。



**目录：**

Docker 基本使用：[Docker](#1、Docker 基本使用)

Linux 常用命令：[Linux](#2、Linux 常用命令)

Gitlab CI/CD 持续集成部署：[Gitlab CI/CD](#3、Gitlab-CI/CD 持续集成部署)



## 1、Docker 基本使用

官方文档：https://docs.docker.com/



### 1-1、虚拟机与容器技术

**虚拟机：**

 <img src="/imgs/img1.png" style="zoom: 50%;" />

传统的虚拟机需要模拟整台机器包括硬件，每台虚拟机都需要有自己的操作系统，虚拟机一旦被开启，预分配给他的资源将全部被占用。每一个虚拟机包括应用，必要的二进制和库，以及一个完整的用户操作系统。这就带来了问题：

- 资源占用多
- 启动慢



**容器：**

 <img src="/imgs/img2.png" style="zoom:50%;" />

容器技术是与宿主机共享硬件资源及操作系统可以实现资源的动态分配。容器包含应用和其所有的依赖包，但是与其他容器**共享内核**。容器在宿主机操作系统中，在用户空间以分离的进程运行。



**容器 VS 虚拟机：**

虚拟机技术与容器技术的最大区别在于：**多个虚拟机使用多个操作系统内核，而多个容器共享宿主机操作系统内核**。

|    特点    |        容器        |          虚拟机          |
| :--------: | :----------------: | :----------------------: |
|    启动    |        秒级        |          分钟级          |
|  硬盘使用  |         MB         |            GB            |
|    性能    |      接近原声      |         弱于原生         |
| 系统支持量 | 单机支持上千个容器 |       一般为几十个       |
|   隔离性   |      进程级别      |         系统级别         |
|   安全性   |         中         | 强（完全虚拟了一个系统） |



### 1-2、docker 概述

Docker 是一个用于开发，发布和运行应用程序的开放平台，docker 是一个容器技术。

Docker 的思想来源于集装箱。在一艘大船上，各种货物要想被整齐摆放并且相互不受到影响，我们就需要把各种货物进行集装箱标准化。有了集装箱，就不需要专门运输水果或者化学用品的船就可以把各种货品通过集装箱打包，然后统一放到一艘船上运输。Docker 就是把各种软件打包成一个集装箱（镜像），然后分发，在运行的时候做到相互隔离。



### 1-3、为什么使用 Docker 进行项目部署

- 在开发的时候，在本机测试环境可以跑，生产环境跑不起来

  比如一个 vue 项目，里面使用了许多依赖，当这些依赖的某一项版本不一致的时候，就可能就会导致应用程序跑不起来这种情况。**Docker 则将程序以及软件环境直接打包在一起，无论在哪个机器上都保证了环境一致。**

- 服务器自己的程序挂了，结果发现是别人程序出了问题把内存吃完了，自己程序因为内存不够就挂了

  这也是一种比较常见的情况，如果你的程序重要性不是特别高的话，公司基本上不可能让你的程序独享一台服务器，这时候你的程序就会跟公司其他人的程序共享一台服务器，所以不可避免地就会受到其他程序的干扰，导致自己的程序出现问题。Docker 很好解决了环境隔离的问题，别人程序不会影响到自己的程序。这也是 Docker 的一个特点：**对进程进行封装隔离,容器与容器之间互不影响**

- 更加高效地利用服务器资源

  Docker 的容器硬盘占用是非常少的，一般是 MB 级别，那么就意味着，一台服务器，能部署更多的项目。

- 更加便捷的升级和扩容

  使用了 Docker 后，部署就像搭积木一样，将项目打包为一个镜像，可以快速地部署到多个服务器



### 1-4、docker 的几个概念

**镜像：**

通俗地讲，它是一个**只读的文件与文件夹的组合**。它包含了容器运行时所需要的所有基础文件和配置信息，是容器启动的基础。镜像是一个静态概念，不包含任何动态的数据，其内容在构建后也不会改变。

实际上，可以把镜像看作是一个类，容器就是类的实例，通过镜像类可以创建多个容器



**容器：**

容器是镜像的运行实体，镜像是静态的，容器可以理解为正在运行的镜像，是动态的，即**容器运行着真正的应用进程**。容器有初建、运行、停止、暂停、删除五种状态，它是可读可写的。



**仓库：**

Docker 的镜像仓库类似于代码仓库，用来存储和分发 Docker 镜像。镜像仓库分为公共镜像仓库和私有镜像仓库。

目前，[Docker Hub](https://hub.docker.com/) 是 Docker 官方的公开镜像仓库，它不仅有很多应用或者操作系统的官方镜像，还有很多组织或者个人开发的镜像供我们免费存放、下载、研究和使用。除了公开镜像仓库，也可以构建自己的私有镜像仓库，例如利用 [Harbor](https://goharbor.io/) 来构建一个企业级镜像仓库。



**三者的基本关系**

 <img src="/imgs/img3.png" style="zoom:50%;" />



###  1-5、docker 的安装

docker 是一个跨平台的解决方案，它支持各大主流平台，包括 Ubuntu、RHEL、CentOS、Debian 等 Linux 环境，同时也可以在Mac、Windows 等非 Linux 平台下安装使用。

下面主要使用 CentOS 安装 Docker：需要 CentOS 7 及以上的发行版本。



如果已经安装过 Docker，可以执行以下命令卸载旧版 Docker

> sudo 作用：使用 root 用户权限操作，主要用来提升权限。

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```



**安装 Docker**

官网 centOS 安装教程：https://docs.docker.com/engine/install/centos/ 



1、安装`yum-utils`包（提供`yum-config-manager` 简化工具，用于修改 yum 的安装源）

```shell
sudo yum install -y yum-utils
```

> 网上看到不少教程都会执行
>
> ```shell
> sudo yum install -y yum-utils device-mapper-persistent-data lvm2
> ```
>
> 旧版的 docker 需要安装，新版的不需要



2、首次安装 Docker，需要添加 Docker 安装源。添加之后，就可以从已经配置好的源，安装和更新 Docker。命令如下：

```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

但是官方的 Docker 源在国外，会比较慢，所以一般使用阿里云的 Docker 源：

```shell
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```





3、一般情况下，安装最新版本的 Docker 即可，最新版本的 Docker 有着更好的稳定性和安全性。命令如下：

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

- docker-ce：社区版，免费的

如果想要安装指定版本的 Docker，可以先列出 Docker 各个版本：

```shell
yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64            18.06.1.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos            docker-ce-stable
```

然后指定需要安装的版本：

```shell
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```



4、安装完成后，启动 Docker：

```shell
sudo systemctl start docker
```



5、通过运行 `hello-world` 镜像验证 Docker 是否安装成功：

```shell
sudo docker run hello-world
```

如果输出下面一段代码，说明安装成功，解析：运行上述命令，Docker 首先会检查本地是否有 `hello-world` 这个镜像，如果发现本地没有这个镜像，Docker 就会去 Docker Hub 官方仓库下载此镜像，然后运行它。最后看到该镜像输出 "Hello from Docker!" 并退出，那么说明安装成功

```shell
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:7f0a9f93b4aa3022c3a4c147a449bf11e0941a1fd0bf4a8e6c9408b2600777c5
Status: Downloaded newer image for hello-world:latest
Hello from Docker!
```

当然，也可以通过查看 docker 版本信息验证 docker 是否安装成功

```shell
sudo docker version
```

输出版本号等信息，也说明安装成功



6、安装完成后默认 docker 命令只能以 root 用户执行，如果想允许普通用户执行 docker 命令，需要执行以下命令配置权限并重启 docker：

```shell
sudo groupadd docker # 创建 docker 用户组

sudo gpasswd -a ${USER} docker # 将当前用户加入到 docker 的用户组中

sudo systemctl restart docker # 重启 docker
```

执行完命令后，退出当前命令行窗口并打开新的窗口即可。

- `systemctl restart docker`：重启 docker
- `systemctl start docker`：启动 docker
- `systemctl stop docker`：关闭 docker



7、卸载 docker

首先，卸载 docker 依赖：

```shell
sudo yum remove docker-ce docker-ce-cli containerd.io
```

第二步，删除 docker 资源目录：

```shell
sudo rm -rf /var/lib/docker # docker 的默认工作路径就是 /var/lib/docker
sudo rm -rf /var/lib/containerd
```



### 1-6、配置阿里云镜像服务

上面配置的是 Docker 安装源，只是在下载 Docker 的时候加速，但是使用 Docker 去拉取镜像的时候,并不会进行加速，那么就需要配一下镜像源，目前比较好用的是阿里云的镜像源。

1. 首先，登陆阿里云，搜索 ==容器镜像服务==【需要登陆的原因是每个人的镜像加速地址都不一样】

   <img src="/imgs/img4.png" style="zoom:50%;" />

2. 然后点击管理控制台

   ![](/imgs/img5.png)

3. 进来就可以看到

   ![](/imgs/img6.png)

4. 然后，根据系统配置镜像加速即可

   ```shell
   sudo mkdir -p /etc/docker
   
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://ja96ojhg.mirror.aliyuncs.com"]
   }
   EOF
   
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

分别执行完上面几步，可以通过

```shell
docker info
```

查看是否配置好镜像加速，如果输出的信息包含如下信息，代表配置成功

```shell
  127.0.0.0/8
 Registry Mirrors:
  https://ja96ojhg.mirror.aliyuncs.com/
```



### 1-7、docker 帮助命令

```shell
docker version        # docker 版本信息

docker info           # docker 的系统信息

docker --help         # 查看一些 docker 命令帮助

docker <指令> --hellp  # 查看指定指令的帮助
```



### 1-8、docker 镜像基本命令

前面说过，镜像是一个只读的 Docker 容器模板，包含启动容器所需要的所有文件系统结构和内容。简单来讲，镜像是一个特殊的文件系统，它提供了容器运行时所需的程序、软件库、资源、配置等静态数据。即镜像不包含任何动态数据，镜像内容在构建后不会被改变。

下面来看看一些 docker 镜像相关的命令。



#### 搜索镜像

可以使用 `docker search <imageName>` 搜索镜像，就相当于去 `Docker Hub` 上搜索官方的镜像一样

```shell
docker search nginx

# 输出
NAME      DESCRIPTION                   STARS     OFFICIAL   AUTOMATED
nginx     Official build of Nginx...    15034     [OK]

# 解析
NAME             镜像名
DESCRIPTION      镜像描述
STARS            镜像的 start
OFFICIAL         官方的
```



#### 拉取镜像

完整的拉取镜像命令如下：

```shell
docker pull [Registry]/[Repository]/[Image]:[Tag]
```

- Registry：为注册服务器，Docker 默认会从 docker.io 拉取镜像，如果有自己的镜像仓库，可以把 Registry 替换为自己的注册服务器。或者下载第三方的镜像资源也可以设置第三方镜像资源的镜像仓库
- Repository：镜像仓库，通常把一组相关联的镜像归为一个镜像仓库，library 是 Docker 默认的镜像仓库。
- Image：镜像名称
- Tag：镜像的标签【也就是版本】，如果不指定拉取镜像的标签，默认为latest

例如拉取 nginx

```js
docker pull docker.io/library/nginx:latest
```

如果都使用默认选项，可以简写为：

```js
docker pull nginx
```



例如这里拉取一下 nginx 镜像

```shell
docker pull nginx

# 输出
Using default tag: latest                # 没有指定镜像 tag，默认使用 latest
latest: Pulling from library/nginx
69692152171a: Pull complete              # 分层下载，docker images 的核心
30afc0b18f67: Pull complete 
596b1d696923: Pull complete 
febe5bd23e98: Pull complete 
8283eee92e2f: Pull complete 
351ad75a6cfa: Pull complete 
Digest: sha256:6d75c99af15565a301e48297fa2d121e15d80ad526f8369c526324f0f7ccb750  # 签名，防伪
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest  # 真实地址

# 一个完整的拉取命令是：docker pull [Registry]/[Repository]/[Image]:[Tag]
# 这里的 docker pull nginx 等价于：
#  docker pull docker.io/library/nginx:latest
```



#### 查看镜像

拉取了镜像，就可以查看本地所有镜像，使用 `docker images` 或者 `docker image ls`

```shell
docker images

# 输出
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    d1a364dc548d   3 weeks ago    133MB
hello-world   latest    d1165f221234   3 months ago   13.3kB

# 解析
REPOSITORY   镜像仓库源【也就是镜像名】
TAG          镜像标签
IMAGE ID     镜像id
CREATED      镜像创建时间
SIZE         镜像大小
```

可以使用 `docker image ls ` 来查询指定的镜像

```shell
docker image ls nginx

# 输出
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx        latest    d1a364dc548d   3 weeks ago   133MB
```

或者使用 `docker images` 命令列出所有镜像，然后使用 grep 命令进行过滤

```shell
docker images |grep nginx  # |grep 之间不要有空格

# 输出
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx        latest    d1a364dc548d   3 weeks ago   133MB
```

可以通过 `docker images -q` 或者 `docker images -aq` 列出所有的镜像id

```shell
docker images -q

# 输出
d1a364dc548d
d1165f221234
```



#### 重命名镜像

如果想要自定义镜像名称或者推送镜像到其他镜像仓库，可以使用 docker tag 命令将镜像重命名。docker tag 的命令格式为：

```shell
docker tag [SOURCE_IMAGE][:TAG] [TARGET_IMAGE][:TAG]
```

例如：

```shell
docker tag hello-world hello
```

再使用查看镜像命令：

```shell
docker images

# 输出
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    d1a364dc548d   3 weeks ago    133MB
hello-world   latest    d1165f221234   3 months ago   13.3kB
hello         latest    d1165f221234   3 months ago   13.3kB
```

可以看到，镜像列表中多了一个 hello 的镜像。hello 和 hello-world 这两个镜像的 `镜像ID` 是完全一样的。为什么呢？实际上它们指向了同一个镜像文件，只是别名不同而已。



#### 删除镜像

可以使用 `docker rmi` 或者 `docker image rm` 命令删除镜像，如果需要强制删除，那么加上 -f，例如：

```shell
docker rmi -f

docker image rm -f
```

可以根据镜像名或者镜像id 进行删除

例如：

```shell
docker rmi -f hello                 # 根据镜像名删除

docker rmi -f d1165f221234          # 根据镜像id删除

dokcer rmi -f $(docker images -aq)  # 查找出所有镜像id，然后全部删除
```



### 1-9、docker 容器基本命令

上面说过，容器是基于镜像创建的可运行实例，并且单独存在，一个镜像可以创建出多个容器。运行容器化环境时，实际上是在容器内部创建该文件系统的读写副本。 容器层允许修改镜像的整个副本。

既然容器是基于镜像创建的，那么先 pull 一个 node 镜像来进行容器的操作。

```shell
docker pull node
```

然后执行一下：

```shell
docker images

# 输出
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
node         latest    d1b3088a17b1   2 weeks ago   908MB
```

拉去镜像成功



#### 启动容器

启动容器的方式有两种：

1、如果这个容器已经存在，直接使用 `docker start` 启动，后面可以跟容器名或者容器id

```shell
docker start <容器id | 容器名>
```



2、如果这个容器不存在，那么需要先根据镜像创建容器，然后启动

使用命令 `docker run`

```shell
docker run [可选参数] image   # 根据镜像 image 创建容器，并且启动容器

# 常用的可选参数
--name container_name          定义这个容器名字
-d                             后台运行【全称是 --detach】
-h                             指定运行的 hostname，可以是域名也可以是 IP【全称是 --hostname】
-p                             端口的映射【全称是 -publish】
      -p 主机端口:容器端口
      -p 容器端口

--restart                      重启方式
      --restart no 默认策略，在容器退出时不重启容器
      --restart on-failure 在容器非正常退出时（退出状态非 0），才会重启容器
      --restart on-failure:3 在容器非正常退出时重启容器，最多重启 3 次
      --restart always 在容器退出时总是重启容器
      --restart unless-stopped 在容器退出时总是重启容器，不会考虑在 Docker 守护进程启动时就已经停止了的容器

-i                             以交互模式运行容器，通常与-t一起使用
-t                             分配一个伪终端
-it                            通常 -i 与 -t 配合使用进入交互模式，在交互模式下，可以通过所创建的终端来输入命令
-v                             将容器的数据卷在主机中存一份【全称是 --volume】
     -v /srv/gitlab/config:/etc/gitlab   主机数据卷:容器数据卷
```

- 端口映射：一般为 主机端口:容器端口，代表通过将容器的某个端口映射到主机的某个端口，可以访问主机端口的时候访问到容器内部的端口

这里执行：

```shell
docker run -it --name node node
```

这里代表：

- 以 node 镜像创建并启动一个容器
- --name node：容器名为 node
- -it /bin/bash：以交互模式进入到容器的终端



docker run 的基本流程：

 <img src="/imgs/img7.png" style="zoom:50%;" />



#### 停止、关闭、重启容器

```shell
docker restart <容器id | 容器名>         重启容器

docker stop <容器id | 容器名>            正常停止容器运行

docker kill <容器id | 容器名>            立即停止容器运行
```



#### 查看容器

```shell
docker ps           # 列出所有正在运行的容器

docker ps -a        # 列出所有容器【a：即 all 的简写】

docker ps -q        # 列出所有正在运行的容器id

docker ps -aq       # 列出所有的容器id
```



#### 查看容器内进程

```shell
docker top <容器id | 容器名>

UID         PID         PPID        C          STIME        TTY         TIME          CMD
root        16887       16866       0          17:33        pts/0       00:00:00      /bin/bash

# 解析
UID：当前的用户id
PID：父id
PPID：进程id
C：
STIME：
TTY：
TIME：
CMD：
```

> 注意：查看进程，需要当前容器是运行状态



#### 查看容器的运行日志

```shell
docker logs [options] <容器id | 容器名>
  options:
    -tail 显示多少条
```



#### 查看容器元数据【容器内信息】

```shell
docker inspect <容器id | 容器名>
```

这个命令能查看当前容器的所有信息



#### 进入、退出容器

进入容器：

```shell
docker exec -it <容器id | 容器名> command

# 解析
<container_id|container_name>：<容器id或者容器名>
command：表示 linux 命令,如/bin/bash
```

 第二种方式

```shell
docker attach <容器id | 容器名>
```

- `docker exec`： 进入容器后开启一个新终端
- `docker attach`：进入容器正在运行的终端



或者在 `docker run` 的时候：【注意：/bin/bash 需要放在最后面】

```js
docker run -it --name node node /bin/bash
```

![](/imgs/img9.png)

会发现，进入容器后，终端明显是不同的。并且通过 ls 输出所有目录，通过 `node --version` 输出了当前 node 版本，说明确实进入到了 node 容器的 /bin，并启动了 bash 终端



退出容器：

```shell
exit              # 退出并停止容器

Ctrl + P + Q      # 退出但是不停止容器
```



#### 从容器内拷贝文件到主机

```shell
# 首先，进入容器
docker exec -it node

# 从容器内拷贝文件到主机
docker cp 容器id:容器内路径 目的主机路径
```

这种方式是通过手动拷贝的形式，更好的方法是通过数据卷 -v 的方式，自动同步容器与主机文件，例如之前的 dokcer run

```js
dokcer run gitlab -v /srv/gitlab/config:/etc/gitlab
```

自动将容器 /srv/gitlab/config 的内容同步到主机的 /etc/gitlab



#### 删除容器

```shell
docker rm -f <容器id | 容器名>         # 根据容器id 或者容器名删除容器

docker rm -f $(docker ps -aq)        # 查找出所有容器id，然后删除所有容器
```



### 1-10、数据卷

> 为什么需要数据卷？

- 如果数据都在容器中，删除容器或者重启容器，数据就会丢失
- 如果两个容器之间，想要进行数据共享，怎么办



在生产环境中使用 Docker ，往往需要对数据进行持久化，或者需要在多个容器之间进行数据共享；数据卷就可以解决上面的问题，它是一个可供一个或多个容器使用的特殊目录，特点：

- 容器可以利用数据卷与宿主机进行数据共享，或者多个容器之间进行数据共享
- 容器对数据卷的修改是实时进行的，修改之后会马上生效
- 数据卷的变化不会影响镜像的更新
- 数据卷默认会一直存在，即使容器被删除

说白了，就是将容器中的目录或者文件挂载份到主机上，同步更新。



#### 使用数据卷

当我们想把主机的目录映射到容器内时，就需要用到主机与容器之间数据共享的方式。例如把 centos 容器中的 /home 目录映射到主机的 /home/centos 目录中，实现方式为：启动容器的时候添加 -v 参数, 格式：-v 主机文件路径:容器文件路径

```shell
docker run -v /home/centos:/home -it centos
```

容器启动后，便可以在容器内的 /home 访问到主机 /home/centos 目录的内容了，并且容器重启后，/home/centos 目录下的数据也不会丢失

通过查看容器信息：

```shell
docker inspect 容器id
```

可以看到：

 <img src="/imgs/img11.png" style="zoom:50%;" />



**数据卷同步**

![](/imgs/img12.png)

1. 进入主机的 /home/centos 目录，执行 ls，没有文件
2. 进入容器的 /home 目录，执行 ls，没有文件
3. 在主机的 /home/centos 下新建 hello.j，执行 ls，可以发现目录下多了一个 hello.js；同时在容器的 /home 下执行 ls，发现也存在 hello.js 文件。



**容器被删除，主机数据卷也还在**

![](/imgs/img13.png)

1. 执行删除容器命令 `docker rm -f`
2. 查看容器 `docker ps -a`，已经没有容器，确定被删除
3. 进到数据 home 目录，执行 ls，发现 centos 目录还在
4. 再进到 centos 目录，执行 ls，发现 hello.js 还在



### 1-11、commit 构建镜像

Docker 中构建镜像的方式主要有两种：

- 使用 `docker commit` 命令将运行中的容器提交为镜像
- 使用 DockerFile 文件构建镜像

 `docker commit` 基本命令

```shell
docker commit -m="提交的描述信息" -a="作者" <容器id | 容器名> 生成的镜像名:[tag版本]
```



**实践一下：**

1. 首先，通过 node 镜像以交互终端的形式启动一个 node 容器

   ```shell
   docker run -it node /bin/bash
   
   
   # 输出，说明进入到了 node 容器的交互终端
   root@7a6ffe0e4002:/# 
   
   
   # 执行 ls 查看当前目录
   root@7a6ffe0e4002:/# ls
   bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
   ```

2. 新建一个 hello.js 文件，并写入 `console.log('hello, node')`

   ```shell
   root@7a6ffe0e4002:/# touch hello.js && echo "console.log('hello, node')" > hello.js
   
   
   # 再通过 ls 查看当前目录，发现多了一个 hello.js 文件，说明没有问题
   root@7a6ffe0e4002:/# ls
   bin  boot  dev	etc  hello.js  home  lib  lib64  media	mnt  opt  proc	root  run  sbin  srv  sys  tmp	usr  var
   
   
   # 执行一下 hello.js 文件，正常打印
   root@7a6ffe0e4002:/# node hello.js
   ```

3. Ctrl + P + Q 退出当前容器，执行 `docker commit` 构建镜像

   ```shell
   docker commit 7a6ffe0e4002 node-hello:1.0.0
   ```

4. 最后，通过 `docker images` 查看所有镜像，发现存在 node-hello 镜像

   ![](/imgs/img10.png)



> ps：这种方式构建镜像了解一下即可，生产实践中不推荐是用这种方法构建镜像，优先使用 DockerFile 构建镜像



### 1-12、Dockerfile

生产实践中强烈建议**优先使用 Dockerfile 的方式构建镜像**。 使用 Dockerfile 构建镜像可以带来很多好处：

- 易于版本化管理，Dockerfile 本身是一个文本文件，方便存放在代码仓库做版本管理，可以很方便地找到各个版本之间的变更历史；
- 过程可追溯，Dockerfile 的每一行指令代表一个镜像层，根据 Dockerfile 的内容即可很明确地查看镜像的完整构建过程；
- 屏蔽构建环境异构，使用 Dockerfile 构建镜像无须考虑构建环境，基于相同 Dockerfile 无论在哪里运行，构建结果都一致。



#### Dockerfile 构建镜像步骤

1. 编写 Dockerfile 文件
2. docker build 构建镜像
3. docker run 运行镜像
4. docker push 发布到镜像仓库



#### Docker 镜像原理

Docker 镜像是由一系列镜像层（layer）组成的，每一层代表了镜像构建过程中的一次提交。下面以一段 DockerFile 代码来说明：

```shell
FROM nginx:alpine

COPY nginx/default.conf /etc/nginx/conf.d

COPY dist/ /usr/share/nginx/html
```

这个 Dockerfile 由三步组成:

- 基于 nginx 构建一个镜像
- 复制主机 nginx/default.conf 到镜像内的 /etc/nginx/conf.d
- 复制主机 dist/ 到镜像内的 /usr/share/nginx/html

实际上，Dockerfile 的每一行命令，都生成了一个镜像层。

 <img src="/imgs/img19.png" style="zoom:50%;" />

这张图就很好解析了 Docker 镜像分层的概念：基于 centos 创建一个基础镜像是一层，下载 jdk 是一层，下载 tomcat 是一层，最后将这些镜像层打包启动为一个可读写的容器。

分层的结构使得 Docker 镜像非常轻量，每一层根据镜像的内容都有一个唯一的 ID 值，当不同的镜像之间有相同的镜像层时，便可以实现不同的镜像之间共享镜像层的效果。



#### Dockerfile 常用指令

| DockerFile 指令 | 指令简介                                                     |
| :-------------: | :----------------------------------------------------------- |
|      FROM       | Dockerfile 除了注释第一行必须是 FROM ，基于哪个基础镜像构建。例如：FROM centos |
|       RUN       | RUN 后面跟一个具体的命令，类似于 Linux 命令行执行命令。例如： RUN npm run build 或者 RUN ['yum', 'install', 'nginx'] |
|       ADD       | 拷贝本机文件或者远程文件到镜像内，如果是 URL 或压缩包会自动下载或自动解压 |
|      COPY       | 拷贝本机文件到镜像内【类似 ADD】                             |
|      USER       | 指定容器启动的用户，即为 RUN、CMD、ENTRYPOINT 执行命令指定运行用户 |
|   ENTRYPOINT    | 容器启动时执行的 shell 命令。ENTRYPOINT /bin/bash -c 'start.sh'  或 ENTRYPOINT [’/bin/bash‘, '-c', 'start.sh'] |
|       CMD       | 容器启动时要执行的 shell 命令。CMD bin/bash -c 'start.sh' 或 CMD [’/bin/bash‘, '-c', 'start.sh']。CMD 与 ENTRYPOINT 区别：CMD 只有最后一个命令会生效，ENTRYPOINT 可以追加命令 |
|       ENV       | 构建的时候设置的环境变量，格式为 key=value                   |
|       ARG       | 定义外部变量，构建镜像时可以使用 build-arg= 的格式传递参数用于构建 |
|     EXPOSE      | 指定容器对外暴露的端口，可以在这里指定，也可以在 docker run 时通过 -p 指定，两者意思是一样的 |
|     WORKDIR     | 为 RUN、CMD、ENTRYPOINT、COPY 和 ADD 命令设置工作目录，也就是指定镜像工作目录 |
|     VOLUME      | 容器数据卷，可以在这里指定，也可以在 docker run 时通过 -v 指定，两者意思是一样的 |

通过一个实例来了解上述命令：

```shell
FROM centos:7
COPY nginx.repo /etc/yum.repos.d/nginx.repo
RUN yum install -y nginx
EXPOSE 80
ENV HOST=mynginx
CMD nginx
```

- 第一行表示基于 centos:7 这个镜像来构建自定义镜像。注意，每个 Dockerfile 的第一行除了注释都必须以 FROM 开头。
- 第二行表示拷贝本地文件 nginx.repo 文件到容器内的 /etc/yum.repos.d 目录下。这里拷贝 nginx.repo 文件是为了添加 nginx 的安装源。
- 第三行表示在容器内运行 yum install -y nginx 命令，安装 nginx 服务到容器内，执行完第三行命令，容器内的 nginx 已经安装完成。
- 第四行声明容器内业务（nginx）使用 80 端口对外提供服务。
- 第五行定义构建的时的环境变量 HOST=mynginx，容器启动后可以获取到环境变量 HOST 的值为 mynginx。
- 第六行设置了容器的启动命令为 nginx



#### Dockerfile 构建镜像，并发布到 DockerHub

接下来实战一下，怎么利用 Dockerfile 构建一个镜像，并发布到 DockerHub 镜像仓库



**Dockerfile 构建镜像**

首先，创建一个 Dockerfile 文件【注意大小写】

```shell
touch Dockerfile
```

接着，`vim DockerFile` 进入 Dockerfile 文件进行编辑

```shell
FROM centos                      # 基于 centos 创建一个镜像

RUN yum install -y vim           # 为这个镜像安装 vim 工具

CMD echo '------end--------'     # 输出 '------end--------'

CMD /bin/bash                    # 打开 /bin/bash 终端
```

然后执行：

```shell
docker build -f Dockerfile文件路径 -t 镜像名:[tag版本] .

# 没有通过 -f 指定 Dockerfile 文件路径，默认使用当前目录下的 Dockerfile 文件
# -t 镜像名:版本号，例如 myceentos:1.0.0
# 注意，末尾一定要有 . 这个符号！！！！！！！！！！！！！！
```

制作完镜像，通过 `docker images` 查看有没有存在镜像

![](/imgs/img20.png)

出现刚刚制作的镜像，代表通过 Dockerfile 制作镜像成功，接下来就是将镜像推送到 DockerHub 仓库



**发布镜像到 DockerHub**

首先，要先到 [DockerHub](https://hub.docker.com/) 上注册一个一个账号。

<img src="/imgs/img21.png" style="zoom:50%;" />

这里要记住 Docker ID 与密码，后面登陆 DockerHub 需要用到

注册完成之后登陆到 DockerHub

```shell
docker login -u DockerId -p Password

# -u 后面跟刚刚注册的 Docker ID
# -p 后面跟密码
```

发布镜像到 DockerHub

```shell
docker push mycentos:1.0

# 使用 DockerId/镜像名:版本 这样的形式
# 其实，在使用 docker build 构建镜像的时候，最好是：docker build -t DockerID/镜像名:版本号 这种形式
# 因为，如果单纯使用 镜像名:版本号，那么可能会有重名冲突，但是 DockerID 是唯一的，就不会存在同名冲突
# 推送的时候，就可以 docker push DockerID/镜像名:版本号
```

发布完成后，进入DockerHub 自己的详情页，如果有刚刚推送上来的镜像，代表发布成功。



### 1-13、图形化工具

Docker 图形化界面管理工具，提供一个管理面板供使用者操作，减少命令操作，比较常见的两个：

- Portainer，轻量好用，具体使用可以参考：https://juejin.cn/post/6960831907999252511

- Rancher，功能齐全，但相对较大



## 2、Linux 常用命令

需要掌握一些常用的 Linux 命令。



### 2-1、pwd

查看当前所在目录路径。linux下所有的绝对路径都是从根目录"/"开始

```shell
pwd
```

- /root：是linux下root用户的根目录

- /home：是linux下其他用户的默认根目录 （例如：在linux上创建了一个bow用户，那么就会在/home下面生成一个bow目录作为bow用户的根目录）

- /etc：是linux下系统配置文件目录

- /tmp：临时文件目录，所有用户都可以用



### 2-2、ls

列出目录下文件

```shell
ls         # 查看当前目录下的文件

ls -l      # 查看当前目录下所有文件的详细信息

ls -a      # 查看当前目录下的文件(包含隐藏文件，例如以 . 开头的文件)

ls -la     # 查看当前目录下的文件(包含隐藏文件)的详细信息

ls /etc    # 查看目录 /etc 下的文件
```



### 2-3、cd

进入一个目录，目录路径可以是绝对路径(以/开始的路径都是绝对路径)，也可以是相对路径

```shell
cd /               # 进入系统根目录

cd local/          # 进入当前目录下的local目录

cd ./bin           # 进入当前目录下的bin目录

cd ..              # 进入当前目录的上一级目录

cd ../etc          # 进入和当前目录同级的etc目录

cd ~               # 进入当前用户的根目录（root 目录）
```



### 2-4、mkdir

创建一个目录，目录路径可以是绝对路径也可以是相对路径

```shell
mkdir data # 在当前目录下创建一个 data 目录

mkdir ./dir # 在当前目录下创建一个 dir 目录

mkdir /root/tmp  #在 /root 目录下创建一个 tmp 目录
```

mkdir 创建目录时，只有在目录的上级目录存在时，才会创建



### 2-5、touch

创建文件

```shell
touch test.txt          # 在当前目录下创建一个 test.txt 文件

touch /root/test.txt    # 在 /root 目录下创建一个 test.txt 文件
```



### 2-6、rm

删除命令

```shell
rm test.txt                 # 删除当前目录下的 test.txt 文件

rm -f test.txt              # 强制删除当前目录下的 test.txt 文件

rm -rf user                 # 删除当前目录下的 user 目录
```



### 2-7、echo

输出命令

```shell
echo hello                 # 打印 hello

echo $PATH                 # 打印环境变量 PATH 的值
```



### 2-8、> 与 >>

输出符号，将内容输出到文件中；>：会覆盖原文件；>>：追加到原文件末尾

```shell
echo hello > test.txt         # 将 hello 输出到当前目录下的 test.txt 文件
                              # 如果当前目录下没有 test.txt 文件会创建一个新文件，
                              # 如果当前目录下有 test.txt，则会删除原文件内容，写入 hello

echo hello >> test.txt        # 将 hello 追加到当前目录下的 test.txt 中，如果文件不存在会创建新文件
```



### 2-9、cat

查看文件内容

```shell
cat test.txt                  # 查看当前目录下 test.txt 的内容
cat /root/test.txt            #查看 /root 目录下 test.txt 文件内容
```



### 2-10、vi 与 vim

这两个都是编辑文件命令。vi 与 vim 命令的使用方法基本一致。个人比较习惯使用 vim。

基本使用：

1. 首先，在 /home 目录下有 test.txt 文件

   ```shell
   vim /home/test.txt                # 打开 /home 目录下的 test.txt 文件
   ```

2. 通过 vim 打开文件，是不能直接编辑的，此时只能查看文件内容。输入 i 进入编辑模式

    <img src="/imgs/img14.png"  />

3. 编辑完之后，需要保存退出；首先，Esc 退出编辑模式，然后组合键 `shift + :` ，出现如下

    ![](/imgs/img15.png)

   接下来，可以输入如下某个命令：

   ```shell
   :w             # 保存文件
   :q             # 退出 vim 命令
   :wq            # 保存并退出
   :w!            # 强制保存
   :q!            # 强制退出但不保存
   :wq!           # 强制保存并退出
   ```




## 3、Gitlab CI/CD 持续集成部署

下面，将使用 Gitlab CI/CD + Docker 持续集成部署前端项目



先将服务器的安全组配置一下，主要是入口方向的，这样外部才能访问到

![](/imgs/img16.png)



### 3-1、使用 Docker 安装 Gitlab

> 安装 Gitlab 至少得 `2核4G` 以上的配置，最好 `2核8G` 以上



Docker 安装 Gitlab 非常简单，只需要一条命令即可：

```shell
sudo docker run --detach \
  --hostname 111.121.55.114 \
  --publish 443:443 --publish 80:80 --publish 222:22 \
  --name gitlab \
  --restart always \
  -v /srv/gitlab/config:/etc/gitlab \
  -v /srv/gitlab/logs:/var/log/gitlab \
  -v /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

其实整个命令简单版就是 `ddocker run --name gitlab gitlab/gitlab-ce:latest`，去 gitlab/gitlab-ce:latest 下载 gitlab 镜像，并且以这个镜像为模板，启动一个名为 gitlab 的容器。

- sudo：用来提升权限，使用 root 权限进行操作
- hostname：主机的 hostname，可以是域名或者 ip 地址
- publish：将容器内的端口映射到主机，那么就可以通过 `主机hostname:端口` 访问到容器内部，这里 443 是 https 端口、80 是 http 端口、22 是 ssl 端口，但是主机 22 端口被占用，所以映射到 222
- name：给启动的容器命名
- restart：重启的方式，always 表示自动重启。除了 always，还有其他多种重启策略，具体可参考上面的 [启动容器](###启动容器) 章节
- v：数据卷映射，将容器的目录到主机目录，实现数据持久化和共享



等到 Gitlab 安装完，并启动好，那么就可以通过配置的 hostname 进行访问，例如这里配置的是 111.121.55.114，浏览器打开这个 ip 地址，如果出现以下界面，则说明 Gitlab 安装成功

![](/imgs/img17.png)



第一次进入这个页面，需要设置 root 管理员密码，这个 root 就是 Gitlab 的超级管理员，看到 GitLab 的所有信息，如用户、项目、流水线等。

设置完 root 密码，既可以登录进入到 Gitlab 主界面了。接下来就通过新建一个项目，推送到 Gitlab。



### 3-2、推送项目到 Gitlab

首先，在 Gitlab 上新建一个项目仓库：

![](/imgs/img18.png)



然后，通过 vue-cli 创建一个项目：

```shell
vue create gitlab-cicd
```

创建完成之后，将这个项目推送到 gitlab 仓库

```shell
cd hello-cicd
git init
git add .
git commit -m "init"
git remote add origin http://111.121.55.114/guanwei/gitlab-cicd.git
git push -u origin master
```

如果推送成功，说明 Gitlab 可以正常使用。



### 3-3、解决 git clone 的地址与 IP 不同的问题

当将项目传上去的时候，你换了一台电脑，或者别人要去进行 git clone 将项目拉到本地的时候，会发现，无论是 http 方式还是 ssl 方式，那么地址与自己本机服务器的 IP 地址对应不上，没办法 clone 到本地。解决：

启动 gitlab 的时候是：

```shell
sudo docker run --detach \
  --hostname 111.121.55.114 \
  --publish 443:443 --publish 80:80 --publish 222:22 \
  --name gitlab \
  --restart always \
  -v /srv/gitlab/config:/etc/gitlab \
  -v /srv/gitlab/logs:/var/log/gitlab \
  -v /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

 那么就进入到 `/etc/gitlab/gitlab.rb` 这个文件编辑：

```shell
docker exec -it 容器id /bin/bash        # 进入容器

vim etc/gitlab/gitlab.rb               # 编辑 gitlab.rb 文件
```

只需要在这个文件内加上：

```shell
external_url '主机IP地址'
```

 <img src="/imgs/img22.png" style="zoom:50%;" />

最后重启容器：

```shell
exit                      # 退出当前容器

docker restart 容器id      # 重启容器
```

等到容器重启成功再进去 gitlab 面板，会发现 gitlab 的 http 和 ssl 的 clone 的 IP 地址已经换成服务器的 IP 地址，此时就可以正常 git clone 项目到本地了



### 3-3、安装 GitLab Runner

