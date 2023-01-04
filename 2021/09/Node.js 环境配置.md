# Node.js 环境配置

## 一、在 Nodejs 安装目录新建两个文件夹

> // 全局包下载存放
>
>node_global
>
>// 缓存
>
>node_cache node

![2021-09-22_230024](https://img.qinweizhao.com/2021/09/2021-09-22_230024.png)

## 二、修改路径

打开 cmd 窗口，执行命令

```sh
npm config set prefix "D:\Node\nodejs\node_global"
npm config set cache "D:\Node\nodejs\node_cache"
```

或：

在Nodejs 的安装目录中找到 node_modules/npm/npmrc 文件，修改如下：

```npmrc
prefix=D:\Node\nodejs\node_global
cache=D:\Node\nodejs\node_cache
```

全局包的存放路径可通过 npm root -g 查看

![2021-09-22_233236](https://img.qinweizhao.com/2021/09/2021-09-22_233236.png)

## 三、配置环境变量

- 变量名： **NODE_PATH**

  变量值： **D:\Nodejs\node_global\node_modules**
  
- 变量名： **PATH**

  变量值： **D:\Nodejs\node_global**
  
  ![2021-09-22_232437](https://img.qinweizhao.com/2021/09/2021-09-22_232437.png)

![2021-09-22_232525](https://img.qinweizhao.com/2021/09/2021-09-22_232525.png)
