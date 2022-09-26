# Blog 项目开发指南

## 一、前台

### 1、项目结构

```txt
blog-frontend-portal
├── module                      // 公共模板目录
│   ├── comment.ftl             // 比如：评论模板
│   ├── layout.ftl              // 比如：布局模板
├── source                      // 静态资源目录
│   ├── css                     // 样式目录
│   ├── images                  // 图片目录
│   ├── js                      // JS 脚本目录
│   └── plugins                 // 前端库目录
├── index.ftl                   // 首页
├── post.ftl                    // 文章页
├── post_xxx.ftl                // 自定义文章模板，如：post_diary.ftl。可在后台发布文章时选择。
├── archives.ftl                // 归档页
├── categories.ftl              // 分类目录页
├── category.ftl                // 单个分类的所属文章页
├── tags.ftl                    // 标签页面
├── tag.ftl                     // 单个标签的所属文章页
├── search.ftl                  // 搜索结果页
├── journals.ftl                // 内置页面：日志
├── 404.ftl                     // 404 页
├── 500.ftl                     // 500 页
├── README.md                   // README，说明
├── screenshot.png              // 主题预览图
├── settings.yaml               // 主题选项配置文件
└── theme.yaml                  // 主题描述文件
```

### 1、其他说明

> 1. 推荐使用 **VSCode** 开发，首先安装 **EasyLess** 插件来转换并压缩 `less` 文件，保存时会自动生成 `*.min.css` 文件，配置如下：

```txt
"less.compile": {
    "out": "./min/",
    "outExt": ".min.css",
    "compress": true,
    "sourceMap": false,
    "autoprefixer": "> 2%, last 2 versions, not ie 6-9"
  }
```

> 2. 安装 **JS & CSS Minifier** 插件来转换并压缩 js 文件，保存时会自动生成 `*.min.js` 文件，配置如下：

```txt
  "es6-css-minify.js": {
    "mangle": false,
    "compress": {
      "unused": true
    },
    "output": {
      "quote_style": 0
    },
    "warnings": true
  },
  // 保存时自动生成，no 为手动，可点击编辑器底部 Minify 按钮生成
  "es6-css-minify.minifyOnSave": "yes",
  "es6-css-minify.jsMinPath": "/source/js/min"
```

> 3. 转换并压缩 `ES6+` 代码（前 2 步里的 js 没有经过 babel 编译，只可用于开发环境）：

- 安装 `nodejs`；
- 主题目录下执行 `npm i` 安装依赖；
- 执行 `npm run build` 即可在相应目录生成可用于生产环境的 js 和 css 文件。

## 二、后台

### 环境准备

1. IDE：[IntelliJ IDEA](https://www.jetbrains.com/idea/download/) （推荐）
2. 工具：[Maven](https://maven.apache.org/)，[Lombok](https://projectlombok.org/) 插件
3. JDK：`8+`

### 运行项目

配置好配置文件后，直接运行 `Application` 主类（配合 IDE 运行）。
