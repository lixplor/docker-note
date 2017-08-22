# Docker


## 概念

* image: 镜像. 是轻量的, 独立的, 可执行的包, 它包含了运行软件的一切内容, 包括代码, 运行环境, 库, 环境变量, 以及配置文件
* container: 容器. 是镜像的一个运行时实例. 也就是镜像被执行后在内存中的体现形式. container默认情况下是完全独立于主机环境的, 仅在进行某些配置的情况下才可以访问主机文件和端口. 容器在本地主机内核中运行应用. 容器比虚拟器拥有更好的效率. 容器可以获取本地访问, 每个容器都运行在单独的进程中, 不会消耗更多的内存.
* Dockerfile: 将程序代码, 运行环境, 依赖等打包为可移植的一个镜像文件
* registry: 注册地. 是仓库的集合
* repository: 仓库. 是镜像的集合. 与Github仓库类似, 但不同的是代码是已被构建过的. 一个账户可以创建多个仓库
* stack: 栈. 定义服务之间如何交互
* service: 服务. 运行中的每个应用都可以看做一个服务


* Docker的3层:
    - Stack: 顶层
    - Service: 中层
    - Container: 底层

## 安装Docker

* 官方文档
    - [支持系统列表](https://docs.docker.com/engine/installation/#cloud)
    - [CentOS](https://docs.docker.com/engine/installation/linux/docker-ce/centos/)
    - [Debian](https://docs.docker.com/engine/installation/linux/docker-ce/debian/)
    - [Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)


## 使用Dockerfile定义一个容器

* 步骤
    1. 创建一个空目录, 进入该目录
    2. 在该目录中创建一个文件, 名为`Dockerfile`
    3. 编辑`Dockerfile`, 在其中填入以下配置
    4. 创建配置文件中的依赖文件
    5. 构建一个Docker镜像: `docker build -t 镜像名称 目录`
    6. 查看镜像: `docker images`


* `Dockerfile`

```bash
# 使用官方Python运行时作为父镜像 Use an official Python runtime as a parent image
FROM python:2.7-slim

# 设置工作目录为 /app Set the working directory to /app
WORKDIR /app

# 将当前目录的内容复制到/app目录中的容器内 Copy the current directory contents into the container at /app
ADD . /app

# 安装requirements.txt文件中指定的所需依赖包 Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# 让80端口可在容器外使用 Make port 80 available to the world outside this container
EXPOSE 80

# 定义环境变量 Define environment variable
ENV NAME World

# 当容器启动时运行app.py Run app.py when the container launches
CMD ["python", "app.py"]
```

* `requirements.txt`
    - 注意这里只是演示, 实际安装Redis会出错

```txt
Flask
Redis
```

* `app.py`

```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```


## 分享镜像

* 首先需要在registry上有一个Docker ID, 可在[cloud.docker.com](cloud.docker.com)注册
* 从本机上登录: `docker login`


## 服务

* `docker-compose.yml`文件: 定义Docker容器的行为



## Docker常用命令

```bash
# 查看docker版本
docker --version

# 构建镜像
docker build -t 镜像名 目录

# 查看本地机器中的镜像
docker images

# 查看所有镜像
docker images -a

# 前台运行镜像(需要Ctrl + c停止)
docker run -p 本机端口:容器端口 镜像名

# 后台运行镜像
docker run -d -p 本机端口:容器端口 镜像名

# 查看正在运行的容器
docker ps

# 查看所有容器, 即使没有运行
docker ps -a

# 停止正在运行的容器
docker stop 容器ID

# 强制停止某个容器
docker kill 容器ID

# 删除容器
docker rm 容器ID

# 删除所有容器
docker rm $(docker ps -a -q)

# 登录Docker ID
docker login

# 标记镜像要上传到哪个仓库
docker tag <image> username/repository:tag

# 上传已标记的镜像到仓库
docker push username/repository:tag

# 从registry中运行镜像
docker run username/repository:tag

# 列出当前Docker主机上运行的所有应用
docker stack ls

# 运行指定的compose文件
docker stack deploy -c <composefile> <appname>

# 列出某个应用相关的所有服务
docker stack services <appname>

# 列出某个应用相关的正在运行的容器
docker stack ps <appname>

# 删除应用
docker stack rm <appname>
```






## 常见问题

#### 启动Docker提示`Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.`
有错误导致Docker退出. 执行`systemctl status docker.service -l`查看错误详情. 注意加上`-l`可以看到详细信息, 比如Linux内核太低导致无法启动

#### Linux内核过低, 如何升级

```shell
# 查看内核信息
uname -a

# 升级kernel
yum list kernel
yum install kernel

# 修改内核启动顺序


# 没有grub2的话安装一下
yum install grub2
```

#### 如何删除id为none的的镜像

* 步骤
    1. 先删除引用了该镜像的容器
        - `docker ps -a`: 查看所有容器
        - `docker rm <container_id>`: 删除容器
    2. 删除该镜像
        - `docker rmi <image_id>`
