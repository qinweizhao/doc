# CentOS 7 Yum 命令

## 一、前置了解

​  当 Yum执行安装或删除的命令时，Yum会查询数据库，有无这一软件包，如果有，则检查其依赖冲突关系。如果有，则会给出提示，询问是否要同时安装或删除有冲突的包。

## 二、Yum 查询

1. 查找软件包

   ```bash
   yum search ~
   ```

2. 列出所有可安装的软件包

   ```bash
   yum list
   ```

3. 列出所有可更新的软件包

   ```bash
   yum list updates
   ```

4. 列出所有已安装的软件包

   ```bash
   yum list installed
   ```

5. 列出所有已安装但不在Yum Repository 內的软件包

   ```bash
   yum list extras
   ```

6. 列出所指定软件包

   ```bash
   yum list ～
   ```

7. 使用YUM获取软件包信息

   ```bash
   yum info ～
   ```

8. 列出所有软件包的信息

   ```bash
   yum info
   ```

9. 列出所有可更新的软件包信息

   ```bash
   yum info updates
   ```

10. 列出所有已安裝的软件包信息

    ```bash
    yum info installed
    ```

11. 列出所有已安裝但不在Yum Repository 內的软件包信息

    ```bash
    yum info extras
    ```

12. 列出软件包提供哪些文件

    ```bash
    yum provides ~
    ```

## 三、清除Yum缓存

​  yum 会把下载的软件包和 header 存储在 cache 中，而不会自动删除。如果我们觉得它们占用了磁盘空间，可以使用 yum clean 指令进行清除，更精确的用法是 yum clean headers 清除 header ，yum clean packages 清除下载的 rpm 包，yum clean all 清除所有。

1. 清除缓存目录(/var/cache/yum)下的软件包

   ```bash
   yum clean packages
   ```

2. 清除缓存目录(/var/cache/yum)下的 headers

   ```bash
   yum clean headers
   ```

3. 清除缓存目录(/var/cache/yum)下旧的 headers

   ```bash
   clean oldheaders
   ```

4. 清除缓存目录（/var/cache/yum）下的软件包及旧的 headers

   ```bash
   yum clean, yum clean all (= yum clean packages; yum clean oldheaders)
   ```

## 四、例子

```bash
yum update 升级系统
yum install ～ 安装指定软件包
yum update ～ 升级指定软件包
yum remove ～ 卸载指定软件
yum grouplist 查看系统中已经安装的和可用的软件组，可用的可以安装
yum grooupinstall ～安装上一个命令显示的可用的软件组中的一个
yum grooupupdate ～更新指定软件组的软件包
yum grooupremove ～ 卸载指定软件组中的软件包
yum deplist ～ 查询指定软件包的依赖关系
yum list yum\* 列出所有以yum开头的软件包
yum localinstall ～ 从硬盘安装rpm包并使用yum解决依赖
```
