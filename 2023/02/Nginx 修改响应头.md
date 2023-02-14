网络安全中，为了不泄露敏感信息，一般会屏蔽服务器类型(当然，这时最基础的步骤)。

# 1、Nginx移除版本信息

如果想关掉 Nginx 关于 OS 和 Nginx 版本的信息，可以简单得再 Nginx 上设置一个：

```
server_tokens off;
```

配置类似于：

```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

http {
    keepalive_timeout   65;
    types_hash_max_size 2048;
	server_tokens off;
    server {
		... ...

    }
}

```

但是返回给客户端的信息，还是存在Nginx服务器类型信息（虽然没有透露版本了）：

Server ：nginx

![2023-02-13_201428](https://img.qinweizhao.com/2023/02/2023-02-13_201428.png)


# 2、完全移除Server HTTP 头

组件：headers-more-nginx-module

GitHub：


## 2.1 下载

```
# 举例目录/app/tools
cd /app/tools/
#下载插件
wget https://github.com/openresty/headers-more-nginx-module/archive/v0.33.tar.gz
#解压
tar -zxvf v0.33.tar.gz
```

## 2.2 加载模块

```
# 查看安装参数命令(取出：configure arguments:)
/app/nginx/sbin/nginx -V
# 在nginx资源目录编译
cd /app/nginx-1.12.2/
# 将上面取出的configure arguments后面追加 --add-module=/app/tools/headers-more-nginx-module-0.33
./configure --prefix=/app/nginx112 --add-module=/app/tools/headers-more-nginx-module-0.33
# 编辑，切记没有make install
make
# 备份
cp /app/nginx112/sbin/nginx /app/nginx112/sbin/nginx.bak 
# 覆盖(覆盖提示输入y)
cp -f /app/nginx-1.12.2/objs/nginx /app/nginx112/sbin/nginx
```

## 2.3 修改配置

```
vim /app/nginx112/conf/nginx.conf
# 添加配置(在http模块)
more_clear_headers 'Server';
```

>  上面配置只是将http响应头中的Server:nginx/1.12.2清除。
>
> [openresty/headers-more-nginx-module: Set, add, and clear arbitrary output headers in NGINX http servers (github.com)](https://github.com/openresty/headers-more-nginx-module)
> 支持添加、修改、清除响应头的操作，

## 2.4 重启nginx

```
/app/nginx112/sbin/nginx -s stop
/app/nginx112/sbin/nginx
```

> 
 直接使用reload可能会无效

![2023-02-13_202034](https://img.qinweizhao.com/2023/02/2023-02-13_202034.png)
