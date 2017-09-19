---
title: Dockerfile 编写
date: 2017-09-19 15:05:08
tags:
   - Docker
categories: Docker
---

## DockerFile 文件导读

Dockerfile 是一个包含了很多指令的文本文件，基于这些指令我们可以使用build创建一个image。

docker build创建image是从dockerfile和context。构建中的上下文(context)是PATH和URL的文件集合，PATH是指本地的文件系统，URL是一个Git仓库地址。一个上下文通常是递归的，对于PATH包含了子目录，URL包含了它的子模块。`docker build .`

build是有Docker 守护进程执行的，执行的第一件事就是将完整的上下文路径（包括递归的子目录）都会发送给守护进程，在大多场景下，最好的方式是用一个空目录作为上下文路径，然后里面放一个Dockerfile.然后在构建(build)将需要的文件加入到目录里面来。

千万注意不要使用根路径"/"作为PATH，它会导致将整个硬盘的文件都会发送给Docker的守护进程

如果要在构建上下文中使用一个文件，Dockerfile 引用了一些特殊的指令，比如COPY指令，为了优化构建，不需要的文件和目录可以加入到上下文目录的.dockerignore文件中。[dockerignore file](https://docs.docker.com/engine/reference/builder/#dockerignore-file)

一般地，Dockerfile被称作Dockerfile，并且放在上下文目录的根目录。你也可以在构建的时候使用 -f 标记来指定一个你文件系统任何地方中的Dockerfile。
`docker build -f /path/to/a/Dockerfile .`

你也可以在构建成功后保存image时指定远端源仓库(repositories)和标签。

`docker build -t shykes/myapp .`

如果要在构建之后给多个仓库(repositories) 打tag，使用多个-t参数就可以了：
`docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .`

在Docker守护进程执行Dockerfile中的指令之前，它都会进行一下预编译来校验语法是否准确，如果有错误的话会返回error。
```
$ docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```
<!--more-->
Docker守护进程在最终生成image输出imageID前都会一条一条执行Dockerfile中的指令并且确认每条指令的结果。最后守护进程会自动清理你发送的上下文。

注意到每一条指定都是独立执行然后创建一个image。因此 RUN cd /tmp 不会对下一个指令有任何影响。
只要允许的话，Docker会重复使用生成的image中的缓存，来加速docker build 执行成功效率。下面的输出中可以明显的看到使用了缓存：
```
$ docker build -t svendowideit/ambassador .
Sending build context to Docker daemon 15.36 kB
Step 1/4 : FROM alpine:3.2
 ---> 31f630c65071
Step 2/4 : MAINTAINER SvenDowideit@home.org.au
 ---> Using cache
 ---> 2a1c91448f5f
Step 3/4 : RUN apk update &&      apk add socat &&        rm -r /var/cache/
 ---> Using cache
 ---> 21ed6e7fbb73
Step 4/4 : CMD env | grep _TCP= | (sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' && echo wait) | sh
 ---> Using cache
 ---> 7ea8aef582cc
Successfully built 7ea8aef582cc
```

镜像构建完就可以尝试将它推送到远程repo;

### 格式

```
#comment
INSTRUCTION argumets

指令 命令 参数
```

指令(INSTRUCTION)是大小写无关的，不过大家约定习俗般将它总是大写来区分指令和命令

Docker 执行Instruction是有顺序的。一个Dockerfile文件必须用`FROM`指令开始。FROM指令制定了构建的基础镜像。FROM只能在在最前面。

\# 是docker的注释符。

### 指令解析


### 环境变量替换

环境变量由ENV语句声明也可以用在Dockerfile中；在dockerfile中使用ENV定义的变量可以使用$variable\_name 和 ${variable_name},当然也有转义字符：'\',如果\$variable\_name ,指代的就是$variable\_name;

```
FROM busybox
ENV foo /bar
WORKDIR ${foo}     # WORKDIR /bar
ADD . ${foo}       # ADD . /bar
COPY \$foo /quux   # COPY $foo /quux
```

环境变量在Dockerfile中可以被以下的执行指令支持：
`ADD, COPY, ENV, EXPOSE, FROM, LABEL, STOPSIGNAL, USER, VOLUME, WORKDIR, ONBUILD`,1.4版本之前的ONBUILD不支持。

在一整条的指令中，环境变量是同一个值：

```
ENV abc=hello
ENV abc=bye def=$abc
ENV ghi=$abc

```

上文中def会返回hello，而不是bye，ghi会返回bye，因为在第二条指令中，set abc=bye 和def=$abc 并不是同一条指令。

### .dockerignore 文件

### 指令

#### FROM

1. FROM <image> [AS <name>]
2. FROM <image>[:<tag>] [AS <name>]
3. FROM <image>[@<digest>] [AS <name>]

FROM指令初始化了一个构建的舞台，并且为接下的操作设置了基础image。一个有效的Dockerfile必须由FROM开始。

默认FROM会有一个`latest` tag。

#### RUN

1. RUN <command>  # shell 形式执行，默认shell执行器，如/bin/sh -c
2. RUN ["executable","param1","param2"] # exec 形式执行

如果使用shell方式可以使用反斜杠来连接语句：
```
RUN /bin/bash -c 'source $HOME/.bashrc; \
echo $HOME'
```
上面相当于一句话：`RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'`


#### CMD

一个Dockerfile里面只能有执行一个CMD指令,如果有多个CMD，只有最后一个会执行。

CMD的三种写法

1. CMD ["executable","param1","param2"]
2. CMD ["param1","param2"]
3. CMD command param1 param2

shell版本：

```
FROM ubuntu
CMD echo "This is a test." | wc -
```

#### LABEL

LABEL是给镜像加源数据，通常是键值对的形式。

`LABEL <key>=<value> <key>=<value> <key>=<value> ...`

实例：

1:

```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

2:

```
LABEL multi.label1="value1" multi.label2="value2" other="value3"
```

3:

```
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

使用`docker inspect`查看：

```
"Labels": {
    "com.example.vendor": "ACME Incorporated"
    "com.example.label-with-value": "foo",
    "version": "1.0",
    "description": "This text illustrates that label-values can span multiple lines.",
    "multi.label1": "value1",
    "multi.label2": "value2",
    "other": "value3"
},
```

#### MAINTAINER [deprecated]

`MAINTAINER <name>` 作者信息

使用LABEL更加灵活：LABEL maintainer="SvenDowideit@home.org.au"

#### EXPOSE

`EXPOSE <port> [<port>...]`

在容器运行的时候暴露端口，不会自己和宿主机绑定。如果要和宿主机绑定需要-p，制定端口。或者-P随机系统端口。


#### ENV

```
ENV <key> <value>
ENV <key>=<value> ...
```

注意第一种没有=号，第二个有等号。
实例1：
```
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
```

实例2：
```
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```

推荐使用实例1的写法，因为这样只会生成一个单缓存层(single cache layer);


#### ADD 

```
ADD <src>... <dest>
ADD ["<src>",... "<dest>"] # 如果有空格必须采取这种方式。
```

ADD的作用就是将src目录，文件，或者url的内容加入到镜像文件系统的dest目录下。

实例：
```
ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```

ADD的匹配规则和golang语言的filepath.Match匹配规则一样。

dest 是一个绝对路径，或者`WORKDIR`的相对路径。

ADD 遵守下面的规则：

1. src 目录必须在构建上下文之内。不可以使用`ADD ../something /something`,因为在第一步使用docker build的时候就已经发送了上下文路径包括子目录给了docker守护进程
2. 如果src是一个url，而这个url没有斜杠结尾，那么它就以为是一个文件，会下载这个文件然后拷贝到dest目录中。
3. 如果src是url，dest没有用斜杠结尾，那么就会生成dest/filename.比如`ADD http://abc.com/foobar /` 会创建成`/foobar`
4. 如果src是一个目录，目录下的所有文件都会被拷贝，包括文件系统的metadata。本身的文件夹不拷贝。


#### COPY

```
COPY <src>... <dest>
COPY ["<src>",... "<dest>"] #如果路径有空格，采用这种方式
```

用法和ADD基本类似：

```
COPY hom* /mydir/        # adds all files starting with "hom"
COPY hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"
```

#### ENTRYPOINT

```
ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)
ENTRYPOINT command param1 param2 (shell form)
```


#### VOLUME

`VOLUME ["/data"]` 也可以用`VOLUME /var/log /var/db` `VOLUME /var/log` 来作为分享的数据卷。

```
FROM ubuntu
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol
```
上面的意思就是挂载一个/myvol，然后在执行的时候用`docker run -v test:/myvol`


#### USER

```
USER <user>[:<group>] or
USER <UID>[:<GID>]
```

#### WORKDIR

```
WORKDIR /path/to/workdir
```

WORKDIR 指令会建立一个RUN，CMD，ENTRYPOINT，COPY，ADD这些指令的工作目录。如果没有WORKDIR，它会被接下来的任何一个Dockerfile中的指令创建，就算创建之后不会使用，它还是会存在。

WORKDIR可以在一个Dockerf中出现多次，如果提供的是相对路径，那么接下来的所有路径都会跟前一个WORKDIR相对关联。比如：

```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

最后执行的pwd路径为/a/b/c。WORKDIR指令也可以有环境变量使用ENV来设置：

```
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```
最后的路径会是/path/$DIRNAME

#### ARG

#### STOPSIGNAL

#### HEALTHCHECK

#### SHELL