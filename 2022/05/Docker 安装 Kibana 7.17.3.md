# Docker 安装 Kibana 7.17.3

## 一、拉取镜像

```bash
docker pull kibana:7.17.3
```

## 二、运行

```sh
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://elasticsearch:9200 -p 5601:5601 --network elk-net -d kibana:7.17.
```

**说明：**

**--network elk-net：**指定为和 ES 同一个网络环境。

**http://elasticsearch:9200：**elasticsearch 为 ES 容器名。

### 三、测试

浏览器访问：http://localhost:5601

![2022-05-06_023600](https://img.qinweizhao.com/2022/05/2022-05-06_023600.png)
