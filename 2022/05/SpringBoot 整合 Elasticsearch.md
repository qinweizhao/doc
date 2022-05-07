# SpringBoot 整合 Elasticsearch

ES 7.15 版本之后，官方将它的高级客户端 RestHighLevelClient 标记为弃用状态。同时推出了全新的 Java API 客户端 Elasticsearch Java API Client，该客户端也将在 Elasticsearch8.0 及以后版本中成为官方推荐使用的客户端。

## 一、导包

```
        <dependency>
            <groupId>co.elastic.clients</groupId>
            <artifactId>elasticsearch-java</artifactId>
            <version>7.17.3</version>
        </dependency>

        <dependency>
            <groupId>jakarta.json</groupId>
            <artifactId>jakarta.json-api</artifactId>
            <version>2.0.1</version>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.12.3</version>
        </dependency>
```

将版本统一：

```xml
    <properties>
        <elasticsearch.version>7.17.3</elasticsearch.version>
    </properties>
```

## 二、配置

```java
@Configuration
public class ElasticsearchConfig {

    @Bean
     ElasticsearchClient elasticsearchClient() {
        // 创建低级客户端
        RestClient restClient = RestClient.builder(
                new HttpHost("localhost", 9200)).build();
        // 使⽤Jackson映射器创建传输层
        ElasticsearchTransport elasticsearchTransport = new RestClientTransport(restClient, new JacksonJsonpMapper());
        // 创建APl客户端
        return new ElasticsearchClient(elasticsearchTransport);
    }
}
```

## 三、使用

### 1、索引

#### 1. 创建

```java
    @Test
    void createIndex() throws IOException {
        CreateIndexResponse qwz_index = client.indices().create(c -> c.index("qwz_index"));
        Boolean acknowledged = qwz_index.acknowledged();
        System.out.println("acknowledged = " + acknowledged);
    }
```

#### 2. 查询

```java
    @Test
    void getIndex() throws IOException {
        GetIndexResponse qwz_index = client.indices().get(g -> g.index("qwz_isndex"));
        Map<String, IndexState> resultMap = qwz_index.result();
        String result = JSON.toJSONString(resultMap);
        System.out.println("result = " + result);
    }
```

**注意：**

**如果索引不存在将会报异常。**

#### 3. 删除

```java
    @Test
    void deleteIndex() throws IOException {
        DeleteIndexResponse qwz_index = client.indices().delete(d -> d.index("qwz_index"));
        boolean acknowledged = qwz_index.acknowledged();
        System.out.println(acknowledged);
    }
```

### 2、文档

Account.java

```java
/**
 * @author qinweizhao
 * @since 2022/5/7
 */
@Data
public class Account {
    private int account_number;
    private int balance;
    private String firstname;
    private String lastname;
    private int age;
    private String gender;
    private String address;
    private String employer;
    private String email;
    private String city;
    private String state;
}
```

#### 1. 新增

```java
    @Test
    void addDoc() throws IOException {
        Account account = new Account();
        account.setFirstname("q");
        account.setLastname("wz");
        account.setGender("M");
        account.setAddress("zg");
        account.setCity("hn");

        IndexResponse bank = client.index(i -> i.index("bank").document(account));
      
//        String s ="{'level': 'warn'}";
//        StringReader stringReader = new StringReader(s);
//        IndexResponse json = client.index(i -> i.index("bank").withJson(stringReader));

        ShardStatistics shards = bank.shards();
        Number successful = shards.successful();
        System.out.println("successful = " + successful);
    }
```

结果：

```json
{
  "_index" : "bank",
  "_type" : "account",
  "_id" : "0SpOn4ABSWOoZKugNd8r",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1001,
  "_primary_term" : 4
}
```

**也可作为更新：**

```java
    @Test
    void addDoc() throws IOException {
        Account account = new Account();
        account.setFirstname("q");
        account.setLastname("wz");
        account.setGender("M");
        account.setAddress("zg");
        account.setCity("hn");

        IndexResponse bank = client.index(i -> i.index("bank").document(account));

        ShardStatistics shards = bank.shards();
        Number successful = shards.successful();
        System.out.println("successful = " + successful);
        account.setFirstname("qin");
        IndexResponse bank2 = client.index(i -> i.index("bank").id(bank.id()).document(account));
        System.out.println(bank2.shards().successful());
        System.out.println("bank2.id() = " + bank2.id());
    }
```

#### 2. 查询

