# Docker 安装 ElK 7.17.3

## 一、Elasticsearch

### 1、拉取镜像

```bash
docker pull elasticsearch:7.17.3
```

### 2、前置准备

```bash
mkdir -p /Users/weizhao/Docker/elasticsearch/config
mkdir -p /Users/weizhao/Docker/elasticsearch/data
echo "http.host: 0.0.0.0" >> /Users/weizhao/Docker/elasticsearch/config/elasticsearch.yml
chmod -R 777 /mydata/elasticsearch/ 保证权限
```

**说明：**

/Users/weizhao/Docker/elasticsearch 为我本地的路径，具体存放位置根据自己安排创建。

### 3、运行

```bash
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms128m -Xmx1024m" \
-v /Users/weizhao/Docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /Users/weizhao/Docker/elasticsearch/data:/usr/share/elasticsearch/data \
-v /Users/weizhao/Docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.17.3
```

**注意：**

-e ES_JAVA_OPTS="-Xms128m -Xmx1024m" \ 测试环境下，设置 ES 的初始内存和最大内存，否则导致过大启动不了 ES。

### 4、测试

浏览器访问：[localhost:9200](http://localhost:9200/)

![2022-05-06_013326](https://img.qinweizhao.com/2022/05/2022-05-06_013326.png)

## 二、Kibana

### 1、拉取镜像

```bash
docker pull kibana:7.17.3
```

### 2、准备

查看ES 暴露的 IP

```
docker inspect 5aa
```

说明：5aa 为 ES 容器 id 的前三位。

![2022-05-06_022851](https://img.qinweizhao.com/2022/05/2022-05-06_022851.png)

## 3、运行

```bash
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://172.17.0.4:9200 -p 5601:5601 -d kibana:7.17.3
```

http://172.17.0.4:9200 为 ES 的地址。

### 4、测试

浏览器访问：[localhost:5601](http://localhost:5601/)
