# 自我修养

#### 阅读Docker官网文档

```
阐述Dockerfile文件如何使用
https://docs.docker.com/engine/reference/builder/
1. 系统指令不区分大小写，但是官方推荐一律大写这已成为一个约定俗成的规范
2. Dockerfile文件的指令是顺序执行
3. Dockerfile文件的第一行必须是`FROM`指令，注释除外
4. `#`开头一般是注释的含义,另一个情况是`Parser directives`
5. .dockerignore忽略某些文件或者目录

阐述服务编排易用、性方便性
https://docs.docker.com/compose/install/
https://docs.docker.com/compose/compose-file/

1.如何安装docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

2. docker-compose是什么
Compose is a tool for defining and running multi-container Docker applications. 
With Compose, you use a YAML file to configure your application’s services. 

3. 如何使用
docker-compose up

4. Compose文件版本
分为三个大的版本：1.0、2.0、3.0，当然每个大的版本中也分了很多小的版本代号。目前官网给出最新的是`version: 3.7`

YAML文件格式，所有：后必须要有空格，否则无法顺利编译
version: "3.7"  指定版本号
services:   服务固定名称
  webapp:   任意取名称
    build:  build === 需要执行某个目录下的dockerfile
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

# 输出

## docker-compose

```
version: '3'
services:
    nginx:
        image: nginx:latest
        ports:
            - "80:80"
        depends_on:
            - "php"
        volumes:
            - "$PWD/conf.d:/etc/nginx/conf.d"
            - "$PWD/html:/usr/share/nginx/html"
        container_name: "compose-nginx"

    php:
        image: php:7.2-fpm
        ports:
            - "9000:9000"
        volumes:
            - "$PWD/html:/var/www/html"
        container_name: "compose-php"
```

## Dockerfile说明

#### 基本语法

从Dockerfile文件执行docker build构建一个镜像。在构建过程中可以指定`PATH`或者`URL`，`PATH`必须是宿主机本地路径，URL是Git仓库地址

因build过程是递归操作，所以`PATH`的子目录和`URL`的submodules都会被执行

```
Usage:	docker build [OPTIONS] PATH | URL | -

Build an image from a Dockerfile

Options:
      --build-arg list             Set build-time variables (default [])
      --cache-from stringSlice     Images to consider as cache sources
      --cgroup-parent string       Optional parent cgroup for the container
      --compress                   Compress the build context using gzip
      --cpu-period int             Limit the CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int              Limit the CPU CFS (Completely Fair Scheduler) quota
  -c, --cpu-shares int             CPU shares (relative weight)
      --cpuset-cpus string         CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string         MEMs in which to allow execution (0-3, 0,1)
      --disable-content-trust      Skip image verification (default true)
  -f, --file string                Name of the Dockerfile (Default is 'PATH/Dockerfile')
      --force-rm                   Always remove intermediate containers
      --help                       Print usage
      --isolation string           Container isolation technology
      --label list                 Set metadata for an image (default [])
  -m, --memory string              Memory limit
      --memory-swap string         Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --network string             Set the networking mode for the RUN instructions during build (default "default")
      --no-cache                   Do not use cache when building the image
      --pull                       Always attempt to pull a newer version of the image
  -q, --quiet                      Suppress the build output and print image ID on success
      --rm                         Remove intermediate containers after a successful build (default true)
      --security-opt stringSlice   Security options
      --shm-size string            Size of /dev/shm, default value is 64MB
  -t, --tag list                   Name and optionally a tag in the 'name:tag' format (default [])
      --ulimit ulimit              Ulimit options (default [])
  -v, --volume list                Set build-time bind mounts (default [])
```

> **注意**： 不推荐使用系统根目录`/`作为`PATH`，因为在构建时Docker会把系统根目录下的所有文件复制到容器内


#### 忽略文件或目录

因`COPY`和`ADD`指令是递归复制操作，但是有时候我们不希望目录中的某个文件或者目录被打包。那么只需要在当前目录中添加`.dockerignore`

- .dockerignore格式说明

