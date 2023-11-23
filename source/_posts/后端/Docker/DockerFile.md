---
title: DockerFile
tags:
  - 基础知识
  - Docker
categories:
  - 后端
  - Docker
date: 2023-11-16 15:57:48
---
## ReadMe

## 定义
Dockerfile是一个用来构建镜像的文本文件，文本内容包含一条条构建镜像所需的指令和说明。其目的是为了构建镜像，不同环境下可以复现相同的容器。
每个指令都会在镜像的构建过程中创建一个**新的镜像层** 
## 包含指令
感谢[https://cloud.tencent.com/developer/article/2327632]
#### FROM
**第一个指令，必须的指令** 用于指定构建新镜像时所基于的基础镜像，可以是官方的也可以是他人的
```dockerfile
格式：
	FROM <image>
	FROM <image>:<tag>
示例：
	FROM nginx:1.25.1-alpine
```
#### MAINTAINER
用于指定镜像的维护者信息，和LABEL指令类似，添加作者、维护者等元数据
```dockerfile
格式：
	MAINTAINER <name>
示例：
	MAINTAINER Henry Fish
	MAINTAINER XXX@qq.com
```
#### LABEL
用于向镜像添加元数据标签，可选指令，为镜像添加描述信息，类似README? **建议尽量选择LABEL指令** ,因为MAINTAINER指令仅局限于指定维护者
```dockerfile
格式：
	LABEL <key>=<value> <key>=<value> <key>=<value>
示例：
	LABEL "com.example.vendor"="ACME Incorporated"
	LABEL com.example.label-with-value="foo"
	LABEL version="1.0"
```
#### RUN
用于再镜像中执行命令，以便再构建过程中安装包配置环境等。
RUN指令执行的**命令**会在新的镜像层中运行，利用D**ocker的缓存机制**在后续构建中，只有该层之前的内容发生变化时才重新运行。
```dockerfile
默认Shell格式：
	RUN apt-get update && apt-get install -y python3
Exec格式：（避免Shell中发生意外的解释问题）
	RUN ["apt-get","update"]
	RUN ["apt-get","install","-y","python3"]
```
因为每个RUN指令都会创建一个新的镜像层，为减少镜像层数，使用&& 连接多个命令
#### CMD
用于定义容器启动时**默认要执行的命令**。一个Dockerfile中**只能包含一个CMD**指令，多个则只有最后一个生效。
使用`docker run`命令启动容器时，没有其他指定的命令，会执行CMD中定义的命令，有则会覆盖CMD定义的默认启动命令。
```dockerfile
Shell格式：在Shell中执行
	CMD python app.py
Exec格式：在容器中直接执行
	CMD ["python","app.py"]
```

#### ENTRYPOINT
用于配置容器启动时的默认执行命令，类似于CMD指令,一样有shell格式和exec数组格式
- ENTRYPOINT指令的命令会在容器启动时始终执行，无论`docker run`是否指定其他命令，不会被**覆盖**，而是作为容器的主要执行命令
- 如果在`docker run`命令指定了其他命令，这些命令将作为ENTRYPOINT指令的参数进行传递
```dockerfile
	FROM ubuntu:latest
	ENTRYPOINT ["echo","Hello"]
```
无其他参数
```dockerfile
$ docker build -t mu_image .
$ docker run my_image
Hello
```
提供其他参数
```
$ docker run my_image "World"
Hello World
```
可以使用ENTRYPOINT指令定义一个可执行的程序或脚本，容器启动时运行该程序，将Docker容器作为可执行应用来使用
#### EXPOSE
用于声明容器在运行时监听的网络端口，**不会实际打开或映射端口**，而是作为一个文档功能，用于告知程序将使用指定的端口。
```dockerfile
FROM ubuntu:latest
# 声明容器将监听80端口
EXPOSE 80
```
在使用docker run命令启动容器时，如果需要通过主机访问容器的80端口，需要使用**-p选项进行端口映射**
```cmd
$ docker run -p 8080:80 my_image
//这样可以通过主机的8080端口访问容器的服务
```

#### ADD
用于将文件、目录或远程URL复制到镜像中，支持**自动解压缩**，将压缩文件解压至指定目录下
```dockerfile
格式：
	ADD SOURCE DESTINATION
示例：
	ADD https://...... /tmp/
```
#### COPY
相比于ADD指令，缺少自动解压功能。在复制文件的场景下，推荐使用COPY，避免引起不必要的意外行为。
#### ENV
设置环境变量的指令，环境变量在容器运行时可用
```dockerfile
格式：
	ENV KEY VALUE
示例：
	ENV MY_NAME  HENRY
	RUN mkdir $MY_NAME
	CMD echo "Hello, $MY_NAME"
```
同时可以通过docker run命令的-e选项覆盖环境变量值
```
$ docker run -e MY_NAME="ALICE" my_image
```
#### VOLUME


#### WORKDIR
设置工作目录(当前目录)，容器启动时，进程的当前工作目录将被设置为WORKDIR指令指定的目录
```dockerfile
FROM ubuntu:latest
WORKDIR /APP
```
#### USER
#### ARG
#### ONBUILD
#### STOPSIGNAL
#### HEALTHCHECK
#### SHELL

## 使用
```dockerfile
# 使用官方的python 3基础镜像
FROM python:3
# 将当前目录下的文件复制到镜像中的 /APP目录中
COPY . /APP
#设置工作目录为
WORKDIR /APP
#安装依赖包（自定义的requirements.txt）
RUN pip install -r requirements.txt
#暴露容器监听端口
EXPOSE 80
#定义容器启动时运行的命令
CMD ["python", "app.py"]
```


## 问题
### 优化
#### 多阶段构建
在一个Dockerfile中使用多个FROM指令，每个FROM指令代表一个构建阶段。每个构建阶段，可以从之前的阶段复制所需的文件，并执行特定的构建操作。**多阶段构建使得最终生成的镜像只包含运行应用程序所必须的依赖和文件**，不包含构建过程中产生的不必要文件和依赖
eg:
```dockerfile
# 构建阶段1
FROM golang:1.17 as builder
WORKDIR /app
COPY . .
RUN go build -o myapp # 编译应用程序
# 构建阶段2
FROM alpine:latest
# 复制编译后的应用程序
COPY --from=builder /app/myapp /usr/local/bin/
WORKDIR /usr/local/bin
CMD ["myapp"]
```
#### 多层构建优化
在一个Dockerfile中使用多个RUN指令来构建镜像，通过&&操作符将多个命令合并成一个，减少Docker镜像的大小
## 参考文献
[1]https://cloud.tecent.com/developer/article/2327632
[2]