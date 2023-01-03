# CentOS 7 安装和配置 Git

## 一、安装依赖

```sh
sudo yum install -y wget
sudo yum install -y gcc-c++
sudo yum install -y zlib-devel perl-ExtUtils-MakeMaker curl-devel expat-devel
```

## 二、获取压缩包

```sh
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.33.0.tar.gz
```

## 三、解压

```sh
tar -zxvf tar -zxvf git-2.33.0.tar.gz
```

## 四、编译

```sh
# 进入解压后的目录
cd git-2.33.0/
# 指定安装位置
./configure --prefix=/usr/local
# 执行 mack 命令编译
make 
# 安装
make install
```

  解释：

>1）./configure 命令就是执行当前目录的名为 configure 的脚本，主要的作用是对即将安装的软件进行配置，检查当前的环境是否满足要安装软件的依赖关系，并把生成的makefile放到....onePackage/install下
>2）编译 make
>make 的基本用处是自动根据 makefile 里的指令来编译源文件。
>3） 安装 make install
>make install：将程序安装至系统中。如果原始码编译无误，且执行结果正确，便可以把程序安装至系统预设的可执行文件存放路径。默认/usr/local/bin

  补充：

  如果需要安装到其他位置则需要配置环境变量。

> 配置文件：  /etc/profile

```config
# git
export PATH=/xxx/git/bin:$PATH
export PATH=$PATH:/xxx/git/libexec/git-core:$PATH
```

## 五、验证

```sh
git --version
```

## 五、配置

1. 配置用户名

   ```sh
   # git config --global user.name "Your Name"
   git config --global user.name "qinweizhao"
   ```

2. 配置邮箱

   ```sh
   # git config --global user.email "email@example.com"
   git config --global user.email "qinweizhao1997@163.com"
   ```

3. 生成公钥和私钥

   ```sh
   # ssh-keygen -t rsa -C "youremail@example.com"
   ssh-keygen -t rsa -C "qinweizhao1997@163.com"
   ```
