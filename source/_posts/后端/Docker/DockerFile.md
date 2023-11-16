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
用于声明容器在运行时监听的网络端口，**不会实际打开或映射端口**，而是作为一个文档功能，用于告知程序将使用指定的端口


#### ADD
#### COPY
#### ENV
#### VOLUME

#### WORKDIR
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
## 参考文献
[1]https://cloud.tecent.com/developer/article/2327632
[2]