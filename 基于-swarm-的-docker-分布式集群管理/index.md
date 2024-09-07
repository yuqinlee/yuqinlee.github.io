# 基于 Swarm 的 Docker 分布式集群管理


<!-- # 基于 Swarm 的 Docker 分布式集群管理 -->

## 1. 序

### 1.1 各虚拟机信息

> CentOS 7  
> VirtualBox

| 虚拟机名  | 虚拟磁盘大小 |       IP       | Host 映射 |
| :-------: | :----------: | :------------: | :-------: |
| myCent_00 |     100G     | 192.168.56.101 |  master   |
| myCent_01 |     50G      | 192.168.56.104 |  slave01  |
| myCent_02 |     50G      | 192.168.56.103 |  slave02  |

注：各虚拟机节点网卡设置如下

1. Nat：负责虚拟机与外网链接，保持默认
2. Host-Only：设置静态 ip 保证集群之间通信

确保每台虚拟机能 ping 通外网网络设置好后，更改主机名与映射，主机名修改后重启生效

```bash
vim /etc/hostname #修改主机名
vim /etc/hosts    # 修改映射
```

修改后如下：

![图 2](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650679575241.png)

### 1.2 ssh 免密登录

<font color='red'>注：免密登录配置的为各节点的 hadoop 用户</font>

以 master 节点为例，生成公钥、修改权限等命令如下

```bash
cd ~
mkdir .ssh
cd .ssh
# rm ./*                                # 删除当前目录下所有内容（首次配置时该目录下无内容，不用删除）
ssh-keygen -t rsa                       # 生成公钥
cat ./id_rsa.pub >> ./authorized_keys   # 添加至认证文件
chmod 700 ~/.ssh/                       # 赋予.ssh文件700权限,有的机器必须，但有的是可选
chmod 600 ./authorized_keys             # 赋予认证文件600权限
ssh localhost                           # 测试
```

退出登录执行 `exit` 即可，至此 master 节点上公钥、文件权限修改完成，要使得 master 能免密登录其他节点，将 master 的公钥传给其他节点，并让其追加至认证文件即可  
即：要免密登录谁，就将自己的公钥给他，让他写入认证文件 `authorized_keys` 中

发送 master 的公钥至 slave01 节点

```bash
cd ~.ssh
scp ./id_rsa.pub slave01:~/.ssh/id_master
```

登录至 slave01 ，将 master 的公钥追加到认证文件，其他节点类似

```bash
cd ~/.ssh
cat ./id_master >> ./authorized_keys
chmod 700 ~/.ssh/           # 赋予.ssh文件700权限,有的机器必须，但有的是可选
chmod 600 ./authorized_keys # 赋予认证文件600权限
```

### 1.3 CentOS 安装 Docker

#### 1.3.1 安装

1. 更新 yum

   ```bash
   sudo yum -y update
   ```

2. 添加源，选一个即可，考虑国内网络，这里使用的是阿里的源

   ```bash
   # 阿里仓库
   sudo yum-config-manager --add-repo http://mirrors.  aliyun.com/docker-ce/linux/centos/docker-ce.repo

   # 中央仓库
   # sudo yum-config-manager --add-repo http://download. docker.com/linux/centos/docker-ce.repo
   ```

   <font color='red'>注：这里没有添加源的话，yum 默认是没有 docker 的包安装的</font>

3. 查看可用版本

   ```bash
   yum list docker-ce --showduplicates | sort -r
   ```

4. 安装

   ```bash
   sudo yum -y install docker-ce-18.03.1.ce
   ```

   安装完成如图

   ![图 1](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650642608352.png)

#### 1.3.2 更改用户组

1. 查看用户组命令

   ```bash
   groups hadoop
   # hadoop : hadoop docker
   ```

2. 创建 docker 用户组，添加当前用户  
   在安装完 docker 后默认会创建 docker 用户组

   ```bash
   sudo groupadd docker            # 新建 docker 组
   sudo usermod -aG docker $USER   # 添加当前用户至docker组
   newgrp docker                   #更新
   ```

   执行更新命令后就不会再出现 `Got permission denied while trying to connect to the Docker daemon socket...` 报错了，或者修改用户组后退出当前终端并重新登录进行测试

3. docker 启动重启等相关命令  
   这里启动 docker 服务并设置开机自启即可，执行以下命令**前两条**

   ```bash
   # 启动 docker
   systemctl start docker

   # 设置开机自启
   systemctl enable docker

   # 设置开机自启并立即启动docker服务（相当于以上两条命令都执行）
   systemctl enable --now docker

   # 重启docker
   systemctl restart docker

   # 停止docker
   systemctl stop docker

   # 移除开机自启
   systemctl disable docker
   ```

4. 测试安装是否成功

   ```bash
   docker run --rm hello-world
   ```

   ![图 3](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650681556408.png)

   注：首次执行需要拉去镜像，会花一段时间

5. 在准备创建集群的各节点安装 docker，操作同上

## 2. Docker 网络基础

### 2.1 基本原理

1. 每启动一个 docker，docker 就会给容器分配一个 ip，只要安装了 docker，就会有一个 docker0 的网卡，这里的网卡采用的是**桥接模式**，使用的技术是 evth-pair 技术
2. 当启动一个容器时，本地就会多一个网卡（**与容器的网卡成对**）
   ![图 2](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650962858622.png)
