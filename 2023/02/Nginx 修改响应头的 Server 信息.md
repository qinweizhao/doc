# Nginx 修改响应头的 Server 信息

默认情况下，使用 Nginx 作为 web 服务器时，响应头中会显示 Nginx 以及使用的版本信息。网络安全中，为了不泄露敏感信息，一般会屏蔽服务器类型。

## 一、修改源码

文件路径：**nginx-x.x.x\src\http\ngx_http_header_filter_module.c**

说明：nginx-x.x.x 为 Nginx 源码文件夹。

```txt
static u_char ngx_http_server_string[] = "Server: nginx" CRLF;
static u_char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;
static u_char ngx_http_server_build_string[] = "Server: " NGINX_VER_BUILD CRLF;
```

修改为：

```txt
static u_char ngx_http_server_string[] = "Server: Qwz" CRLF;
static u_char ngx_http_server_full_string[] = "Server: " Qwz CRLF;
static u_char ngx_http_server_build_string[] = "Server: " Qwz CRLF;
```

效果：

![2023-02-13_203334](https://img.qinweizhao.com/2023/02/2023-02-13_203334.png)

## 二、使用插件

插件源码：[headers-more-nginx-module](https://github.com/openresty/headers-more-nginx-module) ，此插件可以在 Nginx 服务器中设置、添加和清除任意输出标头。

### 1、下载

文件目录：**/home/app** （环境为 Linux）

进入文件目录

```sh
cd /home/app/
```

下载插件

```sh
wget https://github.com/openresty/headers-more-nginx-module/archive/v0.33.tar.gz
```

解压

```sh
tar -zxvf v0.33.tar.gz
```

### 2、加载模块

进入 Nginx 源码目录

```sh
cd /home/app/nginx-1.12.2/
```

配置

```sh
./configure --prefix=/home/app/nginx112 --add-module=/home/app/headers-more-nginx-module-0.33
```

编译

```
make
```

执行结束后 **/home/app/nginx112** 即为编译后的 Nginx。

### 3、修改配置

```sh
vim /app/nginx112/conf/nginx.conf
```

添加配置(在http模块)

```sh
more_clear_headers 'Server';
```

### 4、重启nginx

```
/app/nginx112/sbin/nginx -s stop
/app/nginx112/sbin/nginx
```

>
直接使用reload可能会无效

![2023-02-13_202034](https://img.qinweizhao.com/2023/02/2023-02-13_202034.png)

## 三、补充

移除版本信息

如果隐藏 Nginx 的版本信息，可以在 Nginx 的配置文件中的 http 模块下添加配置：

```
server_tokens off;
```

配置文件如下：

![2023-02-13_201312](https://img.qinweizhao.com/2023/02/2023-02-13_201312.png)

但是返回给客户端的信息，还是存在Nginx服务器类型信息（虽然没有透露版本了）：

![2023-02-13_201428](https://img.qinweizhao.com/2023/02/2023-02-13_201428.png)
