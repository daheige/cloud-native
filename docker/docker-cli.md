# docker实用的命令
查看包含 pulsar 关键字的images
```shell
docker images --digests | grep pulsar
```
删除镜像
```shell
docker image rm xxx
docker image rm 605c77e624dd
```
列出镜像版本tag
```shell
docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```
列出镜像摘要信息
```shell
docker image ls --digests
```
删除运行的容器
```shell
docker stop 97b8604d68d
docker rm 97b8604d68d
```
查看镜像历史记录
```shell
docker history mynginx:1.0
IMAGE          CREATED        CREATED BY                                       SIZE      COMMENT
5e1c99715dbd   4 months ago   RUN /bin/sh -c echo "<h1>hello docker nginx<…   28B       buildkit.dockerfile.v0
<missing>      4 months ago   MAINTAINER nginx-demo <zhuwei313@hotmail.com>    0B        buildkit.dockerfile.v0
<missing>      4 months ago   /bin/sh -c set -x     && apkArch="$(cat /etc…   29.2MB    
<missing>      4 months ago   /bin/sh -c #(nop)  ENV NJS_VERSION=0.7.9         0B        
<missing>      4 months ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B        
<missing>      4 months ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT            0B        
<missing>      4 months ago   /bin/sh -c #(nop)  EXPOSE 80                     0B        
<missing>      4 months ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B        
<missing>      4 months ago   /bin/sh -c #(nop) COPY file:e57eef017a414ca7…   4.62kB    
<missing>      4 months ago   /bin/sh -c #(nop) COPY file:abbcbf84dc17ee44…   1.27kB    
<missing>      4 months ago   /bin/sh -c #(nop) COPY file:5c18272734349488…   2.12kB    
<missing>      4 months ago   /bin/sh -c #(nop) COPY file:7b307b62e82255f0…   1.62kB    
<missing>      4 months ago   /bin/sh -c set -x     && addgroup -g 101 -S …   4.45MB    
<missing>      4 months ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1             0B        
<missing>      4 months ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.23.3      0B        
<missing>      4 months ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B        
<missing>      4 months ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]               0B        
<missing>      4 months ago   /bin/sh -c #(nop) ADD file:40887ab7c06977737…   7.05MB
```

删除无效的镜像
```shell
docker image prune
```

# Dockerfile 定制镜像
实现原理：
- 每个镜像都由很多层次构成，Docker 使用 Union FS 将这些不同的层结合到一个镜像中去；
- 通常 Union FS 有两个用途, 一方面可以实现不借助 LVM、RAID 将多个 disk 挂到同一个目录 下,另一个更常用的就是将一个只读的分支和一个可写的分支联合在一起，Live CD 正是基于此方法可 以允许在镜像不变的基础上允许用户在其上进行一些写操作。

基础指令：
- FROM 指定基础镜像
对于基础镜像 mysql,redis,nginx,mongo,php,python,node,golang在官方都有提供；对于操作系统ubuntu,centos,debian,alpine都有对应的版本；
- RUN 执行命令
    RUN 指令是用来执行命令行命令的；
    - shell 格式： RUN <命令>，比如 RUN echo "hello"
    - exec 格式： RUN ["可执行文件", "参数1", "参数2"]
    关于Dockerfile层的理解：
    - Dockerfile 中每一个指令都会建立一层， RUN 也不例外，当执行完毕后，就会commit提交当前层；
    - 为了减少很多运行时不需要的东西，建议RUN 在写命令时候使用 && 操作符将各个所需命令串联起来，减少层的叠加；
    - Dockerfile 支持 Shell 类的行尾添加 \ 的命令换行方 式，以及行首 # 进行注释的格式；
    - 清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，减小镜像的大小；
- COPY复制
    例如：COPY ./package.json /app/
    - COPY 这类指令中的源文件的路径都是相对路径；
    - 关于.实际上是在指定上 下文的目录， docker build 命令会将该目录下的内容打包交给 Docker 引擎以帮助构建镜像;
    - 一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下;
    - .dockerignore 忽略文件，该文件是用于剔除不需要作 为上下文传递给 Docker 引擎的；
- CMD 容器启动命令
    CMD 指令的格式：
    - shell 格式： CMD <命令> 
        如果使用 shell 格式的话，实际的命令会被包装为 sh -c 的参数的形式进行执行，比如：CMD echo $HOME 实际执行变成：CMD [ "sh", "-c", "echo $HOME" ]
    - exec 格式： CMD ["可执行文件", "参数1", "参数2"...]
    - 参数列表格式： CMD ["参数1", "参数2"...] 。在指定了 ENTRYPOINT 指令后，用 CMD 指定具体的参数。
    在运行时可以指定新的命令来替代镜像设置中的这个默认命令CMD；
    Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 systemd 去启动后台服务，容器内没有后台服务的概念。
    比如说：CMD service nginx start 容器执行后就立即退出了，对于这个，正确的做法：CMD ["nginx", "-g", "daemon off;"]
