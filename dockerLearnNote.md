# Docker入门学习笔记

B站——[【公开课】Docker入坑教程【33集】 庄七](https://www.bilibili.com/video/av17854410?from=search&seid=1543774132825041407)

***
[1. 什么是Docker？](#1)

[2. Docke的目标](#2)

[3. Docker通常应用场景](#3)

[4. Docker的基本组成](#4)

[5. Docker基本操作](#5)

[6. 守护式容器](#6)

[7. 在容器中部署静态网站](#7)

[8. 镜像的相关操作](#8)

[9. Docker的C/S模式](#9)

[10. Docker守护进程的配置和操作](#10)

[11. Docker的远程访问](#11)

[12. Dockerfile指令](#12)

[13. Dockerfile构建过程](#13)

[14. Docker容器的网络连接](#14)

[15. Docker容器的数据管理](#15)

[16. Docker容器的跨主机连接](#16)

***
<h4 id='1'>1. 什么是Docker？</h4>

- 将应用程序自动部署到容器

***
<h4 id='2'>2. Docke的目标</h4>

- 提供简单轻量的建模方式
- 职责的逻辑分离
- 快速高效的开发周期
- 鼓励面向服务的架构

***
<h4 id='3'>3. Docker通常应用场景</h4>

- 使用Docker容器开发、测试、部署服务
- 创建隔离的运行环境
- 搭建多用户的平台即服务（PaaS）基础设施
- 提供软件即服务（SaaS）应用程序
- 高性能、超大规模的宿主机部署

***
<h4 id='4'>4. Docker的基本组成</h4>

- Docker Client 客户端
- Docker Daemon 守护进程
- Docker Image 镜像
- Docker Container 容器
- Docker Registey 仓库

***
<h4 id='5'>5. Docker基本操作</h4>

运行容器

    # docker run IMAGE [COMMAND] [ARG…]
        run 在新容器中执行命令

启动交互式容器：
    
    # docker run -i -t IMAGE /bin/bash
        -i  --interactive=true|false 默认为false
        -t  --tty=true|false 默认为false

查看容器：

    # docker ps [-a] [-l]
    # docker inspect 容器名/id

自定义容器名：

    # docker run --name=自定义名 -i -t /bin/bash

重新启动已经停止的容器：

    # docker start [-i] 容器名/id

删除停止的容器：

    # docker rm 容器名/id

***
<h4 id='6'>6. 守护式容器</h4>

- 能够长期运行
- 没有交互式会话
- 适合运行应用程序和服务

以守护形式运行容器

    # docker run -i -t IMAGE /bin/bash
    Ctrl+P Ctrl+Q

附加到运行中的容器

    # docker attach 容器名/id

启动守护式容器

    # docker run -d 镜像名 [COMMAND] [ARG…]
        -d  以后台运行方式启动

查看容器日志

    # docker logs [-f] [-t] [--tail] 容器名/id
        -f --follows=true|false 默认为false 
            一直跟踪日志变化并返回结果，Ctrl+C停止
        -t --timestamps=true|false 默认为false
            附加时间戳
        --tail ="all"
            返回结尾处日志，未指定则返回所有日志

查看容器内进程

    # docker top 容器名/id

在运行中的容器内启动新进程

    # docker exec [-d][-i][-t] 容器名/id [COMMAND] [ARG…]

停止守护容器

    # docker stop 容器名/id     发送信号后等待服务器停止
    # docker kill 容器名/id     直接终止

使用docker帮助文档

    man docker-run
        
***
<h4 id='7'>7. 在容器中部署静态网站</h4>

设置容器的端口映射

    run [-P] [-p]
        -P --publish-all=true | false 默认为false
            # docker run -P -i -t ubuntu /bin/bash
            映射所有端口
        -p --publish=[]     指定映射端口
            containerPort
                # docker run -p 80 -i -t ubuntu /bin/bash
            hostPort:containerPort
                # docker run -p 8080:80 -i -t ubuntu /bin/bash
            ip:containerPort
                # docker run -p 0.0.0.0:80 -i -t ubuntu /bin/bash
            ip:hostPort:containerPort
                # docker run -p 0.0.0.0:8080:80 -i -t ubuntu /bin/bash

Nginx部署静态网页流程

    - 创建映射80端口的交互式容器
    - 安装Nginx
    - 安装文本编辑器vim
    - 创建静态页面
    - 修改Nginx配置文件
    - 运行Nginx
    - 验证网站访问

```vim
# docker run -p 80 --name web -i -t ubuntu /bin/bash
apt-get update
apt-get install -y nginx
apt-get install -y vim

mkdir -p /var/www/html
cd /var/www/html
vim index.html
    <html>
        <head>
            <title>Nginx in Docker</title>
        </head>
        <body>
            <h1>hello, I'm website in docker!</h1>
        </body>
    </html>

whereis nginx
vim /etc/nginx/sites-enabled/default
    修改 root 的值为 /var/www/html

cd /
nginx
ps -ef
Ctrl+P Ctrl+Q 退出，后台运行

# docker ps     查看运行的容器
# docker port web       查看端口映射情况
# docker top web        查看容器进程情况

curl http://127.0.0.1:32768     以映射的端口为准

# docekr inspect web        查看容器对应的IPAddress
curl 172.17.0.2

# 关闭容器
# docker stop web
# docker start -i web
# ps -ef
# nignx并未启动

# docker exec web nginx
# 重新启动Nginx
# 映射的端口和IP地址均已改变
```

***
<h4 id='8'>8. 镜像的相关操作</h4>

Docker镜像：
        
        容器的基石
        层叠的只读文件系统
        联合加载（union mount）

        # docker info
        查看docker配置的相关信息

列出镜像

    # docker images [OPTIONS] [REPOSITORY]
        -a  --all=False 显示所有镜像
        -f  --filter=[] 显示时的过滤条件
        --no-trunc=false    不使用截断的形式显示命令
        -q  --quiet=false   只显示镜像的v-id

查看镜像

    # docker inspect [OPTIONS] CONTAINER|IMAGE[CONTAINER|IMAGE……]

删除镜像

    # docker rmi [OPTIONS] IMAGE [IMAGE…]
    -f  --force=false   强制删除镜像
    --no-prune=false    保留删除镜像中未打标签的父镜像

查找镜像

    Docker Hub  https://hub.docker.com/
    # docker search [OPTIONS] TERM
        --automated=false   只显示自动化构建的选项
        --no-trunc=false    以截断方式显示image id
        -s  --stars=0   设置显示结果的最低星级
        一次最多返回25个结果

拉取镜像

    # docker pull [OPTIONS] NAME[:TAG]
        -a  --all-tags=false    下载仓库中所有标签镜像

    提速
        使用--registry-mirror选项
            1.修改：vim /etc/default/docker
            2.添加：DOCKER_OPTS = "--registry-mirror=http://MIRROR-ADDR"
                https://www.daocloud.io

推送镜像

    # docker push NAME[:TAG]

构建镜像

    保存对容器的修改，并再次使用
    自定义镜像的能力
    以软件的形式打包并分发服务及其运行环境

    1.使用commit构建镜像
        # docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
        -a  --author="" 指定镜像作者，一般：name@email
        -m  --message=""    提交信息
        -p  --pause=true    在commit过程中不暂停容器

        ********************demo***************
        # docker run -it -p 80 --name commit_test ubuntu /bin/bash
        # apt-get update
        # apt-get install -y nginx
        # exit
        # docker ps -l
        生成镜像
        # docker commit -a "summer" -m "nginx" commit_test summer258/commit_test1
        # docker images
        # docker run -d --name nginx_web -p 80 summer258/commit_test1 nginx -g "daemon off;"
        # docker ps
        # curl http://127.0.0.1:32770
        ***************************************

    2.使用dockerfile构建镜像

        # docker build [OPTIONS] PATH | URL | -
            --force-rm=false
            --no-cache=false
            --pull=false
            -q  --quiet=false
            --rm=true
            -t  --tag=""

        创建一个Dockerfile
            # first dockerfile
            FROM ubuntu:14.04
            MAINTAINER  name "name@email"
            RUN apt-get update
            RUN apt-get install -y nginx
            EXPOSE 80

        ********************demo***************
        # mkdir -p dockerfile/df_test1
        # cd dockerfile/df_test1
        # vim Dockerfile
            # first dockerfile
            FROM ubuntu:14.04
            MAINTAINER  name "name@email"
            RUN apt-get update
            RUN apt-get install -y nginx
            EXPOSE 80
        # docker build -t='summer258/df_test1' .
        # docker run -d --name nginx_web2 -p 80 summer258/df_test1 nginx -g "daemon off;"
        # docker ps
        # curl http://127.0.0.1:32770
        ***************************************
        
***
<h4 id='9'>9. Docker的C/S模式</h4>

![Alt](https://img-blog.csdnimg.cn/20190801111028168.png)

Remote API

RESTful风格的API
STDIN、STDOUT、STDERR

![Alt](https://img-blog.csdnimg.cn/2019080111200989.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTY0NTU1,size_16,color_FFFFFF,t_70)


连接方式

unix:///var/run/docker.sock
tcp://host:port
fd://socketfd

![Alt](https://img-blog.csdnimg.cn/20190801111918280.png)

test

    # ps -ef | grep docker
    # docker version
    # nc -U /var/run/docker.sock
    GET /info HTTP/1.1

***
<h4 id='10'>10. Docker守护进程的配置和操作</h4>

查看守护进程

    # ps -ef | grep docker
    # status docker 

使用service命令管理

    # service docker start|stop|restart

Docker的启动选项

    # docker -d [OPTIONS]
        -d 表示以守护进程形式运行

        运行相关：
        -D  --debug=false
        -e  --exec-drive="native"
        -g  --graph="/var/lib/docker"
        --icc=true
        -l  --log-level="info"
        --label=[]
        =p  --pidfile="/var/run/docker.pid"

        Docker服务器连接相关：
        -G,--group="docker"
        -H,--host=[]
        --tls=false
        --tlscacert="/home/sven/.docker/ca.pem"
        --tlscert="/home/sven/.docker/cert.pem"
        --tlskey="/home/sven/.docker/key.pem"
        --tlsverify=false

        RemoteAPI相关：
        --api-enable-cors=false

        存储相关：
        -s,--storage-drive=""
        --selinux-enabled=false
        --storage-opt=[]

        Registry相关：
        --insecure-registry=[]
        --registry-mirror=[]

        网络设置相关：
        -b,--bridge=""
        --bip=""
        --fixed-cidr=""
        --fixed-cidr-v6=""
        --dns=[]
        --dns-search=[]
        --ip=0.0.0.0
        --ip-forward=true
        --ip-masq=true
        --iptables=true
        --ipv6=false
        --mtu=0

docker启动配置文件：/etc/default/docker

    DOCKER_OPTS = "……"

***
<h4 id='11'>11. Docker的远程访问</h4>

环境准备：

    第二台安装Docker的服务器
    修改Docker守护进程启动选项，区别服务器
    保证Client API和Server API版本一致

    # docker version
        查看版本API是否一致

修改服务器端配置

    修改Docker守护进程启动项
    -H  tcp://host:port
        unix:///path/to/socket
        fd://* or fd://socketfd
    守护进程默认配置：
    -H  unix:///var/run/docker.sock

    vim /etc/default/docker
        DOCKER_OPTS = "-H tcp://0.0.0.0:2375"
        # 一般选择2375端口
    重启服务 # service docker restart
    ifconfig    获取本机IP地址

远程访问

    curl http://服务器IP：配置port/info

客户端远程访问：

    修改客户端配置
    使用Docker客户端命令选项
    -H  tcp://host:port
        unix:///path/to/socket
        fd://* or fd://socketfd
    客户端默认配置：
    -H  unix:///var/run/docker.sock
    
    # docker -H tcp://ip:port info

    使用环境变量简化操作DOCKER_HOST
    export DOCKER_HOST="tcp://ip:port"

    将环境变量置空则返回本机服务
    export DOCKER_HOST=""

设置了远程访问模式后的服务端不再支持本机连接

    1.将本机作为远程客户端，通过DOCKER_HOST连接
    2.修改本机配置，添加默认配置
    vim /etc/default/docker
        DOCKER_OPTS = "-H tcp://0.0.0.0:2375 -H  unix:///var/run/docker.sock"
    
***

<h4 id='12'>12. Dockerfile指令</h4>


创建一个Dockerfile

    # first dockerfile
    FROM ubuntu:14.04
    MAINTAINER  name "name@email"
    RUN apt-get update
    RUN apt-get install -y nginx
    EXPOSE 80

指令格式

    # Comment
    INSTRUCTION argument

FROM \<image>[:\<tag>]

    已经存在的镜像
    基础镜像
    必须是第一条非注释指令

MAINTAINER  \<name>

    指定镜像的作者信息，包含镜像的所有者和联系信息

RUN

    指定当前镜像中运行的命令
    shell模式
        RUN <command>
        默认shell   /bin/sh -c command
    exec模式
        RUN [ "executable", "param1", "param2"]
        可以指定其他shell   RUN ["/bin/bash", "-c", "echo hello"]

EXPOSE \<port> [\<port>…] 

    指定运行该镜像的容器使用的端口
    在使用时仍需指定    -p port

CMD

    提供容器运行时的命令，作为默认设置，会被docker run中相同命令覆盖
    CMD [ "executable", "param1", "param2"] (exec模式)
    CMD command param1 param2   (shell模式)
    CMD ["param1", "param2"]    (作为ENTRYPOINT指令的默认参数)

ENTRYPOINT

    ENTRYPOINT [ "executable", "param1", "param2"] (exec模式)
    ENTRYPOINT command param1 param2   (shell模式)
    默认不会被docker run中命令覆盖
    可以使用 docker run entrypoint 覆盖

ADD、COPY

    将文件/目录复制到Dockerfile构建的文件中，源地址，目标地址
    ADD <src>…<dest>
    ADD ["<src>"…"<dest>"]  (使用于文件路径中有空格的情况)

    COPY <src>…<dest>
    COPY ["<src>"…"<dest>"]  (使用于文件路径中有空格的情况)

    ADD vs COPY
        ADD包含类似tar的解压功能
        如果单纯复制文件，Docker推荐使用COPY

VOLUME ["/data"]

    用于向基于镜像创建的容器添加数据卷，共享数据/数据持久化

WORKDIR /path/to/workdir

    在镜像创建新容器时，指定工作目录，一般使用绝对路径，相对路径会持续传递

ENV \<key>\<value>

ENV \<key>=\<value>…

    用户设置环境变量

USER daemon

    指定镜像运行的用户，默认为root

ONBUILD [INSTRUCTION]

    镜像触发器
    当一个镜像被其他镜像作为基础镜像时执行
    会在构建过程中插入指令

***
<h4 id='13'>13. Dockerfile构建过程</h4>

- 从基础镜像运行一个容器
- 执行一条指令，对容器做出修改
- 执行类似docker commit的操作，提交一个新的镜像层
- 基于刚提交的镜像运行一个新容器
- 执行Dockefile中的下一条指令，直至所有指令执行完毕
- 
- 
- 中间层镜像不会被删除，中间层容器会被删除
- 可以使用中间层镜像调试

    查找错误

- 构建缓存
- 不适用构建缓存
    ```vim
    # docker build --no-cache
    或者
    在Dockerfile中设置缓存刷新时间
    ENV REFRESH_DATE 2019-8-2
    ```
- 查看镜像构建的过程
    ```vim
    # docker history [image]
    ```

***

<h4 id='14'>14. Docker容器的网络连接</h4>

- Docker容器的网络基础

![Alt](https://img-blog.csdnimg.cn/20190802093313966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTY0NTU1,size_16,color_FFFFFF,t_70)

    docker0——Linux虚拟网桥

        Linux虚拟网桥:
            可以设置IP地址
            相当于拥有一个隐藏的虚拟网卡

        docker0 地址划分
            IP:172.17.42.1  子网掩码：255.255.0.0
            MAC：02:42:ac:11:00:00到02:42:ac:11:ff:ff
            总共提供了65534个地址
        
        安装网桥管理程序
            yum -y install bridge-utils
        查看网桥设备
            # brctl show

    自定义Docker0

        修改docker0地址
        $ sudo ifconfig docker0 192.168.200.1 netmask 255.255.255.0

    自定义虚拟网桥

        添加虚拟网桥
            $ sudo brctl addbr br0
            $ sudo ifconfig br0 192.168.100.1 netmask 255.255.255.0
        更改docker守护进程的启动配置
            /etc/default/docker 中添加DOCKER_OPTS值
            -b=br0

- Docker容器的互联

环境准备

    用于测试的Docker镜像Dockerfile：
        # ./cct/Dockerfile
        # container connection test
        FROM ubuntu:14.04
        RUN apt-get install -y ping
        RUN apt-get update
        RUN apt-get install -y nginx
        RUN apt-get install -y curl
        EXPOSE 80
        CMD /bin/bash

        # docker build -t summer258/cct .
    
允许所有容器间互联

    默认设置
    --icc=true  默认

    启动第一个容器
    # docker run -it --name cct1 summer258/cct
    /# nginx
    Ctrl+P Ctrl+Q

    启动第二个容器
    # docker run -it --name cct2 summer258/cct
    /# ifconfig
    Ctrl+P Ctrl+Q

    inet addr:172.17.0.3

    连接到第一个容器cct1
    # docker attach cct1
    /# ifconfig

    inet addr:172.17.0.2

    测试连接：ping 172.0.3

    连接到第一个容器cct2
    # docker attach cct2
    测试连接：curl http://172.17.0.2

    --link
        # docker run --link=[CONTAINER_NAME]:[ALIAS] [IMAGE] [COMMAND]

    启动第三个容器
    # docker run -it --name cct3 --link=cct1:webtest summer258/cct
    /# ping webtest

    重启docker服务
        # systemctl restart docker
        # docker restart cct1 cct2 cct3
        /# ping webtest

拒绝所有容器间互联

    Docker守护进程的启动选项
        DOCKER_OPTS= " --icc=false"

    # ubuntu
    $ sudo vim /etc/default/docker
        DOCKER_OPTS = "--icc=false"
    # centos
    $ sudo vim /etc/docker/daemon.json
        {
            …
            “icc”:false,
            …
        }

    
    $ sudo systemctl restart docker
    $ ps -ef | grep docker
    $ sudo docker restart cct1 cct2 cct3
    $ sudo docker attach cct3
    /# ping webtest

允许特定容器间的连接

    Docker守护进程的启动选项
        --icc=false --iptables=true
        --link

    $ sudo iptables -F
    $ sudo iptables -L -n

- Docker容器与外部网络的连接

ip-forward

    --ip-forward=true
        决定系统是否会转发流量

    $ sudo sysctl net.ipv4.conf.all.forwarding

iptables

    iptables是与Linux内核集成的包过滤防火墙系统，
    几乎所有的Linux发行版本都会包含iptables的功能

![Alt](https://img-blog.csdnimg.cn/20190802105416694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTY0NTU1,size_16,color_FFFFFF,t_70)

    filter表中包含的链：
        INPUT
        FORWARD
        OUTPUT

    $ sudo iptables -L -n

限制IP访问容器

    $ sudo iptables -I DOCKER -s 10.211.55.3 -d 127.17.0.7 -p TCP --dport 80 j DROP

端口映射访问

***

<h4 id='15'>15. Docker容器的数据管理</h4>

- Docker容器的数据卷

    什么是数据卷？（Data Volume）

    - 数据卷是经过特殊设计的目录，可以绕过联合文件系统（UFS），为一个或多个容器提供访问
    - 数据卷设计的目的，在于数据的永久化，它完全独立于容器的生存周期，因此，Docker不会在容器删除时删除其挂载的数据卷，也不会存在类似的垃圾收集机制，对容器引用的数据卷进行处理。

    ![Alt](https://img-blog.csdnimg.cn/20190802111712300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTY0NTU1,size_16,color_FFFFFF,t_70)

    数据卷（Data Volume）的特点

    - 数据卷在容器启动时初始化，如果容器使用的镜像在挂载点包含了数据，这些数据会拷贝到新初始化的数据卷中。
    - 数据卷可以在容器之间共享和重用。
    - 可以对数据卷里的内容直接进行修改
    - 数据卷的变化不会影响镜像的更新
    - 卷会一直存在，及时挂载数据卷的容器已经被删除

    为容器添加数据卷

        $ sudo docker run -it -v ~/container_data:/data ubuntu /bin/bash
        /# ls

        $ sudo docker inspect 容器名/id
        可以查看Volume目录信息

    为数据卷添加访问权限

        $ sudo docker run -it -v ~/datavolume:/data:ro ubuntu /bin/bash
        ro：只读

    使用Dockerfile构建包含数据卷的镜像

        Dockerfile指令：VOLUME["/data"]
        不能映射到已经存在的文件目录，系统默认创建


- Docker的数据卷容器

    什么是数据卷容器：

        命名的容器挂载数据卷，其他容器通过挂载这个容器实现数据共享，挂载数据卷的容器，就叫做数据卷容器。

    ![Alt](https://img-blog.csdnimg.cn/20190802115901717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTY0NTU1,size_16,color_FFFFFF,t_70)

    挂载数据卷容器的方法

        $ sudo docker run --volumes-from [CONTAINER NAME]


- Docker数据卷的备份和还原

    数据备份方法

        $ sudo docker run --volumes-from [container name] -v $(pwd):/backup[:wr] ubuntu tar cvf /backup/backup.tar [container data volume]

    ![Alt](https://img-blog.csdnimg.cn/2019080212104349.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTY0NTU1,size_16,color_FFFFFF,t_70)

    数据还原方法

        $ sudo docker run --volumes-from [container name] -v $(pwd):/backup[:wr] ubuntu tar xvf /backup/backup.tar [container data volume]

***

<h4 id='16'>16. Docker容器的跨主机连接</h4>

- 使用网桥实现跨主机连接

    原理：

    网络拓扑

    ![Alt](https://img-blog.csdnimg.cn/20190802151358850.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTY0NTU1,size_16,color_FFFFFF,t_70)

    环境准备：

        Mac OS X + Parallels
        两台Ubuntu 14.04 虚拟机
        安装网桥管理工具：
            apt-get install bridge-utils
        IP地址：Host1：10.211.55.3
                Host2：10.211.55.5
        
    网络配置：

        修改/etc/network/interfaces 文件
        auto br0
        iface br0 inet static
        address 10.211.55.3
        netmask 255.255.255.0
        gateway 10.211.55.1
        bridge_ports eth0

    Docker设置：

        修改/etc/default/docker文件
        -b 指定使用自定义网桥
            -b=br0
        --fixed-cidr限制IP地址分配范围
            IP地址划分：
            Host1：10.211.55.64/26
                地址范围：10.211.555.65~10.211.55.126
            Host2:192.168.59.126/26
                地址范围：10.211.55.129~10.211.55.190
    优点：

        配置简单，不依赖第三方软件

    缺点：

        与主机在同网段，需要小心划分IP地址
        需要有网段控制权，在生产环境中不易实现
        不容易管理
        兼容性不佳

- 使用Open vSwitch实现跨主机容器连接

    Open vSwitch是什么？

    &emsp;&emsp;Open vSwitch是一个高质量、多层虚拟交换机，使用开源Apache2.0许可协议，由Nicira Networks开发，主要实现代码为可移植的C代码。它的目的是让大规模网络自动化可以通过编程扩展，同时仍然支持标准的管理接口和协议（例如NetFlow，sFlow，SPAN，RSPAN，CLI，LACP，8022.lag）

    原理：

    ![Alt](https://img-blog.csdnimg.cn/20190802153203460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTY0NTU1,size_16,color_FFFFFF,t_70)

    什么是GRE隧道？

    &emsp;&emsp;GRE：通用路由协议封装

    &emsp;&emsp;隧道技术（Tunneling）是一种通过使用互联网络的基础设施在网络之间传递数据的方式。使用随带传递的数据（或负载）可以使不同协议的数据帧或包。隧道协议将其它协议的数据帧或包重新封装然后通过隧道发送新的帧头提供路由信息，以便通过互联网传递被封装的负载数据。

    环境准备：

        Mac OS X + Virtualbox
        两台Ubuntu14.04虚拟机
        双网卡，Host-Only&NAT
        安装Open vSwitch：
            apt-get install openvswitch-switch
        安装网桥管理工具：
            apt-get install bridge-utils
        IP地址：Host1：192.168.59.103
                Host2：192.168.59.104

    操作：

        建立ovs网桥
        添加gre连接
        配置docker容器虚拟网桥
        为虚拟网桥添加ovs接口
        添加不同Docker容器网段路由

        主机：192.168.59.103
        查看ovs状态
        $ sudo ovs-vsctl show
        建立ovs网桥
        $ sudo ovs-vsctl add-br obr0
        添加gre接口
        $ sudo ovs-vsctl add-port obr0 gre0
        $ sudo ovs-vsctl set interface gre0 type=gre options:remote_ip=192.168.59.104
        $ sudo ovs-vsctl show

        建立本机docker需要使用的网桥
        $ sudo brctl addbr br0
        $ sudo ifconfig br0 192.168.1.1 netmask 255.255.255.0
        $ sudo brctl addif br0 obr0
        $ sudo brctl show

        配置docker，用新建的网桥代替docker0
        $ sudo vim /etc/default/docker
            DOCKER_OPTS="obr0"
        $ sudo service docker restart

        新建一个docker容器
        $ sudo docker run -it ubuntu /bin/bash
        /# ifconfig
            inet addr:192.168.1.2
        /# ping 192.168.59.104

        切换主机：192.168.59.104
        $ ifconfig
            inet addr:192.168.2.1
        启动一个docker容器
        $ sudo docker run -it ubuntu /bin/bash
        /# ifconfig
            inet addr:192.168.2.4
        /# ping 192.168.59.104

        切回主机：192.168.59.103
        /# ping 192.168.2.4
            ping不通，因为不同网段需要查找路由表来确定不同网段的网络地址
        $ route
        $ sudo ip route add 192.168.2.0/24 via 192.168.59.104 dev eth0

        $ sudo docker run -it ubuntu /bin/bash
        /# ping 192.168.2.4
        
- 使用weave实现跨主机容器连接

    weave是什么？

        语义：编织
        建立一个虚拟的网络，用于将运行在不同主机的Docker容器连接起来
        https://www.weave.works/
        https://github.com/weaveworks/weave#readme

    ![Alt](https://img-blog.csdnimg.cn/20190802160953349.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM5NTY0NTU1,size_16,color_FFFFFF,t_70)

    环境准备：

        Mac OS X + Virtualbox
        两台Ubuntu14.04虚拟机
        双网卡，Host-Only&NAT
        IP地址：Host1：192.168.59.103
                Host2:192.168.59.104
    
    操作：

        安装weave
        启动weave
            $ weave launch
        连接不同主机
        通过weave启动容器

        主机：192.168.59.103
        $ sudo wget -O /usr/bin/weave https://raw.githubusercontent.com/zettio/weave/master/weave
        $ sudo chmod a+x /usr/bin/weave
        $ weave launch
        $ sudo docker ps -l

        切换主机：192.168.59.104
        重复安装操作
        $ sudo wget -O /usr/bin/weave https://raw.githubusercontent.com/zettio/weave/master/weave
        $ sudo chmod a+x /usr/bin/weave
        $ weave launch 192.168.59.103
        启动容器，将ID赋值给c2
        c2=$(weave run 192.168.1.2/24 -it ubuntu /bin/bash)
        $ echo $c2
        $ docker attach $ c2
        /# ifconfig
        多了一个网络设备ethwe，inet addr：192.168.1.2

        切换Host1:192.168.59.103
        $ weave run 192.168.1.10/24 -it --name wc1 ubuntu /bin/bash
        相同网段的IP地址
        $ sudo docker aatach wc1
        /# ifconfig
        增加wthwe，inet addr:192.168.1.10
        /# ping 192.168.1.2
        能够ping通
        
***
