# webpack

## 一、简介

webpack 是一个前端资源加载/打包工具。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。

## 二、准备

### 1、全局安装

```sh
npm install -g webpack webpack-cli
```

### 2、查看版本号

```sh
webpack -v
```

### 3、初始化项目

创建 webpack-sample 文件夹

```sh
# webpack-sample 目录下执行
npm init -y
```

创建 index.html 文件

创建 src 文件夹，并新增 js文件

1. common.js

   ```js
   exports.info = function (str) {
       document.write(str);
   }
   ```

2. utils.js

   ```js
   exports.add = function (a, b) {
       return a + b;
   }
   ```

3. main.js

   ```js
   const common = require('./common');
   const utils = require('./utils');
   common.info('Hello world!' + utils.add(100, 200));
   ```

![2021-10-11_163856](https://img.qinweizhao.com/2021/10/2021-10-11_163856.png)

## 三、打包

### 1、js

1. 在 webpack-sample 目录下创建配置文件 webpack.config.js

   ```js
   # 读取当前项目目录下src文件夹中的main.js（入口文件）内容，分析资源依赖，把相关的js文件打包，打包后的文件放入当前目录的dist文件夹下，打包后的js文件名为bundle.js
   const path = require("path"); //Node.js内置模块
   module.exports = {
       entry: './src/main.js', //配置入口文件
       output: {
           path: path.resolve(__dirname, './dist'), //输出路径，__dirname：当前文件所在路径
           filename: 'bundle.js' //输出文件
       }
   }
   ```

2. 命令行执行编译命令

   ```sh
   webpack #有黄色警告
   webpack --mode=development #没有警告
   #执行后查看bundle.js 里面包含了上面两个js文件的内容并进行了代码压缩
   ```

   也可以配置项目的npm运行命令，修改package.json文件

   ```json
   "scripts": {
       //...,
       "dev": "webpack --mode=development"
    }
   ```

   运行npm命令执行打包

   ```sh
   npm run dev
   ```

3. 在 index.html 中引入编译后的文件

   ```html
   <body>
       <script src="dist/bundle-npm.js"></script>
   </body>
   ```

### 2、css

1. 安装 style-loader 和 css-loader

   Webpack 本身只能处理 JavaScript 模块，如果要处理其他类型的文件，就需要使用 loader 进行转换。

   Loader 可以理解为是模块和资源的转换器。

   首先我们需要安装相关Loader插件，css-loader 是将 css 装载到 javascript；style-loader 是让 javascript 认识css

   ```sh
   npm install --save-dev style-loader css-loader 
   ```

2. 修改webpack.config.js

    ```js
    const path = require("path"); //Node.js内置模块
    module.exports = {
        //...,
        entry: './src/main.js', //配置入口文件
        output: {
            path: path.resolve(__dirname, './dist'), //输出路径，__dirname：当前文件所在路径
            filename: 'bundle-npm.js' //输出文件
        },
        module: {
            rules: [  
                {  
                    test: /\.css$/,    //打包规则应用到以css结尾的文件上
                    use: ['style-loader', 'css-loader']
                }  
            ]  
        }
    }
    ```

3. 在 src 文件夹创建 style.css

    ```css
    body{
        background:pink;
    }
    ```

4. 修改main.js

   在第一行引入style.css

   ```js
   require('./style.css');
   ```

5. 运行编译命令

   ```sh
   npm run dev
   ```

## 四、浏览器中查看 index.html

看看背景是不是变成粉色啦？

![2021-10-11_163710](https://img.qinweizhao.com/2021/10/2021-10-11_163710.png)
