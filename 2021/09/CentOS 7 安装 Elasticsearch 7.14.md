# CentOS 7 安装 Elasticsearch 7.14

## 一、下载 Elasticsearch

1. 获取下载链接

   [官网下载地址](https://www.elastic.co/cn/downloads/elasticsearch)

2. 下载压缩包

   ```bash
   wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.14.1-linux-x86_64.tar.gz
   ```

3. 解压（到上层目录）

   ```bash
   tar -zxvf elasticsearch-7.14.1-linux-x86_64.tar.gz -C ../
   ```

4. 重命名

   ```bash
   # 返回上层目录
   cd ..
   
   # 重命名
   mv elasticsearch-7.14.1 elasticsearch
   ```

## 二、修改配置

### 1、修改 Linux 配置

1. 限制 Linux 资源，配置文件为 **/etc/security/limits.conf**

   ```bash
   vim /etc/security/limits.conf
   ```

   在配置文件中增加：

   ```conf
   # 任何用户可以打开的最大的文件描述符数量，默认1024，这里的数值会限制tcp连接
   * soft nofile 65536 
   * hard nofile 65536
   # 任何用户可以打开的最大进程数
   * soft nproc 2048
   * hard nproc 4096
   # 锁住 swapping 因此需要在这个配置文件下再增加两行代码
   elasticsearch soft memlock unlimited
   elasticsearch hard memlock unlimited
   ```

   ![2021-09-12_174651](https://img.qinweizhao.com/2021/09/2021-09-12_174651.png)

2. Linux 内核配置，配置文件为 **/etc/sysctl.conf**

   ```bash
   vim /etc/sysctl.conf
   ```

   在配置文件中增加：

   ```conf
   # 修改虚拟内存的大小
   vm.max_map_count=655360
   
   fs.file-max=655360
   ```

   ![2021-09-12_174234](https://img.qinweizhao.com/2021/09/2021-09-12_174234.png)

   更改完成后执行命令使得系统文件配置生效

   ```bash
   sysctl -p
   ```

   ![2021-09-12_175221](https://img.qinweizhao.com/2021/09/2021-09-12_175221.png)

### 2、修改 Elasticsearch 配置

1. Elasticsearch 的配置文件为安装目录的 **config/elasticsearch.yml**

   ```bash
   # 进入 es 的安装目录
   cd /usr/local/elasticsearch/config
   
   # 编辑 yml 配置文件
   vim elasticsearch.yml
   ```

   在配置文件中增加：

   ```yaml
   cluster.name: my-application
   node.name: node-1
   
   network.host: 192.168.79.79
   http.port: 9200
   
   discovery.seed_hosts: ["192.168.79.79"]
   cluster.initial_master_nodes: ["node-1"]
   
   http.cors.enabled: true 
   http.cors.allow-origin: "*"
   ```

   补充：配置详解

   ```yaml
   # 配置的集群名称，默认是elasticsearch，es服务会通过广播方式自动连接在同一网段下的es服务，通过多播方式进行通信，同一网段下可以有多个集群，通过集群名称这个属性来区分不同的集群。
   cluster.name: elasticsearch
   
   # 当前配置所在机器的节点名，你不设置就默认随机指定一个name列表中名字，该name列表在es的jar包中config文件夹里name.txt文件中，其中有很多作者添加的有趣名字。
   node.name: "Franz Kafka"
   
   # 指定该节点是否有资格被选举成为node（注意这里只是设置成有资格， 不代表该node一定就是master），默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master。
   node.master: true
   
   # 指定该节点是否存储索引数据，默认为true。 
   node.data: true
   
   # 设置默认索引分片个数，默认为5片。
   index.number_of_shards: 5
   
   # 设置默认索引副本个数，默认为1个副本。如果采用默认设置，而你集群只配置了一台机器，那么集群的健康度为yellow，也就是所有的数据都是可用的，但是某些复制没有被分配
   # （健康度可用 curl 'localhost:9200/_cat/health?v' 查看， 分为绿色、黄色或红色。绿色代表一切正常，集群功能齐全，黄色意味着所有的数据都是可用的，但是某些复制没有被分配，红色则代表因为某些原因，某些数据不可用）。 
   index.number_of_replicas: 1
   
   # 设置配置文件的存储路径，默认是es根目录下的config文件夹。
   path.conf: /path/to/conf
   
   # 设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开，例：
   # path.data: /path/to/data1,/path/to/data2 
   path.data: /path/to/data
   
   # 设置临时文件的存储路径，默认是es根目录下的work文件夹。
   path.work: /path/to/work
   
   # 设置日志文件的存储路径，默认是es根目录下的logs文件夹  
   path.logs: /path/to/logs
   
   # 设置插件的存放路径，默认是es根目录下的plugins文件夹, 插件在es里面普遍使用，用来增强原系统核心功能。 
   path.plugins: /path/to/plugins
   
   # 设置为true来锁住内存不进行swapping。因为当jvm开始swapping时es的效率 会降低，所以要保证它不swap，可以把ES_MIN_MEM和ES_MAX_MEM两个环境变量设置成同一个值，并且保证机器有足够的内存分配给es。 同时也要允许elasticsearch的进程可以锁住内# # 存，linux下启动es之前可以通过`ulimit -l unlimited`命令设置。 
   bootstrap.mlockall: true
   
   # 设置绑定的ip地址，可以是ipv4或ipv6的，默认为0.0.0.0，绑定这台机器的任何一个ip。 
   network.bind_host: 192.168.0.1
   
   # 设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址。 
   network.publish_host: 192.168.0.1
   
   # 这个参数是用来同时设置bind_host和publish_host上面两个参数。
   network.host: 192.168.0.1
   
   # 设置节点之间交互的tcp端口，默认是9300。 
   transport.tcp.port: 9300
   
   # 设置是否压缩tcp传输时的数据，默认为false，不压缩。
   transport.tcp.compress: true
   
   # 设置对外服务的http端口，默认为9200。
   http.port: 9200
   
   # 设置内容的最大容量，默认100mb。
   http.max_content_length: 100mb
   
   # 是否使用http协议对外提供服务，默认为true，开启。
   http.enabled: false
   
   # gateway的类型，默认为local即为本地文件系统，可以设置为本地文件系统，分布式文件系统，hadoop的HDFS，和amazon的s3服务器等。
   gateway.type: local
   
   # 设置集群中N个节点启动时进行数据恢复，默认为1。
   gateway.recover_after_nodes: 1
   
   # 设置初始化数据恢复进程的超时时间，默认是5分钟。
   gateway.recover_after_time: 5m
   
   # 设置这个集群中节点的数量，默认为2，一旦这N个节点启动，就会立即进行数据恢复。
   gateway.expected_nodes: 2
   
   # 初始化数据恢复时，并发恢复线程的个数，默认为4。
   cluster.routing.allocation.node_initial_primaries_recoveries: 4
   
   # 添加删除节点或负载均衡时并发恢复线程的个数，默认为4。
   cluster.routing.allocation.node_concurrent_recoveries: 2
   
   # 设置数据恢复时限制的带宽，如入100mb，默认为0，即无限制。
   indices.recovery.max_size_per_sec: 0
   
   # 设置这个参数来限制从其它分片恢复数据时最大同时打开并发流的个数，默认为5。
   indices.recovery.concurrent_streams: 5
   
   # 设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）。
   discovery.zen.minimum_master_nodes: 1
   
   # 设置集群中自动发现其它节点时ping连接超时时间，默认为3秒，对于比较差的网络环境可以高点的值来防止自动发现时出错。
   discovery.zen.ping.timeout: 3s
   
   # 设置是否打开多播发现节点，默认是true。
   discovery.zen.ping.multicast.enabled: false
   
   # 设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点。
   discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]"]
   ```

   elasticsearch 7 版本引入的新集群协调子系统，新增两个如下配置项

   ```yaml
   discovery.seed_hosts
   cluster.initial_master_nodes
   
   # 例子
   discovery.seed_hosts:
      - 192.168.1.10:9300
      - 192.168.1.11 
      - seeds.mydomain.com 
      
   cluster.initial_master_nodes: 
      - master-node-a
      - master-node-b
      - master-node-c
   ```

2. 修改堆内存配置文件为安装目录的 **jvm.options**

   ```bash
   vim config/jvm.options
   ```

   在配置文件中增加：

   ```options
   -Xms512m
   -Xmx512m
   ```

   ![2021-09-12_220444](https://img.qinweizhao.com/2021/09/2021-09-12_220444.png)

3. 使用 Elasticsearch 自带的 jdk（位置与 bin 目录同级的 jdk 文件夹）

   ```bash
   # 进入 bin 目录
   cd bin
   
   # 编辑
   vim elasticsearch-env
   ```

   增加内容：

   ```elasticsearch-env
   ES_JAVA_HOME="/usr/local/elasticsearch/jdk"
   ```

   ![2021-09-12_185303](https://img.qinweizhao.com/2021/09/2021-09-12_185303.png)

## 五、启动 Elasticsearch

### 1、新增系统用户

​由于 Elasticsearch 不能使用root账户启动。

1. 新增用户

   ```bash
   useradd wz
   # 设置密码（可选）
   passwd wz
   ```

2. 更改权限

   ```bash
   chown -R wz /usr/local/elasticsearch
   ```

3. 切换用户

   ```bash
   su wz
   ```

### 2、启动并验证

1. 启动

   ```bash
   # 在 bin 目录下
   ./elasticsearch
   ```

2. 检查是否成功

   ```bash
   curl 192.168.79.79:9200
   ```

   ![2021-09-12_195418](https://img.qinweizhao.com/2021/09/2021-09-12_195418.png)
