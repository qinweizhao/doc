# GitHub加速

## 一、获取GitHub官方CDN地址

- [github.com](https://github.com.ipaddress.com/)

- [assets-cdn.github.com](https://github.com.ipaddress.com/assets-cdn.github.com)

- [github.global.ssl.fastly.net](https://fastly.net.ipaddress.com/github.global.ssl.fastly.net)

## 二、修改 Windows的 hosts 文件（C:\Windows\System32\drivers\etc）

```
140.82.112.4    github.com

185.199.108.153    assets-cdn.github.com
185.199.109.153    assets-cdn.github.com
185.199.110.153    assets-cdn.github.com
185.199.111.153    assets-cdn.github.com

199.232.5.194    github.global.ssl.fastly.net
```

### 三、以管理员方式打开cmd执行 `ipconfig /flushdns` 手动刷新系统DNS缓存

![2021-07-22_190021](https://img.qinweizhao.com/2021/07/2021-07-22_190021.png)

