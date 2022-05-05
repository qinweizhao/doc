# Docker 安装 Elasticsearch 7.17.3

## 一、拉取镜像

```bash
docker pull elasticsearch:7.17.3
```

## 二、前置准备

```bash
mkdir -p /Users/weizhao/Docker/elasticsearch/config
mkdir -p /Users/weizhao/Docker/elasticsearch/data
echo "http.host: 0.0.0.0" >> /Users/weizhao/Docker/elasticsearch/config/elasticsearch.yml
```

**说明：**

/Users/weizhao/Docker/elasticsearch 为我本地的路径，具体存放位置根据自己安排创建。

## 三、运行

```bash
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms128m -Xmx1024m" \
-v /Users/weizhao/Docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /Users/weizhao/Docker/elasticsearch/data:/usr/share/elasticsearch/data \
-v /Users/weizhao/Docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.17.3
```

## 四、测试

浏览器访问：[localhost:9200](http://localhost:9200/)

![2022-05-06_013326](https://img.qinweizhao.com/2022/05/2022-05-06_013326.png)
