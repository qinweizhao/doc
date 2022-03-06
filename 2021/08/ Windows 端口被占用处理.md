# Windows 端口被占用处理

## 一、查看指定端口对应的进程 Id

```bash
# netstat -ano | findstr "端口号"
netstat -ano | findstr "4682"
```

![2021-08-17_161403](https://img.qinweizhao.com/2021/08/2021-08-17_161403.png)

## 二、通过 Id 查找对应的进程名称（可选）

```bash
# tasklist |findstr "进程id号"
tasklist |findstr "23468"
```

![2021-08-17_161926](https://img.qinweizhao.com/2021/08/2021-08-17_161926.png)

## 三、使用进程 Id 或者进程名称杀掉进程

```bash
# taskkill /f /t /im "进程id或者进程名称"
taskkill /f /t /im "java.exe"
```

![2021-08-17_161953](https://img.qinweizhao.com/2021/08/2021-08-17_161953.png)