规则 | 行为
---|---
# comment | Ignored.
*/temp* | Exclude files and directories whose names start with temp in any immediate subdirectory of the root. For example, the plain file /somedir/temporary.txt is excluded, as is the directory /somedir/temp.
*/*/temp | Exclude files and directories starting with temp from any subdirectory that is two levels below the root. For example, /somedir/subdir/temporary.txt is excluded.
temp? | Exclude files and directories in the root directory whose names are a one-character extension of temp. For example, /tempa and /tempb are excluded.


```
# comment
*/temp*
*/*/temp*
temp?
```

**目录结构**
```
├── aa.tar.gz
├── Dockerfile
├── .dockerignore
└── files
    ├── aa.tar.gz
    ├── php-nginx-compose.tar.gz
    └── test.php
```

**执行构建命令**
```
docker build -t local_ubuntu .
```

# 语法规则

**通过官方文档得知**
1. 系统指令不区分大小写，但是官方推荐一律大写这已成为一个约定俗成的规范
2. Dockerfile文件的指令是顺序执行
3. Dockerfile文件的第一行必须是`FROM`指令，注释除外
4. `#`开头一般是注释的含义,另一个情况是`Parser directives`

```
# Comment
FROM nginx
```

#### FROM语法

基础镜像，Dockerfile第一行，执行`docker build`如果本地存在给镜像则跳过,否则默认从Docker Hub官方下载一个镜像

`FROM`在Dockerfile必须出现一次或者多次

```
FROM <image> [AS <name>]
FROM <image>[:<tag>] [AS <name>]
FROM <image>[@<digest>] [AS <name>]
```

- `ARG`指令定义Dockerfile参数，需定义在FROM指令之前
- `FROM`指令可以定义多个，如果定义了则以最后一个作为镜像ID

```
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```

#### RUN语法

```
RUN <command>
RUN ["executable", "param1", "param2"]
```

#### CMD语法

提供容器运行时默认执行`CMD`

```
CMD ["executable","param1","param2"] (exec form, this is the preferred form)
CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
CMD command param1 param2 (shell form)
```

> 注意：Dockerfile文件仅仅只能存在一个`CMD`指令，如果存在多个则只有最后一个`CMD`指令生效

#### EXPOSE语法

指定容器运行时所需暴露监听端口

```
EXPOSE <port> [<port>/<protocol>...]
```

#### ENV语法

设置容器运行时所需环境变量

```
ENV <key> <value>
ENV <key>=<value> ...

ENV APP_VERSION=1.0.0
```

#### ADD语法

增加文件或者目录

```
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"] (this form is required for paths containing whitespace)

ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"

ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
ADD test /absoluteDir/         # adds "test" to /absoluteDir/
```

> 注意：如果添加压缩文件，`ADD`指令会自动解压。支持格式有：`identity, gzip, bzip2 or xz`

> 注意：添加远端压缩文件不会自动解压

> 如果dest是个相对路径则自动添加到`WORKDIR`目录下


#### COPY语法

复制文件或者目录

```
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"] (this form is required for paths containing whitespace)

COPY hom* /mydir/        # adds all files starting with "hom"
COPY hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"

COPY test relativeDir/   # adds "test" to `WORKDIR`/relativeDir/
COPY test /absoluteDir/  # adds "test" to /absoluteDir/
```

> 如果dest是个相对路径则自动添加到`WORKDIR`目录下


#### ENTRYPOINT语法

容器运行时默认执

```
ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)
ENTRYPOINT command param1 param2 (shell form)
```

**ENTRYPOINT和CMD区别**
- `CMD`参数能被`docker run`参数覆盖


#### VOLUME语法

```
VOLUME ["/data"]
```

docker会自动绑定主机上的一个目录。通过docker inspect 命令可以查看到
```
"Mounts": [
    {
        "Type": "volume",
        "Name": "7b08e9a13f313e30ddf9dc580578445826a496846b1aff2f7c0cb60ca9bfd939",
        "Source": "/var/lib/docker/volumes/7b08e9a13f313e30ddf9dc580578445826a496846b1aff2f7c0cb60ca9bfd939/_data",
        "Destination": "/home",
        "Driver": "local",
        "Mode": "",
        "RW": true,
        "Propagation": ""
    }
],
```


#### WORKDIR语法

设置容器工作目录，[`RUN`, `CMD`, `ENTRYPOINT`, `COPY`, `ADD`]这些命令会使用`WORKDIR`

```
WORKDIR /path/to/workdir

WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

`WORKDIR`可以设置多个，如果后续`WORKDIR`都是是相对路劲则：/a/b/c，否则使用最后一个绝对路径的`WORKDIR`

> 注意：为了避免错误，推荐WORKDIR指令只使用绝对路劲


#### ARG语法

定义构建时变量

```
ARG <name>[=<default value>]

ARG user1=someuser
ARG buildno=1
```

变量定义好后如何使用，只需要在参数名前加上`$`，如下：

```
RUN echo $user1
```

执行命令：docker build `--build-arg` user1=11  -t local_ubuntu .

--build-arg参数会覆盖Dockerfile参数值，另外如果Dockerfile中没有user1参数则会报错

```
[Warning] One or more build-args [user1] were not consumed
```


#### 例子
```
FROM php:7.2-fpm

RUN apt-get update \
    && apt-get install -y iputils-ping \
    && docker-php-ext-install mysqli && docker-php-ext-enable mysqli
```