---
title: "Docker 常用命令"
date: 2020-07-27T00:22:31+08:00
publishdate: 2020-07-27T00:22:31+08:00
lastmod: 2020-11-25T11:51:31+08:00
draft: false
tags: ["docker", "command"]
---
## 查看 Docker 信息
- 常看 docker 版本：
    ```shell
    $ docker version
    ```

- 查看 docker 系统信息：
    ```shell
    $ docker info
    ```

## 镜像操作
- 根据 Dockerfile 创建 Docker 镜像
    ```shell
    # 1. 首先创建一个新的目录 docker
    # 2. 在目录下新建 Dockerfile 文件
    # 3. 完善 Dockerfile 文件内容
    # 4. 在新创建的目录下使用 docker build 命令构建新的镜像
    # 
    # 示例：[docker build -t go1.16.0:alpine .]

    $ docker build -t <新的镜像名>:<新镜像的tag> <Dockerfile文件所在路径>
    ```
- 从 DockerHub 检索镜像
    ```shell
    $ docker search <image_name>
    ```
- 拉取镜像
    ```shell
    $ docker pull <image_name>
    ```
- 删除镜像
    ```shell
    $ docker rmi <image_name>
    ```
- 列出镜像
    ```shell
    $ docker images
    ```
- 显示镜像历史
    ```shell
    $ docker history <image_name>
    ```
- 上传镜像到 Docker Hub
    ```shell
    # 为镜像添加附带作者信息的标签
    $ docker tag <image_name>:<label> <author_name>/<image_name>:<label>

    # 上传附带作者信息标签的镜像到 Docker Hub
    $ docker push <author_name>/<image_name>:<label>
    ```
- 清理镜像
    ```shell
    $ docker image prune
    ```

## 容器启动
- 启动容器并启动 bash：
    ```shell
    $ docker run -i -t <image_name/continar_id> /bin/bash
    ```

- 启动容器并启动 sh：
    ```shell
    $ docker run -i -t <image_name/continar_id> /bin/sh
    ```

- 启动容器并以后台方式运行：
    ```shell
    $ docker run -d -it <image_name/continar_id>
    ```

- 启动容器，为容器命名并指定数据卷
    ```shell
    $ docker run -i -t --name <continar_name> -v <source_dir>:<target_dir> <image_name/continar_id> /bin/bash
    ```

- 启动容器并指定端口映射
    ```shell
    $ docker run -i -t -p 5000:5000 <image_name/continar_id> /bin/bash

    # 指定一个地址
    $ docker run -i -t -p 127.0.0.1:5000:5000 <image_name/continar_id> /bin/bash

    # 指定 TCP 端口
    $ docker run -i -t -p 127.0.0.1:5000:5000/tcp <image_name/continar_id> /bin/bash

    #  绑定 localhost 的任意端口到容器的 5000 端口，本地主机会自动分配一个端口
    $ docker run -i -t -p 127.0.0.1::5000 <image_name/continar_id> /bin/bash
    ```

## 容器操作
- 列出正在运行中的容器
    ```shell
    $ docker ps
    ```
- 列出所有容器
    ```shell
    $ docker ps -a 
    ```
- 启动容器
    ```shell
    $ docker start <id/container_name>
    ```
- 停止容器
    ```shell
    $ docker stop <id/container_name>
    ```
- 重启容器
    ```shell
    $ docker restart <id/container_name>
    ```
- 中断容器
    ```shell
    $ docker kill <id/container_name>
    ```
- 进入正在运行的容器并运行 bash
    ```shell
    $ docker exec -t -i <id/container_name>  /bin/bash
    ```
- 退出容器并保持后台运行
    ```
    Ctrl+P+Q
    ```

- 删除容器
    ```shell
    $ docker rm <id/container_name>
    ```
- 修改容器名称
    ```shell
    $ docker rename <old_container_name> <new_container_name>
    ```
- 查看容器日志
    ```shell
    $ docker logs <id/container_name>
    ```
- 实时查看容器日志
    ```shell
    $ docker logs -f <id/container_name>
    ```
- 显示运行中容器的进程信息
    ```shell
    $ docker top <id/container_name>
    ```
- 查看容器内部详细信息
    ```shell
    $ docker inspect <id/container_name>
    ```

## 数据卷
- 创建数据卷
    ```shell
    $ docker volume create <VOLUME_NAME>
    ```
- 查看数据卷
    ```shell
    $ docker volume ls
    ```
