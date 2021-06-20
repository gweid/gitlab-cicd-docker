# Gitlab CI/CD + Docker 前端持续集成部署

基于 gitlab ci/cd + docker 进行前端自动化部署。



环境说明：基于阿里云的服务器，配置为 2 核 4G，带宽为 3M，系统是 CentOS 7.7 64 位。



## 1、Docker

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

既然容器是基于镜像创建的，那么先 pull 一个 centos 镜像来进行容器的操作。

```shell
docker pull centos
```



#### 启动容器

启动容器的方式有两种：

1、如果这个容器已经存在，直接使用 `docker start` 启动，后面可以跟容器名或者容器id

```shell
docker start <container_id | container_name>
```



2、如果这个容器不存在，那么需要先根据镜像创建容器，然后启动

使用命令 `docker run`

```shell
docker run [可选参数] image   # 根据镜像 image 创建容器，并且启动容器

# 常用的可选参数
--name container_name          定义这个容器名字
-d                             代表后台运行【全称是 --detach】
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
```



docker run 的基本流程：

 <img src="/imgs/img7.png" style="zoom:50%;" />



#### 停止|关闭|重启容器

```shell
docker restart <container_id | container_name>         重启容器

docker stop <container_id | container_name>            正常停止容器运行

docker kill <container_id | container_name>            立即停止容器运行
```



#### 查看容器

```shell
docker ps           # 列出所有正在运行的容器

docker ps -a        # 列出所有容器【a：即 all 的简写】

docker ps -q        # 列出所有正在运行的容器id

docker ps -aq       # 列出所有的容器id
```



#### 进入容器





#### 删除容器

```shell
docker rm -f <container_id | container_name>   # 根据容器id 或者容器名删除容器

docker rm -f $(docker ps -aq)                  # 查找出所有容器id，然后删除所有容器
```











