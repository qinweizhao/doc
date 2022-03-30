# Spring Boot 整合 Freemarker

## 一、导包

`pom.xml`

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-freemarker</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.7.22</version>
        </dependency>
    </dependencies>
```

## 二、配置

`application.yml`

```yaml
server:
  port: 8080
spring:
  freemarker:
    suffix: .ftl
    cache: false
    charset: UTF-8
```

## 三、准备

### 1、后端

controller/`IndexController`

```java
/**
 * @author qinweizhao
 * @since 2022/3/30
 */
@Controller
@Slf4j
public class IndexController {


    @GetMapping(value = {"", "/"})
    public ModelAndView index(HttpServletRequest request) {
        ModelAndView mv = new ModelAndView();

        User user = (User) request.getSession().getAttribute("loginUser");
        if (ObjectUtil.isNull(user)) {
            mv.setViewName("redirect:/user/login");
        } else {
            mv.setViewName("page/index");
            mv.addObject(user);
        }

        return mv;
    }
}
```

controller/`UserController`

```java
/**
 * @author qinweizhao
 * @since 2022/3/30
 */
@Controller
@RequestMapping("/user")
@Slf4j
public class UserController {
    @PostMapping("/login")
    public ModelAndView login(User user, HttpServletRequest request) {
        ModelAndView mv = new ModelAndView();
        mv.addObject(user);
        mv.setViewName("redirect:/");
        request.getSession().setAttribute("loginUser", user);
        return mv;
    }

    @GetMapping("/login")
    public ModelAndView login() {
        return new ModelAndView("page/login");
    }
}
```

model/`User.java`

```java
/**
 * @author qinweizhao
 * @since 2022/3/30
 */
@Data
public class User {

    /**
     * 用户名
     */
    private String username;

    /**
     * 密码
     */
    private String password;
}
```

### 2、前端

common/`head.ftl`

```ftl
<head>
	<meta charset="UTF-8">
	<meta name="viewport"
	      content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
	<meta http-equiv="X-UA-Compatible" content="ie=edge">
	<title>spring-boot-freemarker</title>
</head>
```

page/`index.ftl`

```ftl
<!doctype html>
<html lang="en">
<#include "../common/head.ftl">
<body>
<div id="app" style="margin: 20px 20%">
	欢迎登录，${loginUser.username} ！
</div>
</body>
</html>
```

page/`login.ftl`

```ftl
<!doctype html>
<html lang="en">
<#include "../common/head.ftl">
<body>
<div id="app" style="margin: 20px 20%">
	<form action="/user/login" method="post">
		用户名<input type="text" name="username" placeholder="用户名"/>
		密码<input type="password" name="password" placeholder="密码"/>
		<input type="submit" value="登录">
	</form>
</div>
</body>
</html>
```

项目结构：

![2022-03-30_193031](https://img.qinweizhao.com/2022/03/2022-03-30_193031.png)

## 四、测试

启动项目访问 **localhost:8080** 即可。

## 五、附加
FreeMarker 常用技巧

### global

全局赋值语法，利用它定义的变量在模板的所有位置均可用

```html
<#global name=value>
```

### assign

局部变量

```html
<#assign name=value>   
<#assign name1=value1 name2=value2 ... nameN=valueN> 
```

### setting

用来设置整个系统的一个环境

```html
<#setting name=value>
```

### import

导入模板或宏，可以有自己的命名空间

```html
<#import "/libs/mylib.ftl" as my>
<@my.copyright date="1999-2002"/> 
<#-- "my"在freemarker里被称作namespace --> 
```

### include

和import 类似都可以引入模板。<#include filename options> ，options 有2个参数，encoding="GBK" 编码格式 parse=true 是否作为ftl语法解析，默认是 true，false 就是以文本方式引入。注意在 ftl 文件里布尔值都是直接赋值

```html
<#include "/common/header.ftl">
```

### macro (宏)

macro 也就是宏，主要用于抽离公用模板，做代码复用。

```javascript
// 定义一个公共模板，可定义参数，也可定义有默认值的参数
<#--公共header，接受type和id参数，其中id默认值为0，非必传-->
<#macro header type id=0>
	<head>
		<script type="text/javascript" src="/static/js/common.js"></script>
		// nested 相当于插槽，外部填充的内容会渲染到这里
		<#nested>
	</head>
</#macro>

// 在其他 ftl 模板中使用
<#include "header.ftl">
<@header type="post"/>
  
