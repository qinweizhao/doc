# Nginx 服务器 SSL 证书安装部署

## 一、证书安装

### 1、下载并解压缩证书文件包到本地目录。解压缩后，可获得相关类型的证书文件

![2021-07-21_192413](https://img.qinweizhao.com/2021/07/2021-07-21_192413.png)

### 2、将 Nginx 中的证书文件和私钥文件从本地目录拷贝到 Nginx 服务器的 `/usr/local/nginx/conf` 目录（此处为 Nginx 默认安装目录，请根据实际情况操作）下

![2021-07-21_192537](https://img.qinweizhao.com/2021/07/2021-07-21_192537.png)

![2021-07-21_193005](https://img.qinweizhao.com/2021/07//2021-07-21_193005.png)

### 3、编辑 Nginx 根目录下的 `conf/nginx.conf` 文件。修改内容如下

 ```sh
 vim /usr/local/nginx/conf/nginx.conf
 ```

#### 说明

- 由于版本问题，配置文件可能存在不同的写法。例如：Nginx 版本为 `nginx/1.15.0` 以上请使用 `listen 443 ssl` 代替 `listen 443` 和 `ssl on`。

``` conf
server {
 # SSL 访问端口号为 443
 listen 443 ssl; 
 # 填写绑定证书的域名
 server_name www.qinweizhao.com; 
 # 证书文件名称
 ssl_certificate 1_www.qinweizhao.com_bundle.crt; 
 # 私钥文件名称
 ssl_certificate_key 2_www.qinweizhao.com.key; 
 ssl_session_timeout 5m;
 # 请按照以下协议配置
 ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 
 # 请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
 ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
 ssl_prefer_server_ciphers on;
 location / {
    # 网站主页路径。此路径仅供参考，具体请您按照实际目录操作。
    root html; 
    index  index.html index.htm;
}
}
```

### 4、在 Nginx 根目录下，通过执行以下命令验证配置文件问题

 ```sh
 ./sbin/nginx -t
 ```

### 5、重启 Nginx，即可使用 `https://www.qinweizhao.com` 进行访问

 ```sh
 ./sbin/nginx -s reload
 ```

## 二、HTTP 自动跳转 HTTPS 的安全配置（可选）

### 1、根据实际需求，选择以下配置方式

- 在页面中添加 JS 脚本。
- 在后端程序中添加重定向。
- 通过 Web 服务器实现跳转。
- Nginx 支持 rewrite 功能。若您在编译时没有去掉 pcre，您可在 HTTP 的 server 中增加 `return 301 https://$host$request_uri;`，即可将默认80端口的请求重定向为 HTTPS。修改如下内容：

```conf
server {
    # SSL 访问端口号为 443
    listen 443 ssl; 
    # 填写绑定证书的域名
    server_name www.qinweizhao.com; 
    # 证书文件名称
    ssl_certificate 1_www.qinweizhao.com_bundle.crt; 
    # 私钥文件名称
    ssl_certificate_key 2_www.qinweizhao.com.key; 
    ssl_session_timeout 5m;
    # 请按照以下协议配置
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; 
    # 请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
    ssl_prefer_server_ciphers on;
    location / {
        # 网站主页路径。此路径仅供参考，具体按照实际目录操作。
        root html; 
        index  index.html index.htm;
    }
}
server {
    listen 80;
    # 填写绑定证书的域名
    server_name www.qinweizhao.com; 
    # 把http的域名请求转成https
    return 301 https://$host$request_uri; 
}
```

### 2、若修改完成，重启 Nginx。即可使用 `http://www.qinweizhao.com` 进行访问

 ```sh
 ./sbin/nginx -s reload
 ```