- ENTRYPOINT 入口点
    - ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。 ENTRYPOINT 在运行 时也可以替代，不过比 CMD 要略显繁琐，需要通过 docker run 的参数 --entrypoint 来指定。
    - 当指定了 ENTRYPOINT 后， CMD 的含义就发生了改变，不再是直接的运行其命令，而是将 CMD 的内容作为参数传给 ENTRYPOINT 指令。换句话说实际执行时，将变为：<ENTRYPOINT> "<CMD>"
    - ENTRYPOINT 可作为应用运行前的准备工作，以及当作命令执行；
- ENV 设置环境变量
    支持的格式：
    - ENV <key> <value> 格式；
    - ENV <key1>=<value1> <key2>=<value2>... 如果存在多个就用\来换行处理；
    - ENV设置的环境变量支持COPY 、 ENV 、 EXPOSE 、 FROM 、 LABEL 、 USER 、 WORKDIR 、 VOLUME 、 STOPSIGNAL 、 ONBUILD 、 RUN展开；
- ARG 构建参数
    - 格式： ARG <参数名>[=<默认值>] 和ENV不同，ARG 所设置的构建环境的环 境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 ARG 保存密码之类 的信息，因为 docker history 还是可以看到所有值的。
    - Dockerfile 中的 ARG 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 docker build 中用 --build-arg <参数名>=<值> 来覆盖。
- VOLUME 定义匿名卷
    - 容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的 应用，其数据库文件应该保存于卷(volume)中。
    - 为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定 某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储 层写入大量数据；
    设置格式如下：
    - VOLUME ["<路径1>", "<路径2>"...]
    - VOLUME <路径>
    在容器运行时，可以动态改变，比如说 VOLUME /data
    ```shell
    docker run -d -v mydata:/data xxxx
    ```
- EXPOSE 声明端口
    - EXPOSE 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应 用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者 理解这个镜像服务的守护端口，以方便配置映射；
    - 另一个用处则是在运行时使用随机端口映射时，也就 是 docker run -P 时，会自动随机映射 EXPOSE 的端口
    声明格式：
    - 格式为 EXPOSE <端口1> [<端口2>...]
    - 要将 EXPOSE 和在运行时使用 -p <宿主端口>:<容器端口> 区分开来。 -p ，是映射宿主端口 和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 EXPOSE 仅仅是声明容器 打算使用什么端口而已，并不会自动在宿主进行端口映射。
- WORKDIR 指定工作目录
    格式为 WORKDIR <工作目录路径>
    - 使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指 定的目录，如该目录不存在， WORKDIR 会帮你建立目录。
    比如说：
    ```shell
    WORKDIR /app
    RUN echo "hello" > world.txt
    ```
- USER 指定当前用户
    格式： USER <用户名>[:<用户组>]
    和 WORKDIR 一样， USER 只是帮助你切换到指定用户而已，这个用户必须是事先建立好 的，否则无法切换
- 构建镜像docker build -t xxx .

# 运行时指定name
下面的运行容器，指定了--name
```shell
docker run -d --name web -p 80:80 myweb:v1
```

# 多阶段构建
    通过as设置构建别名，比如说
    ```shell
    FROM golang:1.9-alpine as builder
    ```
    从构建好的镜像，复制内容，通过--from来处理。比如说：
    ```shell
    COPY --from=builder /go/src/github.com/go/helloworld/app .
    ```
# 操作容器
容器是 Docker 又一核心概念。 简单的说，容器是独立运行的一个或一组应用，以及它们的运行态环境。对应的，虚拟机可以理解为模 拟运行的一整套操作系统（提供了运行态环境和其他系统环境）和跑在上面的应用。

# 启动容器
```shell
docker run -itd 镜像 xxx --name xxx
```
比如说：
```shell
docker run ubuntu:20.04 /bin/echo "hello,world"
hello,world
```
-t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
-i 则让容器的标准输入保持打开
-d 让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下

```shell
ocker run -ti ubuntu:20.04 /bin/bash
root@6393cc883b98:/# ls
```

# 停止容器
```shell
docker stop e578fdddcc9dc
e578fdddcc9dc
```
# 进入容器
```shell
docker run -itd ubuntu:20.04
e20b0dd2ecf50a7fc66cf42c8b2ea66de7e7ff4c44cde3f82a9e6bd82ac29e4c
$ docker exec -it e20b0dd2 /bin/bash
```
# 删除容器
```shell
docker rm f848d2a -f
f848d2a
```
-f 表示强制删除

# 清理所有处于终止状态的容器
```
#以查看所有已经创建的包括终止状态的容器
docker container ls -a
#清理已终止的容器
docker container prune
```

# 搜索镜像
```shell
docker search centos --filter=stars=10
```
--filter=stars=N 参数可以指定仅显示收藏数量为 N 以上的镜像

# 给镜像打tag
```shell
docker tag ubuntu:20.04 daheige/ubuntu:18.04
```

# 推送镜像
```shell
docker login
docker push daheige/ubuntu:20.04
```
