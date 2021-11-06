# Docker 命令集合

## 一、基本命令

 ```bash
 # Docker 安装
 yum -y install docker-io
 docker --version
 # 启动 Docker 进程
 systemctl start docker
 systemctl status docker
 # 设置开机自启动 docker 服务
 systemctl enable docker
 ```

## 二、 镜像操作

```bash
# 查看镜像并下载
docker search hello-world
docker pull hello-world
# 查看已下载的镜像
docker images
# 运行镜像
docker run hello-world
# 查看所有容器（容器来自于镜像）
docker ps -a
# 使用 docker logs 查看容器控制台输出获取容器的日志
docker logs hello-world
# 从一个 tar 包创建一个镜像，往往和 export 结合使用
docker import
# 使用 Dockerfile 创建镜像（推荐）
docker build
# 从容器创建镜像
docker commit
# 删除一个镜像
docker rmi
# 从一个 tar 包创建一个镜像，和 save 配合使用
docker load
# 将一个镜像保存为一个tar包，带 layers 和 tag 信息
docker save
# 显示生成一个镜像的历史命令
docker history
# 为镜像起一个别名
docker tag
```

## 三、容器操作

```bash
# 创建一个容器但是不启动它
docker create
# 创建并启动一个容器
docker run
# 停止容器运行，发送信号 SIGTERM
docker stop
# 启动一个停止状态的容器
docker start
# 重启一个容器
docker restart
# 删除一个容器
docker rm
# 发送信号给容器，默认 SIGKILL
docker kill
# 连接（进入）到一个正在运行的容器
docker attach
# 阻塞一个容器，直到容器停止运行
docker wait
# 执行 在容器里执行一个命令，可以执行bash进入交互式
docker exec
# 显示状态为运行 (Up) 的容器
docker ps
# 显示所有容器,包括运行中 (Up) 的和退出的 (Exited)
docker ps -a
# 深入容器内部获取容器所有信息
docker inspect
# 查看容器的日志 (stdout/stderr)
docker logs
# 得到 docker 服务器的实时的事件
docker events
# 显示容器的端口映射
docker port
# 显示容器的进程信息
docker top
# 显示容器文件系统的前后变化
docker diff
# 从容器里向外拷贝文件或目录
docker cp
# 将容器整个文件系统导出为一个 tar 包，不带 layers、tag 等信息
docker export
```

## 四、镜像仓库 (registry) 操作

```bash
# 登录到一个 registry
docker login
# 从 registry 仓库搜索镜像
docker search
# 从仓库下载镜像到本地
docker pull
# 将一个镜像 push 到 registry 仓库中
docker push
```
