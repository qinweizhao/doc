# Docker 安装 Kibana 7.17.3

## 一、拉取镜像

```bash
docker pull kibana:7.17.3
```

## 二、准备

查看ES 暴露的 IP

```
docker inspect 5aa
```

说明：5aa 为 ES 容器 id 的前三位。

![2022-05-06_022851](https://img.qinweizhao.com/2022/05/2022-05-06_022851.png)

## 三、运行

```bash
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://172.17.0.4:9200 -p 5601:5601 -d kibana:7.17.3
```

http://172.17.0.4:9200 为 ES 的地址。

### 4、测试

浏览器访问：http://localhost:5601

![2022-05-06_023600](https://img.qinweizhao.com/2022/05/2022-05-06_023600.png)