```java
    @Test
    void getDoc() throws IOException {
        GetRequest request = new GetRequest.Builder().index("bank").id("2Cpen4ABSWOoZKugEt8G").build();

        GetResponse<Account> accountGetResponse = client.get(request, Account.class);
        Account source1 = accountGetResponse.source();
        assert source1 != null;
        String firstname = source1.getFirstname();
        System.out.println("firstname = " + firstname);

        // 或  
				// 如果为 json 则使用 ObjectNode.class
      
        GetResponse<Account> bank = client.get(g -> g.index("bank").id("2Cpen4ABSWOoZKugEt8G"), Account.class);
        if (bank.found()){
            Account source = bank.source();
            assert source != null;
            System.out.println(source.getLastname());
        }
    }
```

#### 3. 更新

```java
    @Test
    void updateDoc() throws IOException {
        Account account = new Account();
        account.setFirstname("qin");
        account.setLastname("qinweizhao");
        account.setGender("M");
        account.setAddress("zg");
        account.setCity("hn");
        UpdateResponse<Account> bank = client.update(u -> u.id("2Cpen4ABSWOoZKugEt8G").index("bank").doc(account), Account.class);
        Number successful = bank.shards().successful();
        System.out.println("successful = " + successful);
        String id = bank.id();
        System.out.println("id = " + id);
    }
```

#### 4. 删除

```java
    @Test
    void deleteDoc() throws IOException {
        DeleteResponse bank = client.delete(d -> d.index("bank").id("2Cpen4ABSWOoZKugEt8G"));
        Number successful = bank.shards().successful();
        System.out.println("successful = " + successful);
    }
```

#### 5. 批量

```
@Test
void bulk() throws IOException {
    List<Account> list = new ArrayList<>();
    Account a1 = new Account();
    a1.setFirstname("qin");
    Account a2 = new Account();
    a2.setFirstname("q");
    list.add(a1);
    list.add(a2);

    BulkRequest.Builder br = new BulkRequest.Builder();

    for (Account account : list) {
        br.operations(op -> op
                .index(idx -> idx
                        .index("products")
                        .document(account)
                )
        );
    }
    BulkResponse result = client.bulk(br.build());

    if (result.errors()) {
        for (BulkResponseItem item : result.items()) {
            if (item.error() != null) {
                System.out.println(item.error().reason());
            }
        }
    }
}
```

#### 6. 复杂检索

```http
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "should": [
        {
          "match": {
            "address": "lane"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "email": "baluba.com"
          }
        }
      ]
    }
  }
}

```

```java
    @Test
    void search() throws IOException {
        SearchRequest.Builder searchBuilder = new SearchRequest.Builder();


        BoolQuery.Builder boolBuilder = new BoolQuery.Builder();

        boolBuilder.must(b ->
                b.match(m ->
                        m.field("address").query("mill")
                )
        );
        boolBuilder.mustNot(b ->
                b.match(m ->
                        m.field("email").query("baluba.com")
                )
        );
        boolBuilder.should(b->
                b.match(m ->
                        m.field("address").query("lane")
                )
        );
        searchBuilder.query(q ->
                q.bool(boolBuilder.build())
        );

        SearchRequest request = searchBuilder.build();
        SearchResponse<Account> search = client.search(request, Account.class);
        TotalHits total = search.hits().total();
        assert total != null;
        long value = total.value();
        System.out.println("total-value = " + value);
    }
```

#### 7. 聚合

```http
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "size": 0,
  "aggs": {
    "group_by_age": {
      "terms": {
        "field": "age"
      }
    },
    "avg_age": {
      "avg": {
        "field": "age"
      }
    }
  }
}
```

```java
@Test
    void aggregation() throws IOException {
        SearchResponse<Account> response = client.search(s ->
                        s
                                .index("bank")
                                .query(q -> q.matchAll(m -> m))
                                .aggregations("group_by_age", a ->
                                        a.terms(t ->
                                                t.field("age")
                                        )
                                )
                , Account.class);

        List<LongTermsBucket> buckets = response.aggregations().get("group_by_age").lterms().buckets().array();
        for (LongTermsBucket bucket : buckets) {
            String key = bucket.key();
            long l = bucket.docCount();
            System.out.println("key=" + key + "---l = " + l);
        }

    }
```

## 附

>代码地址：
>
>https://github.com/qinweizhao/qwz-integration/tree/master/spring-boot-elasticsearch

