# CentOS 更换 Yum 源

## 一、将原来的CentOS-Base.repo进行备份

```sh
# 进入源所在目录
cd /etc/yum.repos.d

# 备份
mv CentOS-Base.repo CentOS-Base.repo_back
```

## 二、下载阿里源

```sh
# 下载源
wget -nc http://mirrors.aliyun.com/repo/Centos-7.repo
```

## 三、更改阿里yum源为默认源

```sh
 mv Centos-7.repo CentOS-Base.repo
```

## 四、更新缓存

```sh
# 清除
yum clean all
# 生成
yum makecache
```

## 五、更新

```sh
yum -y update
```

## 六、补充国内 yum 源

- 阿里yum源：<http://mirrors.aliyun.com/repo/>
- 163(网易)yum源：<http://mirrors.163.com/.help/>

- 中科大的Linux安装镜像源：<http://centos.ustc.edu.cn/>

- 搜狐的Linux安装镜像源：<http://mirrors.sohu.com/>

- 北京首都在线科技：<http://mirrors.yun-idc.com/>
