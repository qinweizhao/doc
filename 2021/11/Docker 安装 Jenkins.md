# Docker 安装 Jenkins

>Jenkins 是一个是基于 Java 开发的一种持续集成工具。

## 一、拉取镜像

```bash
docker pull jenkins/jenkins
```

## 二、创建挂载目录

```bash
# 当前位置应该为 /usr/local
mkdir -p mount/jenkins/jenkins_home
# 授权
chmod 777 mount/jenkins/jenkins_home
```

## 三、创建并启动容器

```bash
docker run -d -p 8080:8080 -v /usr/local/mount/jenkins/jenkins_home:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkins jenkins/jenkins
```

命令解释

>**-d** 后台运行镜像
>
>**-p 8080:8080** 将镜像的8080端口映射到服务器的10240端口。
>
>**/usr/local/mount/jenkins/jenkins_home:/var/jenkins_mount /var/jenkins_home** 目录为容器jenkins工作目录，我们将硬盘上的一个目录挂载到这个位置，方便后续更新镜像后继续使用原来的工作目录。这里我们设置的就是上面我们创建的 /var/jenkins_mount目录。
>
>**-v /etc/localtime:/etc/localtime** 让容器使用和服务器同样的时间设置。
>
>**--name jenkins** 给容器起一个别名

## 四、查看是否成功

```bash
# 查看运行中的容器
docker ps

# 或

# 查看日志
docker logs jenkins
```

![2021-11-23_163858](../../../img/2021/11/2021-11-23_163858.png)

## 五、配置镜像加速

 当前位置 /usr/local/mount/jenkins/jenkins_home，修改 hudson.model.UpdateCenter.xml 文件。

![2021-11-23_164513](../../../img/2021/11/2021-11-23_164513.png)

更换 url 为 清华大学官方镜像

> https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

## 六、解锁 Jenkins

**访问 http:ip:8080** ，出现如下界面。

![2021-11-23_165124](../../../img/2021/11/2021-11-23_165124.png)

因为容器的 **/var/jenkins_home** 挂载在本地的 **/usr/local/mount/jenkins/jenkins_home** ，所以直接去查看 **/usr/local/mount/jenkins/jenkins_home/secrets/initialAdminPassword** 文件获取密码。

![2021-11-23_165555](../../../img/2021/11/2021-11-23_165555.png)

## 七、安装插件

选择安装推荐的插件即可

![2021-11-23_171023](../../../img/2021/11/2021-11-23_171023.png)

