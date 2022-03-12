# frp 实现内网穿透

## 一、简介

> frp 是一个专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。

类似的应用还有：花生壳、nat123、ngrok、frp。frp 项目地址：https://github.com/fatedier/frp

## 二、说明

此次实践是将本地的 Linux 虚拟机通过腾讯的轻量云（系统为 Linux）暴露给 Internet。

## 三、下载

根据个人需求下载需要的版本，下载地址：https://github.com/fatedier/frp/releases。本次服务器和客户端都选择amd64版本。

![2022-03-12_162802](https://img.qinweizhao.com/2022/03/2022-03-12_162802.png)

下载完成，通过工具上传到服务器和本地虚拟机并解压。（上传到 /usr/local/）

文件说明：

```txt
frpc                    # 客户端二进制文件
frpc_full.ini           # 客户端配置文件完整示例
frpc.ini                # 客户端配置文件
frps                    # 服务端二进制文件
frps_full.ini           # 服务端配置文件完整示例
frps.in1                # 服务端配置文件
```

## 四、配置

### 1、server

配置文件详解：

```ini
[common]                        # 通用配置段
bind_addr = 0.0.0.0             # 绑定的IP地址，支持IPv6，不指定默认0.0.0.0；
bind_port = 7000                # 服务端口；
bind_udp_port = 7001            # 是否使用udp端口，不使用删除或注释本行；
kcp_bind_port = 7000            # 是否使用kcp协议，不使用删除或注释本行；
# proxy_bind_addr = 127.0.0.1   # 代理监听地址，默认和bind_addr相同；

# 虚拟主机
vhost_http_port = 80            # 是否启用虚拟主机，端口可以和 bind_port 相同；
vhost_https_port = 443
vhost_http_timeout = 60         # 后端虚拟主机响应超时时间，默认为60s；

# 开启 frps 仪表盘可以检查 frp 的状态和代理的统计信息。
dashboard_addr = 0.0.0.0        # frps仪表盘绑定的地址；
dashboard_port = 7500           # frps仪表盘绑定的端口；
dashboard_user = admin          # 访问frps仪表盘的用户；     
dashboard_pwd = admin           # 密码；
assets_dir = ./static           # 仪表盘页面文件目录，只适用于调试；

# 日志配置文件
log_file = ./frps.log           # 日志文件,不指定日志信息默认输出到控制台；
log_level = info                # 日志等级，可用等级“trace, debug, info, warn, error”；
log_max_days = 3                # 日志保存最大保存时间；

token = 12345678                # 客户端与服务端通信的身份验证令牌

heartbeat_timeout = 90          # 心跳检测超时时间，不建议修改默认配置，默认值为90；？

# 指定允许客户端使用的端口范围，未指定则没有限制；
allow_ports = 2000-3000,3001,3003,4000-50000

max_pool_count = 5              # 每个客户端连接服务端的最大连接数；
max_ports_per_client = 0        # 每个客户端最大可以使用的端口，0表示无限制

authentication_timeout = 900    # 客户端连接超时时间（秒），默认为900s；

subdomain_host = frps.com       # 自定义子域名，需要在dns中将域名解析为泛域名；

tcp_mux = true                  # 是否使用tcp复用，默认为true；
                                # frp只对同意客户端的连接进行复用；
```

本次配置示例：

```ini
[common]
bind_addr = 0.0.0.0
bind_port = 7000

# Dashboard configuration
dashboard_addr = 0.0.0.0
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin

# logs
log_file = ./frps.log
log_level = info
log_max_days = 3

# auth token
token = 12345678

max_pool_count = 5
max_ports_per_client = 0
authentication_timeout = 900
tcp_mux = true
```

### 2、client

配置文件详解

```csharp
[common]                        # 通用配置段

server_addr = 0.0.0.0           # server的IP地址；支持IPv6
server_port = 7000              # server的端口；

# 如果要通过http或socks5代理连接frps，可以在此处或在全局环境变量中设置代理，只支持tcp协议；
# http_proxy = http://user:passwd@192.168.1.128:8080
# http_proxy = socks5://user:passwd@192.168.1.128:1080

# 客户端日志
log_file = ./frpc.log       # 指定日志文件；
log_level = info            # 指定日志等级；
log_max_days = 3

token = 12345678            # 客户端与服务端通信的身份验证令牌

# 设置管理地址，用于通过http api控制frpc的动作，如重新加载；
admin_addr = 127.0.0.1
admin_port = 7400
admin_user = admin
admin_passwd = admin

pool_count = 5              # 初始连接池的数量，默认为0；

tcp_mux = true              # 是否启用tcp复用，默认为true；

user = your_name            # frpc的用户名，用于区别不用frpc的代理；

login_fail_exit = true      # 首次登录失败时退出程序，否则连续重新登录到frps；

protocol = tcp              # 用于连接服务器的协议，支持tcp、kcp、websocket;

dns_server = 8.8.8.8        # 为frp 客户端指定一个单独的DNS服务器；

# start = ssh,dns           # 要启用的代理的名字，默认为空表示所有代理；

# 心跳检查
# heartbeat_interval = 30   # 失败重试次数
# heartbeat_timeout = 90    # 超时时间

# 配置示例
[ssh]                       # 代理配置段名称，如果配置user=your_name,则显示为your_name.ssh；
type = tcp                  # 协议默认tcp,可选tcp,udp,http,https,stcp,xtcp;
local_ip = 127.0.0.1        # 本地地址
local_port = 22             # 本地端口
use_encryption = false      # 是否加密服务端和客户端的通信信息，默认为不加密；
use_compression = false     # 是否开启压缩，默认不开启；
remote_port = 6001          # 在服务器端开启的远程端口；
# 负载均衡配置
group = test_group          # 负载均衡组名，会将同一组内的客户端进行负载；
group_key = 123456          # 负载均衡组密钥； 
# web示例
[web01]
type = http                 # 使用http
local_ip = 127.0.0.1        
local_port = 80
use_encryption = false
use_compression = true
http_user = admin           # 访问web01页面启用认证，用户名admin
http_pwd = admin            # 密码
subdomain = web01           # 子域名，需要服务端配置了subdomain_host参数；
custom_domains = web02.example.com # web01的域名，和subdomain二选一
locations = /,/pic          # 指定用于路由的URL前缀；
host_header_rewrite = example.com   # 配置http包头域名重写;
header_X-From-Where = frp           # 添加包头信息X-From-Where: frp；
```

本次配置示例：

```ini
[common]
server_addr = 42.192.49.72
server_port = 7000
log_file = ./frpc.log
log_level = info
log_max_days = 3
token = 12345678
pool_count = 5
tcp_mux = true
login_fail_exit = true
protocol = tcp

[ssh]
type = tcp
local_ip = 192.168.79.79
local_port = 22
remote_port = 2222
```

注意，除了 start 之外，[common] 部分中的参数不会被修改。

## 五、启动

### 1、server

默认前台启动为提供后台允许参数：

当前目录为 **/usr/local/frp/**

```sh
./frps -c frps.ini
```

### 2、client

```sh
./frpc -c frpc.ini
```

此次实践结束。即可验证。

### 3、注册为系统服务（可选）

将 frp 注册为系统服务，这样就可以使用systemctl来控制启动。

此处操作均为 frps，frpc 同理。

 ```sh
 vim /lib/systemd/system/frps.service
 ```

在 frps.service 里写入以下内容：

```
[Unit]
Description=fraps service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
#启动服务的命令（此处写你的frps的实际安装目录）
ExecStart=/usr/local/frp/frps -c /usr/local/frp/frps.ini

[Install]
WantedBy=multi-user.target
```

操作完成后 frps 的操作方法：

```
启动：systemctl start frps
重启：systemctl restart frps
停止：systemctl stop frps
自启动：systemctl enable frps
查看日志：systemctl status frps
```

### 4、可能出现的问题

报错：

```
Warning: frps changed on disk. Run 'systemctl daemon-reload' to reload units.
```

解决方案：

```sh
systemctl daemon-reload
```

重启即可：

```sh
systemctl  start  frps
```

## 六、结果

![2022-03-12_170313](https://img.qinweizhao.com/2022/03/2022-03-12_170313.png)

![2022-03-12_170348](https://img.qinweizhao.com/2022/03/2022-03-12_170348.png)
