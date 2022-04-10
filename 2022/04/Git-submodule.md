#  Git-submodule

## 一、添加

```sh
git submodule add <url> <path>
```

- url：替换为自己要引入的子模块仓库地址
- path：要存放的本地路径

执行添加命令成功后，可以在当前路径中看到一个.gitsubmodule文件，里面的内容就是我们刚刚add的内容

如果在添加子模块的时候想要指定分支，可以利用 -b 参数

```sh
git submodule add -b <branch> <url> <path>
```

**例子：**

- 未指定分支

  ```sh
  git submodule add https://github.com/tensorflow/benchmarks.git 3rdparty/benchmarks
  ```

  .gitsubmodule内容

  ```sh
  [submodule "3rdparty/benchmarks"]
  	path = 3rdparty/benchmarks
  	url = https://github.com/tensorflow/benchmarks.git
  ```

- 指定分支

  ```sh
  git submodule add -b cnn_tf_v1.10_compatible https://github.com/tensorflow/benchmarks.git 3rdparty/benchmarks
  ```

  .gitsubmodule内容

  ```sh
  [submodule "3rdparty/benchmarks"]
  	path = 3rdparty/benchmarks
  	url = https://github.com/tensorflow/benchmarks.git
  	branch = cnn_tf_v1.10_compatible
  ```


##  二、使用

当我们add子模块之后，会发现文件夹下没有任何内容。这个时候我们需要再执行下面的指令添加源码。

```sh
git submodule update --init --recursive
```

这个命令是下面两条命令的合并版本

```sh
git submodule init
git submodule update
```

## 三、更新

我们引入了别人的仓库之后，如果该仓库作者进行了更新，我们需要手动进行更新。即进入子模块后，执行

```sh
git pull
```

进行更新。

## 四、删除

1. 删除子模块目录及源码

```sh
rm -rf 子模块目录
```

1. 删除.gitmodules中的对应子模块内容

```sh
vi .gitmodules
```

1. 删除.git/config配置中的对应子模块内容

```sh
vi .git/config
```

1. 删除.git/modules/下对应子模块目录

```sh
rm -rf .git/modules/子模块目录
```

1. 删除git索引中的对应子模块

```sh
git rm --cached 子模块目录
```
