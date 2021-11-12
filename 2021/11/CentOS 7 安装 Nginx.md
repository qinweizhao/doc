# CentOS 7 安装 Nginx

## 一、准备

##  1、gcc

>gcc 是 Linux 的编译器，可以编译 C、C++、Ada、Object C 和 Java 等语言。

1.  查看gcc版本

   ```bash
   gcc -v
   ```

2. gcc 安装命令

   ```bash
   yum -y install gcc
   ```

## 2、pcre 和 pcre-devel

> nginx 的 http 模块使用 pcre 来解析正则表达式。

```bash
yum install -y pcre pcre-devel
```

## 3、zlib

>nginx使用zlib对http包的内容进行gzip。

```bash
yum install -y zlib zlib-devel
```

## 4、openssl

> openssl用于数据链路通信安全加密。

```bash
yum install -y openssl openssl-devel
```

## 5、nginx 压缩包

1. 获取稳定版本下载链接。官网地址：http://nginx.org/en/download.html

2. 下载 nginx

   ```bash
   wget http://nginx.org/download/nginx-1.20.1.tar.gz  
   ```
   

## 二、安装

### 1、解压

```bash
tar -zxvf  nginx-1.20.1.tar.gz
```

## 2、切换到解压目录，编译并安装。

```bash
# 不需要https模块的， 这里只输入./configure即可
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module

# 编译
make

# 安装
make install
```

## 3、启动

```bash
# 启动
./nginx -s start

# 刷新配置
./nginx -s reload

# 停止nginx
./nginx -s stop

# 查看nginx是否启动成功
ps -ef | grep nginx
```

## 三、配置开机自启

>官方：https://www.nginx.com/resources/wiki/start/topics/examples/redhatnginxinit/

### 1、在 /etc/init.d 下创建文件 nginx

```conf
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

# 特别注意，这里要调整你存放Nginx的目录
nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)

# 特别注意，这里要调整你存放Nginx的目录
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -n "$user" ]; then
      if [ -z "`grep $user /etc/passwd`" ]; then
         useradd -M -s /bin/nologin $user
      fi
      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
      for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
              value=`echo $opt | cut -d "=" -f 2`
              if [ ! -d "$value" ]; then
                  # echo "creating" $value
                  mkdir -p $value && chown -R $user $value
              fi
          fi
       done
    fi
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $prog -HUP
    retval=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

### 2、赋值文件执行权限

```bash
chmod a+x /etc/init.d/nginx
```

### 3、将nginx服务加入chkconfig管理列表

```bash
chkconfig --add /etc/init.d/nginx
```

### 4、设置开机自启
```bash
chkconfig nginx on
```

### 5、其他操作命令 

```bash
# 启动nginx
service nginx start

# 停止nginx
service nginx stop

# 重启nginx
service nginx restart
```



