# Git 生成 SSH key

## 一、检查 SSH keys 是否存在

```sh
ls -al ~/.ssh
```

![2021-08-14_171225](https://img.qinweizhao.com/2021/08/2021-08-14_171225.png)

## 二、生成 ssh key

```sh
# ssh-keygen -t rsa -C "youremail@example.com"
ssh-keygen -t rsa -C "qinweizhao1997@163.com"
```

![2021-08-14_172555](https://img.qinweizhao.com/2021/08/2021-08-14_172555.png)

生成成功会在 **~/** 下生成 **.ssh** 文件夹，打开 **id_rsa.pub** 中的内容即为 **key**。

![2021-08-14_172237](https://img.qinweizhao.com/2021/08/2021-08-14_172237.png)