3. 这些由于容器而产生的**成对**的网卡就是 evth-pair 技术：一对虚拟设备接口，一端连着协议，一端彼此连接
4. 正因为有这个特性，evth-pair 充当一个桥梁，连接各种虚拟设备
5. 因此容器之间也是可以 ping 通的
6. 所有容器在没有指定网络的情况下都是 docker0 进行路由的，docker 会给容器分配默认的可用 ip
   ![图 3](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650963563673.png)  
   ![图 4](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650963614223.png)

<font color='red'>--link 有待学习</font>

### 2.2 自定义网络

Docker 有以下四种网络模式

1. bridge：桥接（默认方式）
2. none：不配置网络
3. host：和宿主机共享网络
4. container：容器内网络连通（用的少）

```bash
docker run -it -P --name mycentos centos
# docker run -it -P --name mycentos --net bridge centos
```

上述两个命令等价，因为默认创建就是使用的 bridge

docker0 特点：默认的，但是域名不能访问，使用 `--link` 可以打通连接

<font color='red'>--link 有待学习</font>

### 2.3 常用命令

[Docker network 子命令](https://blog.csdn.net/gezhonglei2007/article/details/51627821)如下：

```bash
docker network create       # 创建网络
docker network connect      # 连接
docker network ls           # 列出网络
docker network rm           # 删除网络
docker network disconnect   # 断开连接
docker network inspect      # 列出某网络详细信息
```

## 3. Docker 数据卷

### 3.1 基本挂载命令

使用 `-v` 命令挂载

```bash
docker run -it -v 主机目录:容器内目录
```

`-v` 后进行的目录映射，类似于端口映射 `-p 主机端口:容器内端口`

```bash
docker run -it -v /home/hadoop/utest/test_docker_volume/:/home centos /bin/bash
```

这样，当在容器内部指定的目录内创建文件、文件夹等操作后会与主机对应的目录同步；同样的，在主机相应的目录中的操作也会同步到容器的响应目录中，这样当容器关闭后数据就能保存下来

当需要挂载多个目录时可使用多个 `-v` 参数，例如

```bash
docker run -d -p 3310:3306 -v /home/hadoop/utest/test_docker_volume/mysql/conf:/etc/mysql/conf.d -v /home/hadoop/utest/test_docker_volume/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql
```

以上 `-e` 是指定环境，这里指定了密码

### 3.2 具名挂载与匿名挂载

以上挂载时，指定了主机内对应的映射，如果不指定主机的目录，只在 `-v` 后声明容器内要被映射的目录，那就是**匿名挂载**，例如

```bash
docker run -d -P --name nginx01 -v /etc/nginx nginx
```

> -P 是随机端口映射

上述挂载就至指定了容器内的目录

使用

```bash
docker volume ls
```

查看所有卷

```txt
(base) [hadoop@master test_docker_volume]$ docker volume ls
DRIVER              VOLUME NAME
local               137b54d6bf243bdd8d87e89f991597ca283780e1396d9fc7ec32623362cbdb8e
local               441ffaf533e1a0c851f59690a287fc28aa85df60a7ad352c5d98cdc5680f2ddf
local               b63721368502cc85d467e02fa2a7189f25c8a509553c23155e81947402ecf2a2
local               portainer_data
```

上述前三条就是匿名挂载，如果在 `-v` 后指定卷名，那就是具名挂载，例如

```bash
docker run -it -v juming:/home centos /bin/bash
```

使用 `docker volume inspect 卷名` 查看该卷

```txt
(base) [hadoop@master run]$ docker volume inspect juming
[
    {
        "CreatedAt": "2022-04-26T15:04:52+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming/_data",
        "Name": "juming",
        "Options": null,
        "Scope": "local"
    }
]
(base) [hadoop@master run]$
```

可以看到该卷在主机上的挂载点为 `/var/lib/docker/volumes/juming/_data`

所有的 docker 容器卷，在没有指定目录的情况下都是在 `/var/lib/docker/volumes/xxx/_data` 处，其中 `xxx` 是卷名，`_data` 下放数据

通过具名挂载可以很方便找到一个卷，大多数时候都使用**具名挂载**

### 3.3 控制权限

```bash
docker run -it -v juming:/home:ro centos /bin/bash
docker run -it -v juming:/home:rw centos /bin/bash
```

上述添加了 `ro`、`rw` 分别表示只读与可读写，一旦设定了只读，那说明这个目录**只能通过宿主机来进行改变**，容器内部是无法操作的

### 3.4 dockerfile 与数据卷

1. 创建 `dockerfile`

   ```dockerfile
   FROM centos
   VOLUME ["volume01","volume02"]
   CMD echo "--------- end ---------"
   CMD /bin/bash
   ```

   这里的每个命令就是镜像的一层，这里通过生成镜像的方式使得在容器创建时候就挂载不需要再指定参数

2. 构建

   ```bash
   docker build -f dockerfile1 -t uchin_centos .
   ```

   上述 `-f` 指定 dockerfile 文件，`-t` 即 target，指定生成的镜像名，最后指定生成后的路径

   ```txt
   (base) [hadoop@master test_docker_volume]$ docker build -f dockerfile1 -t uchin_centos .
   Sending build context to Docker daemon  2.048kB
   Step 1/4 : FROM centos
    ---> 5d0da3dc9764
   Step 2/4 : VOLUME ["volume01","volume02"]
    ---> Running in 8ce46c7e7766
   Removing intermediate container 8ce46c7e7766
    ---> 3019041d4948
   Step 3/4 : CMD echo "--------- end ---------"
    ---> Running in b4647234f83d
   Removing intermediate container b4647234f83d
    ---> b61a9aa5313f
   Step 4/4 : CMD /bin/bash
    ---> Running in 8669d24b2112
   Removing intermediate container 8669d24b2112
    ---> 8ea7424ac06e
   Successfully built 8ea7424ac06e
   Successfully tagged uchin_centos:latest

   (base) [hadoop@master test_docker_volume]$ docker image ls | grep uchin_centos
   uchin_centos          latest              8ea7424ac06e        2 minutes ago       231MB
   ```

3. 测试

   ```bash
   docker run -it uchin_centos /bin/bash
   ```

   进入容器后发现指定的数据卷就被自动挂载了

   ```txt
   [root@5a1789b7990c /]# ls -l | grep volume
   drwxr-xr-x.   2 root root   6 Apr 26 07:36 volume01
   drwxr-xr-x.   2 root root   6 Apr 26 07:36 volume02
   ```

   并且这个卷和外部是同步的，但是在 dockerfile 中使用的是**匿名挂载**的方式，可以使用 `inspect` 方式查询其具体位置

   这种方式使用十分多，通常我们都会自己构建镜像，假设构建镜像时没有挂载卷，就手动使用 `-v` 方式挂载

### 3.5 数据卷容器

![图 1](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650959416556.png)

创建父容器

```bash
docker run -it --name docker01 uchin_centos /bin/bash
```

在创建一个，使用 `--volumes-from` 参数来实现数据卷的共享

```bash
docker run -it --name docker02 --volumes-from docker01 uchin_centos /bin/bash
```

这样挂载的数据卷就可以实现多容器的共享了，当然这里不使用相同的镜像也可以

<font color='red'>有待继续探究</font>

## 4. Docker Dokcerfile

### 4.1 前言

Dockerfile 是一个用来**构建镜像**的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

当获取的镜像不满足我们使用的需求时，往往需要在创建容器后，自己再在容器中执行命令去安装软件、配置文件等。当多次需要使用到相同环境时这样不方便。

可以使用`docker commit`命令将已经配置好的容器提交为镜像类似于`git`的提交，这是一种手动保存镜像的方式，这里的 Dockerfile 就是一种更加自动化的方式去进行镜像的构建

### 4.2 Dokcerfile 构建

使用 Dockerfile 构建一个镜像流程

1. 创建一个目录并进入其中（以下均在该目录下进行）
2. 新建一个名为 `Dockerfile` 文件，并定义指令（见下节）
3. 在 Dockerfile 文件的存放目录下，执行构建动作 `docker build -t mycentos:v1 .`命令构建  
   其中 `-t` 即 target，参数指定 `镜像名称:镜像标签`； `.` 代表本次执行的**上下文路径**

   > 上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。
   >
   > 解析：由于 docker 的运行模式是 C/S。我们本机是 C，docker 引擎是 S。实际的构建过程是在 docker 引擎下完成的，所以这个时候无法用到我们本机的文件。这就需要把我们本机的指定目录下的文件一起打包提供给 docker 引擎使用。
   >
   > 如果未说明最后一个参数，那么默认上下文路径就是 Dockerfile 所在的位置。
   >
   > 注意：上下文路径下不要放无用的文件，因为会一起打包发送给 docker 引擎，如果文件过多会造成过程缓慢。

### 4.3 Dockerfile 指令

以下为一个 Dockerfile 文件示例

```Dockerfile
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

- `FROM`：定制的镜像都是基于 `FROM` 的镜像，这里的 python:3.7-alpine 就是定制需要的基础镜像。后续的操作都是基于 python:3.7-alpine

- `RUN`：用于执行后面跟着的命令行命令。有以下两种格式：

  ```bash
  RUN <命令行命令>

  RUN ["可执行文件", "参数1", "参数2"]
  # 例如：
  # RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
  ```

  注：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大，尽量使用 `&&` 符号将多个命令写成一行，这样就只会创建一层镜像，例如  
   `RUN yum -y install wget \
  && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
  && tar -xvf redis.tar.gz`

更多指令参考<https://www.runoob.com/docker/docker-dockerfile.html>

## 5. Docker compose

### 5.1 前言

原先是使用 dockerfile 进行镜像创建，使用`docker build`、`docker run` 等命令进行**手动的**操作**单个**容器；这样也不便实现相互之间的依赖关系，于是就有了 docker compose
这一容器编排管理技术

使用 docker compose 能高效管理容器：  
① 定义多个运行容器  
② 实现批量容器编排

使用步骤:

1. `dockerfile`：确保项目在任何地方都可以运行
2. `docker-compose.yml`：定义服务
3. `docker-compose up`：运行

### 5.2 安装

docker compose 不是 docker 所自带，需要额外自行安装

~~注：在使用 swarm 集群时，使用 `docker stack deploy` 指定 compose 文件进行构建时没有安装 docker-compose 也可以~~

1. 下载

   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   # sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
   ```

2. 赋权限

   ```bash
   sudo chmod +x /usr/local/bin/docker-compose
   ```

### 5.3 测试

注：测试来源于 docker 官网

1. 创测试目录

   ```bash
   mkdir composetest
   cd composetest
   ```

2. 创建 `app.py`

   ```python
   import time

   import redis
   from flask import Flask

   app = Flask(__name__)
   cache = redis.Redis(host='redis', port=6379)

   def get_hit_count():
       retries = 5
       while True:
           try:
               return cache.incr('hits')
           except redis.exceptions.ConnectionError as exc:
               if retries == 0:
                   raise exc
               retries -= 1
               time.sleep(0.5)

   @app.route('/')
   def hello():
       count = get_hit_count()
       return 'Hello World! I have been seen {} times.\n'.format(count)

   ```

3. `Dockerfile`

   ```dockerfile
   FROM python:3.7-alpine
   WORKDIR /code
   ENV FLASK_APP=app.py
   ENV FLASK_RUN_HOST=0.0.0.0
   RUN apk add --no-cache gcc musl-dev linux-headers
   COPY requirements.txt requirements.txt
   RUN pip install -r requirements.txt
   EXPOSE 5000
   COPY . .
   CMD ["flask", "run"]
   ```

4. `docker-compose.yml` 文件

   ```yaml
   version: "3.9"
   services:
     web:
       build: .
       ports:
         - "8000:5000"
     redis:
       image: "redis:alpine"
   ```

5. 运行

    ```bash
    docker-compose up
    ```

创建步骤：

1. 应用 `app.py`
2. Dockerfile 使应用能打包为镜像
3. docker-compose 文件（定义整个服务，需要的环境，web、redis）提供完整的上线服务
4. 启动 compose 项目（docker-compose up）

<font color='red'>注：这里的 `web` 服务所使用的镜像是使用 Dockerfile 进行 `build` 的；而 `redis` 服务是直接使用的镜像，两种方式都可，但是在 swarm 集群中是**不能**通过 `build` 来进行镜像的创建的</font>

### 5.4 compose 配置规则

详细配置可参考[官网](https://docs.docker.com/compose/compose-file/#compose-file-and-examples)

一般使用的 compose 文件包含以下三大部分：

```yaml
# 1. 版本
version: "3.9"

# 2. 定义服务
services:
  服务1（web）:
    服务配置（image、build、network）:
  服务2（redis）:
  服务3:

# 3. 其他配置（网络、卷、全局规则）
volumes:
networks:
configs:
```

有时其他配置可不需要

<font color='red'>注：集群中使用 compose 配置进行服务的创建有所不同，具体参考下文</font>

## 6. Swarm 集群部署

进行 swarm 集群搭建前请确保

1. 各节点之间能通讯正常
2. 能通过相互免密登录
3. docker 均已安装其能正常使用

<font color='red'>**注：以下操作都在各节点的 hadoop 用户下操作，集群部署使用的用户名保持各节点一致**</font>

### 6.1 集群创建

如有多块网卡，需指定集群中使用的网卡所对应的 ip，例如，当前有多块网卡，如下图

![图 1](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650677417546.png)

这里将 `enp0s9` 作为集群通信所使用的，那就用其所对应 ip `192.168.56.101`

选择一个节点作为 manager 节点登录，执行初始化命令

```bash
docker swarm init --autolock=false  --advertise-addr 192.168.56.101 --listen-addr 192.168.56.101:2377
```

执行后如下

![图 2](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650645402766.png)

初始化后，当前所在的这个节点就是 manager，接下来可添加 worker,复制以上 Join token 内容，即 `docker swarm join --token SWMTKN-1-5tquxa6ztsbln9l4q0gr5tq6o7wowj16oncu6vg9t26mp9shj7-drdzg6xreehoz3gpm6h48hgpk 192.168.56.101:2377`
，登录 slave01 节点，执行该命令，即可将 slave01 节点以 worker 方式加入 swarm 集群

> **在 swarm 中有两种可用的不同的 token，一种是用作 worker 角色，另一个是用作 manager 角色**

由于集群中允许有多个 manager 与 worker 的存在，因此**在 join 时根据所使用的 token 的不同，加入的方式也不同**，如果使用的是 worker 的 token，那加入后就是 worker，使用的 manager 的 token，加入后就是 manager

常用涉及 token 的命令如下

```bash
docker swarm join-token worker          # 查看加入woker的命令
docker swarm join-token manager         # 查看加入manager的命令
docker swarm join-token --rotate worker # 重置woker的Token
docker swarm join-token -q worker       # 仅打印Token
```

### 6.2 基于 Portainer 的可视化

参考：[可视化工具 Portainer](https://support.websoft9.com/docs/docker/zh/solution-portainer.html#%E7%99%BB%E5%BD%95-portainer)

Swarm 集群部署完成后可以通过各命令操作，当然也可以通过 Portainer 来进行 Swarm 集群的可视化管理

Portainer 是一款轻量级的开源应用，它提供了图形化界面，用于方便地管理 Docker 环境，包括单机环境和集群环境

如果部署了包含 Portainer 的 Docker 环境，可直接登录使用，否则，先安装 Portainer：

```bash
docker volume create portainer_data # 首次创建
docker run -d -p 9000:9000 --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

以上是使用独立容器启动 Portainer，也可以通过 Docker Compose 进行启动，docker-compose.yml 文件内容如下:

```yaml
version: "3"
volumes:
  portainer_data:

services:
  portainer:
    image: portainer/portainer
    ports:
      - "9000:9000"
    command: -H unix:///var/run/docker.sock
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
```

文件编写完毕后通过 `docker-compose up -d` 命令可启动容器（可通过 `-f` 选项指定文件）

<font color='red'>注：这里使用 `docker run` 或使用 `docker-compose` 方式启动的容器都是在一个节点上，**不是** swarm 集群中运行，要让其在 swarm 集群中可使用 `docker stack deploy`命令</font>

容器启动后，访问 `https://192.168.56.101:9000` 即可进入 webUI 界面，如图（`192.168.56.101` 为**运行该容器**所在节点的 ip）

![图 4](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650685819089.png)

注：首次进入需设置用户名与密码

在 portainer 页面可创建、删除容器、查看 swarm 信息等

查看 swarm 信息
![图 5](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650686138442.png)

进入容器
![图 6](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650686499864.png)

容器控制界面
![图 7](%E5%9F%BA%E4%BA%8E%20Swarm%20%E7%9A%84%20Docker%20%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E7%AE%A1%E7%90%86.assets/pic_1650686591366.png)

## 7. Swarm 集群使用

### 7.1 网络模式

参考 [docker swarm 网络模式](https://www.jianshu.com/p/cacdd5ff0f14)

- swarm：
- overlay：
- ingress：特殊的 Overlay 网络，具有负载均衡功能

### 7.2 Docker service

> 服务是一组运行于集群**不同结点中可复制的容器的集合**。通过服务发现机制，**使用该服务的客户端可以透明的访问这些容器**。这些容器的分布、升级、故障管理有集群统一管理。一般地，集群内部服务容器地选择由 动态DNS 实现

- `docker run`：不具有扩缩容的功能
- `docker service`：启动服务，具有**扩缩容功能，滚动更新**等功能

#### 4.1.1 创建服务

```bash
docker service create  -p 8888:80 --name my-nginx nginx
```

创建的服务从集群中**任意主机 ip 都可访问**

![图 5](./基于%20Swarm%20的%20Docker%20分布式集群管理.assets/pic_1650980646777.png)  
上述访问的是 `slave01` 的 ip，但容器运行在 `master` 节点上

![图 6](./基于%20Swarm%20的%20Docker%20分布式集群管理.assets//pic_1650980702383.png)

#### 4.1.2 查看服务（在 manager 节点上执行）

```bash
docker service ls
docker service inspect my-nginx # 查看该服务详细信息
```

```txt
(base) [hadoop@master composetest]$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
vah2ryttads6        my-nginx            replicated          1/1                 nginx:latest        *:8888->80/tcp
```

可以看到在服务中有`REPLICAS`（副本）的概念，其代表这一服务有多少分副本

#### 4.1.3 更新服务

```bash
docker service update --replicas 3 my-nginx
```

上述命令将 `my-nginx` 服务副本更新为 3，即集群中有 3 个运行 nginx 的容器，当选择的节点上没有该镜像时会自动下载；如指定 replicas 为 1 就能回到开始状态，达到动态缩容的效果

注：**只要是一个服务，那么在集群中的任意一个节点都可以访问**

除了上述的 `update` 方式，还可以使用 `scale` 命令扩缩容

```bash
docker service scale my-nginx=2
```

#### 4.1.4 删除服务命令

```bash
docker service rm my-nginx
```

![图 7](./基于%20Swarm%20的%20Docker%20分布式集群管理.assets/pic_1650982073596.png)

![图 8](./基于%20Swarm%20的%20Docker%20分布式集群管理.assets/pic_1650982138022.png)

命令 -> 管理节点 -> api -> 调度 -> 工作节点（创建容器，即 Task 任务）

![图 9](./基于%20Swarm%20的%20Docker%20分布式集群管理.assets/pic_1650982654368.png)

### 7.3 Docker stack

参考:<https://yeasy.gitbook.io/docker_practice/swarm_mode/stack>

- docker-compose:单机部署
- docker stack：集群上部署

> 使用 `docker-compose.yml` 来一次配置、启动多个容器，在 Swarm 集群中也可以使用 `compose` 文件 （`docker-compose.yml`） 来配置、启动多个服务。
>
> `docker service create` 一次只能部署一个服务，使用 `docker-compose.yml` 我们可以一次启动多个关联的服务。

命令如下，其中 `-c` 指定 compose 文件名

```bash
docker stack deploy -c docker-compose.yml wordpress
```

```bash
docker service create --mount type=volume,src=volume,dst=/data --name uchin_v_test busybox ping
```

[Docker Swarm（四）Volume 数据（挂载）持久化](https://www.cnblogs.com/caoweixiong/p/12381838.html)

[创建数据卷](http://m.blog.chinaunix.net/uid-25530360-id-5825518.html)

---

创建 overlay

```bash
docker network create kafka --driver overlay --subnet 172.30.0.0/16 --gateway 172.30.0.1
docker network create kafka --driver overlay  --attachable --subnet 172.30.0.0/16 --gateway 172.30.0.1
```

创建数据卷

参考：<https://blog.csdn.net/even160941/article/details/99324273>

```bash
docker volume create --name volume_kafka
```

```bash
docker service create --name uchin_test --network kafka --mount type=volume,src=volume_kafka,dst=/home --replicas 2 centos
```

```bash
docker service create --network kafka -p 8888:80 --name my-nginx --mount type=volume,src=volume_kafka,dst=/home  nginx
```

[docker swarm 模式下，如何固定容器的 ip？](https://ask.csdn.net/questions/770590)

docker run -it --network kafka -v /home/hadoop/workspace/zkk/zk_volumes/app/:/home --name myCL python /bin/bash
docker run -it --network kafka -v /home/hadoop/workspace/zkk/zk_volumes/app/:/home --name myConsumer continuumio/anaconda3 /bin/bash

## 8. [Swarm 工作原理](https://holynull.gitbooks.io/docker-swarm/content/swarmmo-shi-gong-zuo-yuan-li.html)

### 8.1 节点工作原理

参考：[swarm 节点工作原理](https://holynull.gitbooks.io/docker-swarm/content/swarmmo-shi-gong-zuo-yuan-li/jie-dian-gong-zuo-yuan-li.html)

Swarm 集群中的节点分为两类：managers 和 workers，集群中可有多个 manager 和 worker。Manager 通过 Raft 协议来使 Swarm 的整体维持在一个一致的内部状态上，service 则运行在这个一致的内部状态之上。

![swarm-diagram](././基于%20Swarm%20的%20Docker%20分布式集群管理.assets/swarm-diagram.png)

#### 8.1.1 Manager 节点

Manager 节点用来处理集群管理任务：

- 维护集群状态
- 调度 service
- 提供 swarm 模式 HTTP API 端点

推荐使 Swarm 的节点为**奇数**个，来实现高可用的要求。当采用多个 manager 节点时，我们就可以在不停止集群运行的情况下恢复停止运行的 manager 节点。

- 3 个 manager 节点最大容错允许 1 个 manager 不可用
- 5 个 manager 节点最大容错允许 2 个 manager 不可用。
- N 个 manager 节点最大容错允许(N-1)/2 个 manager 节点不可用。
- 建议最大 manager 节点数为 7 个。

#### 8.1.2 Worker 节点

Worker 节点也是 Docker Engine 的实例，目的是用来运行 Container。Worker 节点不会像 Manager 节点那样提供集群的管理、任务调度和 API。

**可以创建仅有一个 manager 节点的 Swarm，但是不能在没有 manager 节点的前提下，创建 Worker 节点。在默认情况下，manager 节点同时也是一个 worker 节点。** 在一个 manager 节点的集群中，执行``docker service create`命令，调度器会将所有的 task 调度在本地 Docker Engine 上执行。

要阻止调度器将 task 分配到 manager 节点上执行，就需要将 manager 节点的可用性状态设置成 DRAIN。调度器会停止 DRAIN 节点上的 task，转移到其他状态为 ACTIVE 的节点上继续运行停止的 task。并且调度器不会再将新的 task 分配给 DRAIN 状态的节点。

#### 8.1.3 改变节点角色

我们可以通过``docker node promote`来将一个 worker 节点修改成一个 manager 节点。例如，某些维护需要，我们将停止一个 manager 节点，这时我们可以将某一个正在运行的 worker 节点变成 manager 节点，来代替停止的那个 manager 节点。当然，同样也可将一个 manager 节点改成一个 worker 节点。

### 8.2 Service 工作原理

参考[Service 工作原理](https://holynull.gitbooks.io/docker-swarm/content/swarmmo-shi-gong-zuo-yuan-li/servicegong-zuo-yuan-li.html)

为了部署一个应用的镜像到 Swarm 模式的 Docker Engine 中，我们需要创建一个 service。通常 service 是拥有大型应用系统上下文信息的微服务镜像。例如，一个服务可能包含一个 HTTP 服务器、一个数据库、或者其他的软件，我们需要这些软件运行在一个分布式环境中。

当创建 service 时，我们需要指定用什么镜像运行 container，以及 container 内部运行什么样的命令。我们还需要定义以下内容：

- service 向 Swarm 以外暴露的可用端口
- 一个 overlay network，用来支持 service 之间相互通信
- CPU 和内存的限制和预设
- 滚动更新的策略
- 镜像在 Swarm 中运行的副本数

#### 8.2.1 Service，Tasks and Containers

当我们部署一个 service 到 swarm 时，manager 节点将接受我们对 service 定义，并作为 service 的理想状态。Manager 节点在 swarm 节点之间调度 service 的副本。不同节点上运行的 task 是相互独立的。

例如，我们设想在 3 个 HTTP 监听实例之间实现负载均衡。下图展示了拥有 3 个副本的 HTTP 监听器。每一个监听器实例就是一个 swarm 中的 task。

![alt](././基于%20Swarm%20的%20Docker%20分布式集群管理.assets/services-diagram.png)

一个 container 是一个被隔离的进程。在 swarm 模式的模型中，每一个 task 运行一个 container。**task 就像调度器放置 container 使用的一个“槽”**。一旦 container 运行起来，调度器就能意识到 task 是在运行状态。如果 container 不可用或者被终止，则 task 也将被终止。

#### 8.2.2 task 和调度

task 是 swarm 调度的原子单元。当创建和更新 service 时，我们声明了 service 的理想状态，协调器通过调度 task 来实现这个理想状态。例如，我们定义了一个 service，要求协调器在任何时候都能保证有 3 个 HTTP 监听器在运行。协调器创建 3 个 task 作为回应。每一个 task 就像一个“槽”，调度器将产生 3 个 container 放在“槽”中。container 是 task 的一个实例。当一个 HTTP 监听器不可用或者崩溃，协调器将创建一个新的 task 副本，并放入一个新的 container。

task 的状态变化采用一种单向机制。贯穿 task 生命周期的状态有：已分配状态、已准备状态、正在运行状态，等等。如果 task 运行失败了，协调器会移除这个 task 并停止它的 container，然后创建新的 task 替代它，以保证 service 定义的理想状态。

Swarm 模式背后的逻辑其实是一种通用的调度器和协调器逻辑。service 和 task 被抽象设计成并不需要知道他们实现的 container 是什么样的。假设，我们可以实现其他类型的 task，例如虚拟机 task 或者非容器化的进程 task。调度器和协调器并不需要知道 task 的类型。然而，当前版本的 Docker 只支持容器 task。

下图展示了 Swarm 模式如何接收 service 的创建请求，并调度 task 到 worker 节点。

![service-lifecycle](././基于%20Swarm%20的%20Docker%20分布式集群管理.assets/service-lifecycle.png)

#### 8.2.3 Pending services

由于配置的原因，某些 service 在当前 swarm 中没有节点可以执行它的 task。在这种情况下，service 会一直保持 pending 状态。下面的一些例子会造成 service 保持 pending 状态。

> 注意：如果有意要阻止 service 被发布，应该将 service 的 scale 设置为 0，而不是通过设置让 service 一直保持在 pending 状态上。

- 如果所有的节点都暂停了或者都处于 DRAIN 状态，创建 service 时就会一直处于 pending 状态，知道有节点变为可用状态。事实上，当一个节点可用时，所有的任务都会被分配到这个节点上，所以对于生产环境来说并不是一件好事。
- 可以事先给 service 划定一定的内存。如果没有一个节点具有所需要的内存空间量，service 会一直保持 pending 状态，直到一个节点变的可用。
- 可以强制 service 的放置约束，短时间内可能达不到约束条件。

以上说明需求和 task 的配置没有紧密的结合到 swarm 的当前状态。作为 Swarm 的管理员，生命 Swarm 的理想状态，manager 节点结合其他节点在 Swarm 中创建出这个状态。就不需要将管理的粒度细化到 task 级别。

#### 8.2.4 复制和全局 service

有两种 service 的部署模式，复制和全局。

设置 task 的数量来实现复制模式下的 service。例如，想要部署 3 个副本的 HTTP 服务，每个副本都提供相同的服务。

全局 service 是在所有节点上都有一个 task 的 service。不能指定 task 的数量。每次增加一个节点，协调器会创建一个 task，并有调度器分配到新节点上。全局 service 的最佳应用是部署监控代理，病毒扫描，以及其他希望在 swarm 中每个节点上都部署的 service。

下图展示了 3 个副本的 service 和一个全局 service 部署在 swarm 中的情况，黄色代表 3 个副本的 service，灰色代表全局 service：

![alt](././基于%20Swarm%20的%20Docker%20分布式集群管理.assets/replicated-vs-global.png)

### 8.3 安全(PKI)

参考：[安全](https://holynull.gitbooks.io/docker-swarm/content/swarmmo-shi-gong-zuo-yuan-li/an-quan-ff08-pki.html)

Docker 内置了 PKI，使保障发布容器化的业务流程系统的安全性变得很简单。Swarm 节点之间采用 TLS 来鉴权、授权和加密通信。

当我们运行`docker swarm init`命令时，Docker 指定当前节点为一个 manager 节点。默认情况下，manager 节点会生成一个新的根 CA 证书以及一对密钥。CA 证书和密钥将会被用在节点之间的通信上。也可以通过参数--external-ca 来指定其他 CA 证书。

Manager 节点会生成两个 token，一个用来添加 worker 节点，一个用来添加 manager 节点。每一个 token 包含了 CA 证书的 digest 和一个随机密钥。当节点加入到 Swarm 中时，加入的节点将 digest 发送给远端的 manager 节点，由 manager 节点来验证申请加入节点的根 CA 证书是否合法。远端 manager 节点通过随机密钥来验证申请加入的节点是否被核准加入。

每当节点加入到 Swarm 中后，manager 都会给节点发送一个证书。这个证书中包含了一个随机生成的节点 ID，用来标识这个证书通用名称下的节点，以及组织单元下的角色。在节点的当前 Swarm 生命周期中，节点 ID 是节点的安全加密的身份标识。

下图说明了 manager 节点和 worker 节点如何使用 TSL 1.2 进行通信加密的。

![alt](././基于%20Swarm%20的%20Docker%20分布式集群管理.assets/tls.png)

下面的例子展示了一个 worker 节点获得证书内容：

```bash
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            3b:1c:06:91:73:fb:16:ff:69:c3:f7:a2:fe:96:c1:73:e2:80:97:3b
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN=swarm-ca
        Validity
            Not Before: Aug 30 02:39:00 2016 GMT
            Not After : Nov 28 03:39:00 2016 GMT
        Subject: O=ec2adilxf4ngv7ev8fwsi61i7, OU=swarm-worker, CN=dw02poa4vqvzxi5c10gm4pq2g
...snip...
```

默认情况下，每个节点的证书 3 个月轮换一次。我们可以通过配置修改的这个间隔：docker swarm update --cert-expiry \<TIME PERIOD\>。最小的证书轮换时间为 1 小时。

#### 8.3.1 CA 证书轮换

如果 Swarm 的 CA 密钥或 Manager 节点受到危害，您可以轮换 Swarm 的根 CA 证书，以使所有节点都不再信任旧的根 CA 签名的证书。

运行命令`docker swarm ca --ratate`来生成新的 CA 证书和密钥。也可以通过参数`--ca-cert`和`--external-ca`来指定外部的根 CA 证书。

当运行了`docker swarm ca --rotate`命令后，会按顺序发生下面的事情：

1. Docker 会生成一个交叉签名（cross-signed）证书。即新证书是由旧的证书签署的。这个交叉签名证书将作为一个过渡性的证书。这是为了确保节点仍然能够信任旧的证书，也能使用新的 CA 证书验证签名。

2. 在 Docker 17.06 或者更高版本中，Docker 会通知所有节点立即更新 TLS 证书。根据 Swarm 中节点的数量多少，这个过程可能会花费几分钟时间。

   > 注意：如果 Swarm 中的节点上 Docker 的版本不一致，将会发生下面的情况：
   >
   > - 只有 manager 节点运行了 Docker 17.06 或者更高版本，并且作为 leader 时，才能通知到其他节点更新 TLS 证书。
   > - 只有 Docker 17.06 或者更高版本才会支持这个指令。 最好确保所有的 Swarm 节点上运行 Docker 17.06 或者更高版本。

3. 在所有的节点都更新了新 CA 证书签署的 TLS 证书后，Docker 将不在信任旧的证书，并通知所有节点仅信任新的 CA 证书。

   加入 Swarm 使用的 token 将发生变化，旧的 token 将不可用。

从这时起，所有新的由新根 CA 证书签署的节点证书将颁发给节点，并且完全不存在过度内容。

### 8.4 Task 的状态

Service 是应用运行的理想状态的描述，task 在这个理想状态下完成工作。工作按照下面的流程在 Swarm 节点之间被调度：

1. 使用 CLI 运行命令`docker service create`，或者使用 UCP web 界面。

2. 请求传递给 manager 节点。

3. manager 节点在特定的节点调度 service 的运行。

4. 每一个 service 可以由多个 task 来执行。

5. 每一个 task 都有一个生命周期，生命周期的状态包括：`NEW`，`PENDING`和`COMPLETE`等等。

Task 是一次执行单元。当 task 停止，就不会再被执行，除非一个新的 task 会取代它。

在 task 执行完成或者失败之前，task 会通过一系列的状态变化。task 由`NEW`状态初始化。task 的状态变化过程是不可逆的。例如，一个 task 是永远不会从`COMPLETE`状态变回`RUNNING`状态的。

Task 的状态如下表：

| 状态      | 描述                              |
| --------- | --------------------------------- |
| NEW       | 初始化状态                        |
| PENDING   | 资源分配了任务时的状态            |
| ASSIGNED  | task 被分配到节点后的状态         |
| ACCEPTED  | task 被 worker 节点接受后的状态。 |
| PREPARING | Docker 正在准备 task              |
| STARTING  | Docker 启动 task                  |
| RUNNING   | 正在运行中的状态                  |
| COMPLETE  | task 已经存在，并且没有错误码     |
| FAILED    | task 已经存在，但是有错误码出现   |
| SHUTDOWN  | Docker 被请求关闭 task            |
| REJECTED  | worker 节点拒绝接受 task          |
| ORPHANED  | 节点离线时间超长                  |

#### 8.4.1 查看状态

运行命令`docker service ps <service-name>`来获得 task 的状态。`CURRENT STATE`表示 task 的状态：

---

## 附：docker 命令参考

[docker 常用命令](https://sunbufu.github.io/wiki/docker-command/)

[Docker network 命令](https://blog.csdn.net/gezhonglei2007/article/details/51627821)

[常用命令](https://blog.csdn.net/qq_30118563/article/details/113838214)

[Docker 实践(二)：容器的管理(创建、查看、启动、终止、删除)](https://blog.csdn.net/u010246789/article/details/53958662)

---

## 附：Raft 协议参考

raft 是工程上使用较为广泛的强一致性、去中心化、高可用的分布式协议

参考

- [Raft 协议原理详解，10 分钟带你掌握](https://mp.weixin.qq.com/s/OLcb0nWLp-Oa1DA6ppFbKQ)

- [一文搞懂 Raft 算法](https://www.cnblogs.com/xybaby/p/10124083.html)

---

其他参考：

- [安装教程 01](https://yeasy.gitbook.io/docker_practice/install/centos)

- [安装教程 02](https://cloud.tencent.com/developer/article/1701451)

- [docker swarm 集群创建、配置、可视化管理实验](https://blog.csdn.net/dkfajsldfsdfsd/article/details/79923218)

- [Docker Swarm 管理节点高可用分析](https://www.jianshu.com/p/6e48d01d11c9)

- [使用 swarm 搭建和管理 docker 集群](https://ypdai.github.io/2021/02/24/%E4%BD%BF%E7%94%A8swarm%E6%90%AD%E5%BB%BA%E5%92%8C%E7%AE%A1%E7%90%86docker%E9%9B%86%E7%BE%A4/)

- [Docker Swarm （容器集群，高可用，安全）](https://blog.51cto.com/lwc0329/3010862)

- [<u>Docker Swarm 深入浅出</u>](https://www.gitbook.com/book/holynull/docker-swarm)

- [DockerSwarm 获取 Token 与常用命令](https://www.cnblogs.com/songxingzhu/p/10669497.html)

- [Portainer - Docker 的可视化管理工具使用详解](https://www.hangge.com/blog/cache/detail_2597.html)

- [Docker network 命令](https://blog.csdn.net/gezhonglei2007/article/details/51627821)

- [在 Swarm 集群中使用 Compose 文件](https://laravelacademy.org/post/21850)

- <https://www.cnblogs.com/xiangsikai/p/9935253.html>

|**PS：原《 docker 进阶》见 7.3**

