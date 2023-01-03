# CentOS 7 端口被占用处理

## 一、查看指定端口对应的 PID

```sh
# netstat -anp |　grep "端口号"
netstat -anp |　grep "4682"
```

![2021-08-17_173434](https://img.qinweizhao.com/2021/08/2021-08-17_173434.png)

## 二、使用 PId 杀掉进程

```sh
# kill -9 "进程号"
kill -9 "1905"
```

![2021-08-17_173806](https://img.qinweizhao.com/2021/08/2021-08-17_173806.png)