// 如果需要设置命名空间，需要使用 import 代替 include
<#import "header.ftl" as headInfo>
<@headInfo.header type="post"/>
```

### switch

```html
<#switch typeId>  
  <#case 1>
    <#assign typeName='种类1'>
  	<#break>
  <#case 1>
    <#assign typeName='种类2'>
  	<#break>
  <#default>
    <#assign typeName=''>
</#switch> 
```

### 比较运算符

```plain
=(==)    判断两个值是否相等
!=       不相等
>(gt)    大于
>=(gte)  大于等于
<(lt)    小于
<=(lte)  小于等于
```

### 三目运算符

<#assign theValue = (temp =="default")?string('true','false') >

从 FreeMarker 2.3.23 版本开始废弃：建议使用 [?then("yes", "no")](http://freemarker.foofun.cn/ref_builtins_boolean.html#ref_builtin_then) 来替代

### 默认值

- ${value!666}，当 value 为空使用666；
- ${var?default("hello world")?html} ，var 是 null 时将被 hello world 替代
- (product.color)!"red" 处理 product 或者 color 为 miss value 的情况；
- product.color!"red" 将只处理 color 为miss value 的情况。

### 非空判断??

- 例如：variable?? 如果该变量存在，返回 true，否则返回false；
- product.color?? 将只测试 color 是否为 null，(product.color)?? 将测试 product 和 color 是否为 null；
- <#if mouse?exists>Mouse found<#else> ，也可直接 ${mouse?if_exists}) 输出布尔值。

- https://blog.csdn.net/iteye_1938/article/details/82651924

### 判断2个值是否相等

== 或 =，所有逻辑运算符都只适用于布尔值

### 定义数组

```html
<#assign configArr = [['北京','010'],['上海','021']]>
<#list configArr as it>
	${it[0]} ${it[1]}
</#list>
```

- ${var} 不能直接输出数字或者字符串外的类型，需要转化为字符串才能输出，使用**xxx?string**转化

### list 中获取索引值

```html
<#list projects as project>${project_index}</#list>
```

### 不解析 html

${htmlstr?html}

### #{ expression } 数字计算

\#{ expression;format} 按格式输出数字 format 为 M 和 m

M 表示小数点后最多的位数，m 表示小数点后最少的位数如 #{121.2322;m2M2} 输出 121.23

### 数字循环

1..5 表示从 1 到 5 ，原型number..number

### 对浮点取整数

${123.23?int} 输出 123

### 生成随机数

可以通过时间来计算一个，下面举个从固定数组中取颜色的例子

```html
// 举个列表通过随机值从数组中取颜色的例子
<ul class="joe_detail__friends">
    <#assign colors=["#F8D800", "#0396FF", "#EA5455", "#7367F0", "#32CCBC", "#F6416C", "#28C76F", "#9F44D3", "#F55555", "#736EFE", "#E96D71", "#DE4313", "#D939CD", "#4C83FF", "#F072B6", "#C346C2", "#5961F9", "#FD6585", "#465EFB", "#FFC600", "#FA742B", "#5151E5", "#BB4E75", "#FF52E5", "#49C628", "#00EAFF", "#F067B4", "#F067B4"]>
    <#assign nextRandom = .now?string["HHmmssSSS"]?number>
    <@linkTag method="list">
       <#list links as link>
         	 <#--  colors数组长度为28  -->
           <li class="joe_detail__friends-item">
              	<a class="contain" href="${link.url!}" target="_blank" style="background:${colors[nextRandom % colors?size]}" rel="noopener noreferrer">
                  <span style="font-weight:bold;" class="title">${link.name!}</span>
                  <div class="content">
                     <div class="desc" title="${link.description!}">${link.description!}</div>
                        <img width="40" height="40" class="logo" src="${link.logo!}" alt="${link.name!}">
                   </div>
               	</a>
           </li>
           <#assign nextRandom = nextRandom * 10 % 28>
       </#list>
    </@linkTag>              
</ul>
```

### JS 获取 Halo 博客主题配置

注意使用 string 或 js_string 进行转义

```javascript
<#--  取出主题配置  -->
<script id="theme-config-getter">
  var ThemeConfig = {};
  <#list settings?keys as key>
    <#assign valueString = settings[key]?string>
    <#assign isNeeded = key?index_of('custom_')==-1 && valueString?index_of('<script')==-1 && valueString?index_of('<link')==-1>
    <#if isNeeded>
      var field = '${key}';
      var value = '${valueString?js_string}';
      value = value.replace(/</g,"&lt;").replace(/>/g, "&gt;");
      if(/^(true|false)$/.test(value)) {
        value = JSON.parse(value);
      }
      if(/^\d+$/.test(value)) {
        value = Number(value);
      }
      ThemeConfig[field] = value;
    </#if>
  </#list>
  window.onload = function () {
    // 页面加载完后移除加载配置用的script标签
    document.head.removeChild(document.querySelector('#theme-config-getter'));
    console.log("主题配置：", ThemeConfig);
  }
