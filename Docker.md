# Docker

## 基本概述

Docker 是一个开源的应用容器引擎，诞生于 2013 年初，基于 Go 语言实现， dotCloud 公司出品

Docker 让开发者打包开发应用以及依赖包到一个轻量级、可移植的容器中，可以发布到任何Linux机器上

* 容器是完全使用沙箱机制，相互隔离

* 容器性能开销极低。

Docker 架构：

* **镜像（Image）：**Docker 镜像，就相当于一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统

* **容器（Container）**：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和对象一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等
* **仓库（Repository）**：仓库可看成一个代码控制中心，用来保存镜像

![](https://img.jiapeng.store/img/202307081712491.png)

安装步骤：

```sh
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce
```

配置镜像加速器：

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://hicqe4pi.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```





***



## 操作命令

### 进程相关

* 启动docker服务：

  ```sh
  systemctl start docker
  ```

* 停止docker服务：

  ```sh
  systemctl stop docker
  ```

* 重启doker服务：

  ```sh
  systemctl restart docker
  ```

* 查看doker服务状态：

  ```sh
  systemctl status docker
  ```

* 设置开机启动docker服务：

  ```sh
  systemctl enable docker
  ```

  



### 镜像相关

* 查看镜像：查看本地所有的镜像

  ```sh
  docker images
  docker images –q # 查看所用镜像的id
  ```

* 搜索镜像：从网络中查找需要的镜像

  ```sh
  docker search 镜像名称
  ```

* 拉取镜像：从Docker仓库下载镜像到本地，镜像名称格式为 名称:版本号，如果版本号不指定则是最新的版本。如果不知道镜像版本，可以去docker hub 搜索对应镜像查看

  ```sh
  docker pull 镜像名称
  ```

* 删除镜像：删除本地镜像

  ```sh
  docker rmi 镜像id # 删除指定本地镜像
  docker rmi `docker images -q`  # 删除所有本地镜像 tab上面的键
  ```





### 容器相关

* 查看容器：

  ```sh
  docker ps # 查看正在运行的容器
  docker ps –a # 查看所有容器
  ```

* 创建并启动容器：

  ```sh
  docker run 参数  --name=... /bin/bash
  ```

  参数说明：

  * -i：保持容器运行，通常与 -t 同时使用，加入it这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭
  * -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用
  * -d：以守护（后台）模式运行容器。创建一个容器在后台运行，需要使用docker exec 进入容器。退出后，容器不会关闭
  * **-it 创建的容器一般称为交互式容器，-id 创建的容器一般称为守护式容器**
  * **--name：为创建的容器命名**

* 进入容器：

  ```sh
  docker exec 参数 # 退出容器，容器不会关闭
  ```

* 停止容器：

  ```sh
  docker stop 容器名称
  ```

* 启动容器：

  ```sh
  docker start 容器名称
  ```

* 删除容器：如果容器是运行状态则删除失败，需要停止容器才能删除

  ```sh
  docker rm 容器名称
  ```

* 查看容器信息：

  ```sh
  docker inspect 容器名称
  ```

  



****



## 数据卷

> Docker 容器删除后，在容器中产生的数据也会随之销毁
> Docker 容器和外部机器可以直接交换文件吗？
> 容器之间想要进行数据交互？

<img src="https://img.jiapeng.store/img/202307081712492.png" style="zoom:67%;" />

**数据卷**：数据卷是宿主机中的一个目录或文件，当容器目录和数据卷目录绑定后，对方的修改会立即同步

* 一个数据卷可以被多个容器同时挂载

* 一个容器也可以被挂载多个数据卷

数据卷的作用：

* 容器数据持久化
* 外部机器和容器间接通信
* 容器之间数据交换

配置数据卷

* 创建启动容器时，使用-v参数设置数据卷

  ```sh
  docker run ... –v 宿主机目录(文件):容器内目录(文件) ... 
  docker run -it --name=c1 -v /root(or~)/data:/root/data_container centos:7
  ```

  注意事项：

  1. 目录必须是绝对路径

  2. 如果目录不存在，会自动创建

  3. 可以挂载多个数据卷

多容器进行数据交换：

* 多个容器挂载同一个数据卷
* 数据卷容器

<img src="https://img.jiapeng.store/img/202307081712493.png" style="zoom:50%;" />

* 创建启动c3数据卷容器，使用 –v 参数设置数据卷

  ```sh
  docker run –it --name=c3 –v /volume centos:7 /bin/bash 
  ```

* 创建启动 c1 c2 容器，使用 –-volumes-from 参数设置数据卷

  ```sh
  docker run –it --name=c1 --volumes-from c3 centos:7 /bin/bash
  docker run –it --name=c2 --volumes-from c3 centos:7 /bin/bash  
  ```





***



## 应用部署

### MySQL

在Docker容器中部署MySQL，通过外部mysql客户端操作MySQL Server

端口映射：

* 容器内的网络服务和外部机器不能直接通信，外部机器和宿主机可以直接通信，宿主机和容器可以直接通信

* 当容器中的网络服务需要被外部机器访问时，可以将容器中提供服务的端口映射到宿主机的端口上。外部机器访问宿主机的该端口，从而间接访问容器的服务。这种操作称为：**端口映射**

  ![](https://img.jiapeng.store/img/202307081712494.png)

MySQL部署步骤：搜索mysql镜像，拉取mysql镜像，创建容器，操作容器中的mysql

1. 搜索mysql镜像

   ```sh
   docker search mysql
   ```

2. 拉取mysql镜像

   ```mysql
   docker pull mysql:5.6
   ```

3. 创建容器，设置端口映射、目录映射

   ```sh
   # 在/root目录下创建mysql目录用于存储mysql数据信息
   mkdir ~/mysql
   cd ~/mysql
   ```

   ```sh
   docker run -id \
   -p 3307:3306 \
   --name=c_mysql \
   -v $PWD/conf:/etc/mysql/conf.d \
   -v $PWD/logs:/logs \
   -v $PWD/data:/var/lib/mysql \
   -e MYSQL_ROOT_PASSWORD=123456 \
   mysql:5.6
   ```

   参数说明：

   * `-p 3307:3306`：将容器的 3306 端口映射到宿主机的 3307 端口
   * `-v $PWD/conf:/etc/mysql/conf.d`：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf，配置目录
   * `-v $PWD/logs:/logs`：将主机当前目录下的 logs目录挂载到容器的 /logs，日志目录
   * `-v $PWD/data:/var/lib/mysql` ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。数据目录
   * `-e MYSQL_ROOT_PASSWORD=123456`**：**初始化 root 用户的密码。

4. 进入容器，操作mysql

   ```sh
   docker exec –it c_mysql /bin/bash
   ```

5. 使用外部机器连接容器中的mysql



****



### Tomcat

1. 搜索tomcat镜像

   ```sh
   docker search tomcat
   ```

2. 拉取tomcat镜像

   ```sh
   docker pull tomcat
   ```

3. 创建容器，设置端口映射、目录映射

   ```sh
   # 在/root目录下创建tomcat目录用于存储tomcat数据信息
   mkdir ~/tomcat
   cd ~/tomcat
   ```

   ```sh
   docker run -id --name=c_tomcat \
   -p 8080:8080 \
   -v $PWD:/usr/local/tomcat/webapps \
   tomcat 
   ```

   参数说明：

   * `-p 8080:8080`：将容器的8080端口映射到主机的8080端口

   * `-v $PWD:/usr/local/tomcat/webapps`：将主机中当前目录挂载到容器的webapps

4. 使用外部机器访问tomcat



***



### Nginx

1. 搜索nginx镜像

   ```sh
   docker search nginx
   ```

2. 拉取nginx镜像

   ```sh
   docker pull nginx
   ```

3. 创建容器，设置端口映射、目录映射

   ```sh
   # 在/root目录下创建nginx目录用于存储nginx数据信息
   mkdir ~/nginx
   cd ~/nginx
   mkdir conf
   cd conf
   # 在~/nginx/conf/下创建nginx.conf文件,粘贴下面内容
   vim nginx.conf
   ```

   ```sh
   user  nginx;
   worker_processes  1;
   
   error_log  /var/log/nginx/error.log warn;
   pid        /var/run/nginx.pid;
   
   
   events {
       worker_connections  1024;
   }
   
   
   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;
   
       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
   
       access_log  /var/log/nginx/access.log  main;
   
       sendfile        on;
       #tcp_nopush     on;
   
       keepalive_timeout  65;
   
       #gzip  on;
   
       include /etc/nginx/conf.d/*.conf;
   }
   ```

   ```sh
   docker run -id --name=c_nginx \
   -p 80:80 \
   -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
   -v $PWD/logs:/var/log/nginx \
   -v $PWD/html:/usr/share/nginx/html \
   nginx
   ```

   参数说明：

   * `-p 80:80`：将容器的 80端口映射到宿主机的 80 端口
   * `-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf`：将主机当前目录下的 /conf/nginx.conf 挂载到容器的 :/etc/nginx/nginx.conf，配置目录
   * `-v $PWD/logs:/var/log/nginx`：将主机当前目录下的 logs 目录挂载到容器的/var/log/nginx，日志目录

4. 使用外部机器访问nginx



***



### Redis

1. 搜索redis镜像

   ```sh
   docker search redis
   ```

2. 拉取redis镜像

   ```sh
   docker pull redis:5.0
   ```

3. 创建容器，设置端口映射

   ```sh
   docker run -id --name=c_redis -p 6379:6379 redis:5.0
   ```

4. 使用外部机器连接redis

   ```sh
   ./redis-cli.exe -h 192.168.149.135 -p 6379
   ```





***



## 镜像原理

### 底层原理

> Docker 镜像本质是什么？
> Docker 中一个centos镜像为什么只有200MB，而一个centos操作系统的iso文件要几个个G？
> Docker 中一个tomcat镜像为什么有500MB，而一个tomcat安装包只有70多MB？

操作系统的组成部分：进程调度子系统、进程通信子系统、内存管理子系统、设备管理子系统、文件管理子系统、网络通信子系统、作业控制子系统

Linux文件系统由bootfs和rootfs两部分组成：

* bootfs：包含bootloader（引导加载程序）和 kernel（内核）

* rootfs： root文件系统，包含的就是典型 Linux 系统中的/dev，/proc，/bin，/etc等标准目录和文件

* 不同的linux发行版，bootfs基本一样，而rootfs不同，如ubuntu，centos

Docker镜像原理：

* Docker镜像是一个**分层文件系统**，是由特殊的文件系统叠加而成，最底端是 bootfs，并复用宿主机的bootfs ，第二层是 root文件系统rootfs称为base image，然后再往上可以叠加其他的镜像文件

* 统一文件系统（Union File System）技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统

* 一个镜像可以放在另一个镜像的上面。位于下面的镜像称为父镜像，最底部的镜像成为基础镜像。

* 当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器

问题：

* Docker 中一个Ubuntu镜像为什么只有200MB，而一个Ubuntu操作系统的iso文件要几个个G？
  Ubuntu的iso镜像文件包含bootfs和rootfs，而docker的Ubuntu镜像复用操作系统的bootfs，只有rootfs和其他镜像层
* Docker 中一个tomcat镜像为什么有500MB，而一个tomcat安装包只有70多MB？
  由于docker中镜像是分层的，tomcat虽然只有70多MB，但他需要依赖于父镜像和基础镜像，所有整个对外暴露的tomcat镜像大小500多MB



***



### 镜像制作

![](https://img.jiapeng.store/img/202307081712495.png)

****



### Dockerfile

#### 基本概述

Dockerfile是一个文本文件，包含一条条的指令，每一条指令构建一层，基于基础镜像最终构建出新的镜像

* 对于开发人员：可以为开发团队提供一个完全一致的开发环境

* 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了 

* 对于运维人员：在部署时，可以实现应用的无缝移植 



| 关键字      | 作用                     | 备注                                                         |
| ----------- | ------------------------ | ------------------------------------------------------------ |
| FROM        | 指定父镜像               | 指定dockerfile基于那个image构建                              |
| MAINTAINER  | 作者信息                 | 用来标明这个dockerfile谁写的                                 |
| LABEL       | 标签                     | 用来标明dockerfile的标签 可以使用Label代替Maintainer 最终都是在docker image基本信息中可以查看 |
| RUN         | 执行命令                 | 执行一段命令 默认是/bin/sh 格式: RUN command 或者 RUN ["command" , "param1","param2"] |
| CMD         | 容器启动命令             | 提供启动容器时候的默认命令 和ENTRYPOINT配合使用.格式 CMD command param1 param2 或者 CMD ["command" , "param1","param2"] |
| ENTRYPOINT  | 入口                     | 一般在制作一些执行就关闭的容器中会使用                       |
| COPY        | 复制文件                 | build的时候复制文件到image中                                 |
| ADD         | 添加文件                 | build的时候添加文件到image中 不仅仅局限于当前build上下文 可以来源于远程服务 |
| ENV         | 环境变量                 | 指定build时候的环境变量 可以在启动的容器的时候 通过-e覆盖 格式ENV name=value |
| ARG         | 构建参数                 | 构建参数 只在构建的时候使用的参数 如果有ENV 那么ENV的相同名字的值始终覆盖arg的参数 |
| VOLUME      | 定义外部可以挂载的数据卷 | 指定build的image那些目录可以启动的时候挂载到文件系统中 启动容器的时候使用 -v 绑定 格式 VOLUME ["目录"] |
| EXPOSE      | 暴露端口                 | 定义容器运行的时候监听的端口 启动容器的使用-p来绑定暴露端口 格式: EXPOSE 8080 或者 EXPOSE 8080/udp |
| WORKDIR     | 工作目录                 | 指定容器内部的工作目录 如果没有创建则自动创建 如果指定/ 使用的是绝对地址 如果不是/开头那么是在上一条workdir的路径的相对路径 |
| USER        | 指定执行用户             | 指定build或者启动的时候 用户 在RUN CMD ENTRYPONT执行的时候的用户 |
| HEALTHCHECK | 健康检查                 | 指定监测当前容器的健康监测的命令 基本上没用 因为很多时候 应用本身有健康监测机制 |
| ONBUILD     | 触发器                   | 当存在ONBUILD关键字的镜像作为基础镜像的时候 当执行FROM完成之后 会执行 ONBUILD的命令 但是不影响当前镜像 用处也不怎么大 |
| STOPSIGNAL  | 发送信号量到宿主机       | 该STOPSIGNAL指令设置将发送到容器的系统调用信号以退出。       |
| SHELL       | 指定执行脚本的shell      | 指定RUN CMD ENTRYPOINT 执行命令的时候 使用的shell            |



#### Centos

自定义centos7镜像：

1. 默认登录路径为 /usr

2. 可以使用vim

实现步骤：

1. 定义父镜像：FROM centos:7
2. 定义作者信息：MAINTAINER  seazean < zhyzhyang@sina.com>
3. 执行安装vim命令： RUN yum install -y vim
4. 定义默认的工作目录：WORKDIR /usr
5. 定义容器启动执行的命令：CMD /bin/bash
6. 通过dockerfile构建镜像：docker bulid –f dockerfile文件路径 –t 镜像名称:版本



#### Boot

定义dockerfile，发布springboot项目：

实现步骤：

1. 定义父镜像：FROM java:8

2. 定义作者信息：MAINTAINER seazean < zhyzhyang@sina.com>

3. 将jar包添加到容器： ADD springboot.jar app.jar

4. 定义容器启动执行的命令：CMD java–jar app.jar

5. 通过dockerfile构建镜像：docker bulid –f dockerfile文件路径 –t 镜像名称:版本





***



## 服务编排

### 基本介绍

微服务架构的应用系统中一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启停，维护的工作量会很大。

* 从Dockerfile build image 或者去dockerhub拉取image；
* 创建多个container，管理这些container（启动停止删除）

**服务编排**：按照一定的业务规则批量管理容器

Docker Compose是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建，启动和停止。使用步骤：

1. 利用 Dockerfile 定义运行环境镜像

2. 使用 docker-compose.yml 定义组成应用的各服务

3. 运行 docker-compose up 启动应用

![](https://img.jiapeng.store/img/202307081712496.png)



***



### 功能实现

使用docker compose编排nginx+springboot项目

1. 安装Docker Compose

2. 创建docker-compose目录

   ```sh
   mkdir ~/docker-compose
   cd ~/docker-compose
   ```

3. 编写 docker-compose.yml 文件

   ```sh
   version: '3'
   services:
     nginx:
      image: nginx
      ports:
       - 80:80
      links:
       - app
      volumes:
       - ./nginx/conf.d:/etc/nginx/conf.d
     app:
       image: app
       expose:
         - "8080"
   ```

4. 创建./nginx/conf.d目录

   ```sh
   mkdir -p ./nginx/conf.d
   ```

5. 在./nginx/conf.d目录下编写***.conf文件

   ```sh
   server {
       listen 80;
       access_log off;
   
       location / {
           proxy_pass http://app:8080;
       }
   }
   ```

6. 在~/docker-compose 目录下使用docker-compose启动容器

   ```sh
   docker-compose up
   ```

7. 测试访问

   ```sh
   http://192.168.0.137/hello
   ```

   

***





## 私有仓库

Docker官方的Docker hub（https://hub.docker.com）是一个用于管理公共镜像的仓库，我们可以从上面拉取镜像 到本地，也可以把我们自己的镜像推送上去。但是当服务器无法访问互联网，或者不希望将自己的镜像放到公网当中，那么我们就需要搭建自己的私有仓库来存储和管理自己的镜像

* 私有仓库搭建

  ```sh
  # 1、拉取私有仓库镜像 
  docker pull registry
  # 2、启动私有仓库容器 
  docker run -id --name=registry -p 5000:5000 registry
  # 3、输入地址http://私有仓库服务器ip:5000/v2/_catalog，显示{"repositories":[]} 
  # 4、修改daemon.json   
  vim /etc/docker/daemon.json    
  # 在上述文件中添加一个key，保存退出。此步用于让 docker 信任私有仓库地址；注意将私有仓库服务器ip修改为自己私有仓库服务器真实ip 
  {"insecure-registries":["192.168.0.137:5000"]} 
  # 5、重启docker 服务 
  systemctl restart docker
  docker start registry
  ```

* 将镜像上传至私有仓库

  ```sh
  # 1、标记镜像为私有仓库的镜像     
  docker tag centos:7 私有仓库服务器IP:5000/centos:7
   
  # 2、上传标记的镜像     
  docker push 私有仓库服务器IP:5000/centos:7
  ```

* 从私有仓库拉取镜像 

  ```sh
  #拉取镜像 
  docker pull 私有仓库服务器ip:5000/centos:7
  ```

  

***



## 虚拟机

容器：

* 容器是将软件打包成标准化单元，以用于开发、交付和部署

* 容器镜像是轻量的、可执行的独立软件包 ，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置

* 容器化软件在任何环境中都能够始终如一地运行。

* 容器赋予了软件独立性，使其免受外在环境差异的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突

容器和虚拟机对比：

* 相同：容器和虚拟机具有相似的资源隔离和分配优势

* 不同：

  * 容器虚拟化的是操作系统，虚拟机虚拟化的是硬件。
  * 传统虚拟机可以运行不同的操作系统，容器只能运行同一类型操作系统

  ![](https://img.jiapeng.store/img/202307081712497.png)

  | 特性       | 容器               | 虚拟机     |
  | ---------- | ------------------ | ---------- |
  | 启动       | 秒级               | 分钟       |
  | 硬盘使用   | 一般为MB           | 一般为GB   |
  | 性能       | 接近原生           | 弱于原生   |
  | 系统支持量 | 单机支持上千个容器 | 一般几十个 |

  