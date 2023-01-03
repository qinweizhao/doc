# CentOS 7 安装 Kibana 7.14

## 一、下载 Kibana

1. 获取下载链接

   [官网下载地址](https://www.elastic.co/cn/downloads/kibana)

2. 下载压缩包

   ```sh
   wget https://artifacts.elastic.co/downloads/kibana/kibana-7.14.1-linux-x86_64.tar.gz
   ```

3. 解压（到上层目录）

   ```sh
   tar -zxvf kibana-7.14.1-linux-x86_64.tar.gz -C ../
   ```

4. 重命名

   ```sh
   # 返回上层目录
   cd ..
   
   # 重命名
   mv kibana-7.14.1-linux-x86_64/ kibana/
   ```

## 二、修改配置

Kibana 的配置文件为安装目录的 **config/kibana.yml**

```sh
# 进入 kibana 的安装目录
cd /usr/local/kibana/config

# 编辑 yml 配置文件
vim kibana.yml
```

在配置文件中增加：

```yaml
# 端口
server.port: 5601

# ip
server.host: "192.168.79.79"

# es 地址
elasticsearch.hosts: ["http://192.168.79.79:9200"]

# kibana 索引
kibana.index: ".kibana"

# kibana默认文字是英文，变更成中文
i18n.locale: "zh-CN"
```

## 五、启动 Kibana 7.14

1. 启动

   非 root 用户（推荐）

   ```sh
   # 在 bin 目录下
   ./kibana
   ```

   root 用户

   ```sh
   ./kibana --allow-root
   ```

2. 在浏览器中打开 <http://192.168.79.79:5601> 检查是否成功

   ![2021-09-12_205226](https://img.qinweizhao.com/2021/09/2021-09-12_205226.png)