</script>

// 第二种
<!-- 定义可变属性，会根据页面的改变而变化 -->
<script type='text/javascript'>
	/* <![CDATA[ */
    var PageAttr = {
        "metas": {
			<#if metas??>
				<#list metas?keys as key>
					"${key}": "${metas['${key}']}",
				</#list>
			</#if>
        },
    }
	/* ]]> */
</script>
```

### 判断字符串长度

可以结合 Freemarker 的 ?default('') 设置默认值，然后使用 trim 函数去掉字符串首尾的空白字符，利用 length 函数获取字符串的长度。如果长度大于 0，则执行 <#if> 语句内部的代码。

```html
<#if sysInfo.version?default("")?trim!=''>
    <div>系统版本：${sysInfo.version?trim}</div>
</#if>
```

### 判断字符串中是否包含某个字符

```html
 <#if "a,b,c"?contains("a")>checked</#if>
 <#if "a,b,c"?index_of("a") > -1>checked</#if>
 <#if 135?string?index_of("3") > -1>checked</#if>
```

### 时间格式化

Halo 博客中有个专门的宏 **timeline**

```html
<@global.timeline datetime=journal.createTime />
// 判断今天、昨天（可能有bug）
<#assign diff = (journal.createTime?long / 86400000)?round - (.now?long / 86400000)?round />
<p class="joe_journal_date">
  <i class="joe-font joe-icon-feather"></i>发布于 
  <#if diff==0>
  今天 ${journal.createTime?string('HH:mm')}
  <#elseif diff==-1>
  昨天 ${journal.createTime?string('HH:mm')}
  <#else>
  ${journal.createTime?string('yyyy年MM月dd日 HH:mm')}
  </#if>
</p>
```

### 动态获取对象属性

有时候需要在模板里动态获取对象的属性，可以像下面这样做

```javascript
// 以这种形式来获取：${对象名["${动态属性}"]}，但要注意非空判断，可以使用escape
<#list categories?sort_by("postCount") ? reverse as category>
	<li class="item">
    <a class="link" href="${category.fullPath!}" title="${category.name!}">
      <figure class="inner">
        <span class="views">${category.postCount!}</span>
        <#escape x as x!"">
        	<#assign cover=settings['hot_cover'+(category_index+1)]>
        </#escape>
        <img width="100%" height="120" class="image lazyload" data-src="${cover!}" src="https://cdn.jsdelivr.net/gh/qinhua/halo-theme-joe2.0@master/source/img/lazyload.jpeg" alt="${category.name!}">
        <figcaption class="title">${category.name!}</figcaption>
      </figure>
    </a>
  </li>
</#list>
```

### split 分割字符串

```javascript
<#list "张三三,李思思,,王强,柳树,诸葛正我"?split(",") as name>
  "${name}"
</#list>
```

### 字符串操作

- ?index_of('xxx')
- ?contains('xxx')
- ?last_index_of('xxx')
- ?starts_with('xxx')
- ?ends_with('xxx')
- ?trim
- ?substring(0,2)
- ?replace('a','b')
- ?word_list
- ?capitalize
- [FreeMarker内置命令(字符串命令)](https://blog.csdn.net/liuyong0818/article/details/8276703?_t=t)

### 正则模式

使用 replace 时第三个参数传 'r' 代表开启正则模式

**?replace('[^\\d]','','ri')**

### 

### 参考

[FreeMarker官方文档](http://freemarker.foofun.cn/)

[freemarker页面语法](https://www.iteye.com/blog/jiangsha-372307)

[js获取freemarker变量的值](https://www.cnblogs.com/chn58/p/7084819.html)

[freemarker 类型转换](https://www.cnblogs.com/haoerlv/p/7234612.html)

[FreeMarker js 获取后台设置的request、session](https://blog.csdn.net/diaodongai6794/article/details/101581563)

[block/slot支持](https://github.com/halo-dev/halo/pull/1295)

[freemarker迭代list、map等常规操作，将数据放到模板中](https://www.cnblogs.com/sjxbg/p/9910464.html)

[freemarker空值的多种处理方法](https://blog.csdn.net/leijuly/article/details/4086568)

##  

>代码地址：
>
>https://github.com/qinweizhao/qwz-integration/spring-boot-freemarker

