# TDSQL-MySQL版

## 一、产品简介

### 1、概述

TDSQL MySQL版（TDSQL for MySQL）是部署在腾讯云上的一种支持自动水平拆分、Shared Nothing 架构的分布式数据库。TDSQL MySQL版 即业务获取的是完整的逻辑库表，而后端会将库表均匀的拆分到多个物理分片节点。
TDSQL MySQL版 默认部署主备架构，提供容灾、备份、恢复、监控、迁移等全套解决方案，适用于 TB 或 PB 级的海量数据库场景。

TDSQL MySQL版 提供不同的引擎供用户选择，两者均兼容 MySQL 标准协议：

- InnoDB 版采用 InnoDB 作为数据存储引擎，是 MySQL 的默认存储引擎。
- TDStore 版采用腾讯云自研的新敏态引擎 TDstore 作为数据存储引擎，该引擎可以有效解决客户业务发展过程中业务形态、业务量的不可预知性，适配金融敏态业务。

### 2、实例架构

**InnoDB引擎**

<table>
<thead><tr><th width="22%">实例架构</th><th>定义</th><th>节点</th><th>特点</th></tr></thead>
<tbody><tr>
<td>标准版（一主一从）</td>
<td>每个分片提供主从双活部署的高可用架构</td>
<td>两个节点：一个 Master 节点，一个 Slave 节点</td>
<td rowspan = "2"><li>支持从机只读</li><dx-alert infotype="explain" title="说明">一主一从架构从机只读仅可用于轻量只读任务，请避免大事务等较高负载任务，影响从机备份任务及可用性。</dx-alert><li>故障后节点自动恢复</li><li>默认监控采样粒度：5分钟/次</li><li>最大可配备份时长：7天</li><li>操作日志备份：7天</li><li>支持数据库审计，审计日志存储15天，规则配置个数暂无限制</li></td></tr>
<tr>
<td>标准版（一主二从）</td>
<td>每个分片提供主从多活部署的高可用架构</td>
<td>三个节点：一个 Master 节点，两个 Slave 节点</td></tr>
<tr>
<td>金融定制版（一主一从）</td>
<td>每个分片提供主从双活部署的高可用架构</td>
<td>两个节点：一个 Master 节点，一个 Slave 节点</td>
<td  rowspan = "2"><li>支持其他部署方案，需联系对应商务</li><li>支持从机只读，从机只读时智能负载</li><li>故障后节点自动恢复</li><li>默认监控采样粒度：1分钟/次</li><li>最大可配备份时长：3650天，需 <a href="https://console.cloud.tencent.com/workorder/category">提交工单</a></li><li>操作日志备份：默认60天，归档存储1年</li><li>支持数据库审计：审计日志存储15天</li><li>监管配合：有</li></td></tr>
<tr>
<td>金融定制版（一主二从）</td>
<td>每个分片提供主从多活部署的高可用架构</td>
<td>三个节点：一个 Master 节点，两个 Slave 节点</td></tr>
</tbody></table>
**TDStore 引擎**

内测阶段，TDStore 存储节点采用一主二从的高可用架构（三个节点：一个 Master 节点，两个 Slave 节点）。

## 二、基本原理

### 1、水平分表

#### 概述

水平拆分方案是 TDSQL MySQL版 的基础原理，它的每个节点都参与计算和数据存储，且每个节点都仅计算和存储一部分数据。因此，无论业务的规模如何增长，我们仅需要在分布式集群中不断的添加设备，用新设备去应对增长的计算和存储需要即可。

#### 水平切分

 水平切分（分表）：是按照某种规则，将一个表的数据分散到多个物理独立的数据库服务器中，形成“独立”的数据库“分片”。多个分片共同组成一个逻辑完整的数据库实例。

- 常规的单机数据库中，一张完整的表仅在一个物理存储设备上读写。
![](https://mc.qcloudimg.com/static/img/6c5dca25ea4df9d72e10143c9defe13a/image.png)
- 分布式数据库中，根据在建表时设定的分表键，系统将根据不同分表键自动分布数据到不同的物理分片中，但逻辑上仍然是一张完整的表。
![](https://mc.qcloudimg.com/static/img/fd0bd977b8ecc7ec9858b8f463090d6d/image.png)
- 在 TDSQL MySQL版 中，数据的切分通常就需要找到一个分表键（shardkey）以确定拆分维度，再采用某个字段求模（HASH）的方案进行分表，而计算 HASH 的某个字段就是 shardkey。 HASH 算法能够基本保证数据相对均匀地分散在不同的物理设备中。

##### 写入数据（ SQL 语句含有 shardkey ）

1. 业务写入一行数据。
2. 网关对 shardkey 进行 hash，得出 shardkey 的 hash 值。
3. 不同的 hash 值范围对应不同的分片（调度系统预先分片的算法决定）。
4. 数据根据分片算法，将数据存入实际对应的分片中。
![](https://mc.qcloudimg.com/static/img/5dd0a9883398f72c82a7e7c6b0b0b0e9/image.png)

#### 数据聚合

数据聚合：如果一个查询 SQL 语句的数据涉及到多个分表，此时 SQL 会被路由到多个分表执行，TDSQL MySQL版 会将各个分表返回的数据按照原始 SQL 语义进行合并，并将最终结果返回给用户。
>!执行 SELECT 语句时，建议您在 where 条件带上 shardKey 字段，否则会导致数据需要全表扫描然后网关才对执行结果进行聚合。全表扫描响应较慢，对性能影响很大。

##### 读取数据（有明确 shardkey 值）

1. 业务发送 select 请求中含有 shardkey 时，网关通过对 shardkey 进行 hash。
2. 不同的 hash 值范围对应不同的分片。
3. 数据根据分片算法，将数据从对应的分片中取出。

##### 读取数据（无明确 shardkey 值）

1. 业务发送 select 请求没有 shardkey 时，将请求发往所有分片。
2. 各个分片查询自身内容，发回 Proxy 。
3. Proxy 根据 SQL 规则，对数据进行聚合，再答复给网关。
![](https://mc.qcloudimg.com/static/img/89b6fdca310ab3a51b2a573ba0b63373/image.png)

### 2、读写分离

#### 功能简介

当处理大数据量读请求的压力大、要求高时，可以通过读写分离功能将读的压力分布到各个从节点上。
TDSQL MySQL版 默认支持读写分离功能，架构中的每个从机都能支持只读能力，如果配置有多个从机，将由网关集群（TProxy）自动分配到低负载从机上，以支撑大型应用程序的读取流量。

#### 基本原理

读写分离基本的原理是让主节点（Master）处理事务性增、改、删操作（INSERT、UPDATE、DELETE），让从节点（Slave）处理查询操作（SELECT）。

#### 只读帐号

**说明：**若实例架构为一主一从，只读分离功能仅可用作低负载只读任务，请避免大事务等较高负载任务，影响从机备份任务及可用性。

只读帐号是一类仅有读权限的帐号，默认从数据库集群中的从机（或只读实例）中读取数据。
通过只读帐号，对读请求自动发送到备机，并返回结果。
![](https://main.qcloudimg.com/raw/f375da187dfc94d081d2f4392d0dd8bd.png)



### 3、弹性扩展

#### 概述

TDSQL MySQL版 支持在线实时扩容，扩容方式分为新增分片和现有分片扩容两种方式，整个扩容过程对业务完全透明，无需业务停机。扩容时仅部分分片存在秒级的只读或中断，整个集群不会受影响。

#### 扩容过程

TDSQL MySQL版 主要是采用自研的自动再均衡技术保证自动化的扩容和稳定。

##### 新增分片扩容

1. 在 [TDSQL MySQL版控制台](https://console.cloud.tencent.com/dcdb) 对需要扩容的 A 节点进行扩容操作。
2. 根据新加 G 节点配置，将 A 节点部分数据搬迁（从备机）到 G 节点。
3. 数据完全同步后，A、G 节点校验数据库，存在一至几十秒的只读，但整个服务不会停止。
4. 调度通知 proxy 切换路由。
![](https://mc.qcloudimg.com/static/img/d407c9bf2740c3ceb803392448856cf2/image.png)

##### 现有分片扩容

基于现有分片的扩容相当于更换了一块更大容量的物理分片。
>?基于现有分片的扩容没有增加分片，不会改变划分分片的逻辑规则和分片数量。

1. 按需要升级的配置分配一个新的物理分片（以下简称新分片）。
2. 将需要升级的物理分片（以下简称老分片）的数据、配置等同步数据到新分片中。
3. 同步数据完成后，在腾讯云网关做路由切换，切换到新分片继续使用。
![](https://mc.qcloudimg.com/static/img/d30e97c05742feccf7728e6a326e826f/image.png)

#### 相关操作

分布式数据库由多个分片组成，如您需要将现有 TDSQL MySQL版 实例的规格升级到更高规格，请参见 [升级实例](https://cloud.tencent.com/document/product/557/7706)。

### 4、强同步

#### 背景
传统数据复制方式有如下三种：
- 异步复制：应用发起更新请求，主节点（Master） 完成相应操作后立即响应应用，Master 向从节点（Slave）异步复制数据。
- 强同步复制：应用发起更新请求，Master 完成操作后向 Slave 复制数据，Slave 接收到数据后向 Master 返回成功信息，Master 接到 Slave 的反馈后再应答给应用。Master 向 Slave 复制数据是同步进行的。
- 半同步复制：应用发起更新请求，Master 在执行完更新操作后立即向 Slave 复制数据，Slave 接收到数据并写到 relay log 中（无需执行） 后才向 Master 返回成功信息，Master 必须在接受到 Slave 的成功信息后再向应用程序返回响应。

#### 存在问题
当 Master 或 Slave 不可用时，以上三种传统数据复制方式均有几率引起数据不一致。

数据库作为系统数据存储和服务的核心能力，其可用性要求非常高。在生产系统中，通常都需要用高可用方案来保证系统不间断运行，而数据同步技术是数据库高可用方案的基础。

#### 解决方案
MAR 强同步复制方案是腾讯自主研发的基于 MySQL 协议的并行多线程强同步复制方案，只有当备机数据完全同步（日志）后，才由主机给予应用事务应答，保障数据正确安全。

原理示意图如下：
![](https://main.qcloudimg.com/raw/45fdf5a387130c2f7cabdf962b74977e.png)
在应用层发起请求时，只有当从节点（Slave）返回信息成功后，主节点（Master）才向应用层应答请求成功，以确保主从节点数据完全一致。
MAR 强同步方案在性能上优于其他主流同步方案，具体数据详情可参见 [强同步性能对比数据](https://cloud.tencent.com/document/product/557/10105)。主要特点如下：
- 一致性的同步复制，保证节点间数据强一致性。
- 对业务层面完全透明，业务层面无需做读写分离或同步强化工作。
- 将串行同步线程异步化，引入线程池能力，大幅度提高性能。
- 支持集群架构。
- 支持自动成员控制，故障节点自动从集群中移除。
- 支持自动节点加入，无需人工干预。
- 每个节点都包含完整的数据副本，可以随时切换。
- 无需共享存储设备。

## 三、快速入门

### 1、InnoDB 引擎

#### 创建实例

##### 前提条件

已注册腾讯云账号并完成实名认证。
- 如需注册腾讯云账号：
<div style="background-color:#00A4FF; width: 170px; height: 35px; line-height:35px; text-align:center;"><a href="https://cloud.tencent.com/register?s_url=https%3A%2F%2Fcloud.tencent.com%2F" target="_blank"  style="color: white; font-size:16px;" hotrep="document.guide.3128.btn1">点此注册腾讯云账号</a></div>
- 如需完成实名认证：
<div style="background-color:#00A4FF; width: 170px; height: 35px; line-height:35px; text-align:center;"><a href="https://console.cloud.tencent.com/developer" target="_blank"  style="color: white; font-size:16px;"  hotrep="document.guide.3128.btn2">点此完成实名认证</a></div>

##### 操作步骤

1. 登录 [TDSQL MySQL版 购买页](https://console.cloud.tencent.com/tdsqld/tdmysql-buy)，根据实际需求选择各项配置信息，确认无误后，单击**立即购买**。
 - **计费模式**：支持包年包月和按量计费。
    - 若业务量有较稳定的长期需求，建议选择包年包月。
    - 若业务量有瞬间大幅波动场景，建议选择按量计费。
 - **地域**：选择您业务需要部署 TDSQL MySQL版 的地域。建议您选择与云服务器同一个地域，不同地域的云产品内网不通，购买后不能更换。
 - **网络类型**：TDSQL MySQL版 所属的网络，建议您选择与云服务器同一个地域下的同一私有网络，否则无法通过内网连接云服务器和数据库。
 - **实例版本**：请参见 [实例架构](https://cloud.tencent.com/document/product/557/11332)。
 - **主/从可用区**：选择主备不同可用区，可保护数据库以防发生故障或可用区中断。
 - **内核版本**：5.6内核暂不支持分布式事务（XA），如需请选择 5.7版本内核，详细介绍请参见 [概述](https://cloud.tencent.com/document/product/557/8765)。
 - **分片数量/规格/硬盘**：建议分片数量选择2、4、8，否则会导致数据倾斜的问题，分片配置请参见 [分片配置](https://cloud.tencent.com/document/product/557/9347)。
 - **所属项目**：选择数据库实例所属的项目，缺省设置为默认项目。
 - **标签**：便于分类管理实例资源，请参见 [标签概述](https://cloud.tencent.com/document/product/651/13334)。
 - **安全组**：安全组创建与管理请参见 [云数据库安全组](https://cloud.tencent.com/document/product/557/10106)。
 - **实例名**：可选择创建后命名或立即命名。
 - **支持字符集**：支持 UTF8 、LATIN1 、GBK、UTF8MB4、GB18030 字符集。
 - **表名大小写敏感**：敏感（lower_case_table_names = 0）、不敏感（lower_case_table_names = 1），该参数为初始化参数，数据库初始化完成后无法修改。
 - **开启强同步**：支持强同步(可退化)、异步，详细介绍请参见 [强同步](https://cloud.tencent.com/document/product/557/10570)。
 - 计费详情请参见 [计费概述](https://cloud.tencent.com/document/product/557/7703)。
2. 支付完成后，返回实例列表，待实例状态变为**运行中**，即可进行后续操作。
![](https://qcloudimg.tencent-cloud.cn/raw/a2910cd5cf26c49b04b7bd785f85b968.png)

##### 后续操作

通过内外网地址连接 TDSQL MySQL版 实例，请参见 [连接实例](https://cloud.tencent.com/document/product/557/10238)。

创建 TDSQL MySQL版 实例后，您还需要进行实例的初始化，以轻松启用实例。

##### 操作步骤

1. 登录 [TDSQL MySQL版 控制台](https://console.cloud.tencent.com/dcdb)，选择对应地域后，在实例列表，选择状态为**未初始化**的实例，在**操作**列选择**更多** > **初始化**。
2. 在弹出的初始化对话框，根据需要选择配置后，单击**确定**。
 - 支持字符集：选择 MySQL 数据库支持的字符集。
 - 表名大小写敏感：数据库表名大小写是否敏感。
 - 开启强同步：开启强同步可以保证在主机故障时备机数据的一致性，至少需要2个节点方可正常运行。
![](https://main.qcloudimg.com/raw/7ec7d0fd038f2bb14760d11199afe1f0.png)
3. 返回实例列表，待实例状态变为**运行中**，即可进行连接数据库操作。

##### 后续操作

通过 Windows 云服务器或 Linux 云服务器，以内外网两种不同的方式连接 TDSQL MySQL版，请参见 [连接实例](https://cloud.tencent.com/document/product/557/10238)。

#### 连接实例

##### 连接方式
连接 TDSQL MySQL版 的方式如下：
- **内网地址连接**：通过内网地址连接 TDSQL MySQL版，使用云服务器 CVM 直接连接云数据库的内网地址，这种连接方式使用内网高速网络，延迟低。
 - 云服务器和数据库须是同一账号，且同一个[ VPC](https://cloud.tencent.com/document/product/215/20046) 内（保障同一个地域），或同在基础网络内。
 - 内网地址系统默认提供，可在  [TDSQL MySQL版 控制台](https://console.cloud.tencent.com/tdsqld/instance-tdmysql) 的实例列表或实例详情页查看。
>?对于不同的 VPC 下（包括同账号/不同账号，同地域/不同地域）的云服务器和数据库，内网连接方式请参见 [云联网](https://cloud.tencent.com/document/product/877)。
>
- **外网地址连接**：无法通过内网连接时，可通过外网地址连接 TDSQL MySQL版。外网地址需 [手动开启](#waiwang)，可在 [TDSQL MySQL版 控制台](https://console.cloud.tencent.com/tdsqld/instance-tdmysql) 的实例详情页查看，不需要时也可关闭。
 - 开启外网地址，会使您的数据库服务暴露在公网上，可能导致数据库被入侵或攻击。建议您使用内网连接数据库。
 - 云数据库外网连接适用于开发或辅助管理数据库，不建议正式业务连接使用，因为可能存在不可控因素会导致外网连接不可用（例如 DDOS 攻击、突发大流量访问等）。
 - 目前支持开启外网访问的地域为：广州、上海、北京、成都、南京。
 - 开启外网访问必须绑定安全组，请参见 [配置云数据库安全组](https://cloud.tencent.com/document/product/557/10106)。

##### 准备工作

###### 创建帐号
1. 登录 [TDSQL MySQL版 控制台](https://console.cloud.tencent.com/tdsqld/instance-tdmysql)，在实例列表，单击实例 ID 或**操作**列的**管理**，进入实例管理页面。
2. 在实例管理页面，选择**帐号管理**页，单击**创建帐号**。
![](https://main.qcloudimg.com/raw/bfbaa009dfde227aecb026a97c5b1b2c.png)
3. 在弹出的对话框，输入帐号名、主机、密码等，确认无误后，单击**确认，下一步**。
主机名实际是网络出口地址，支持%的匹配方式，代表所有 IP 均可访问。
4. 进入修改权限对话框，根据需求分配权限后，单击**确定修改**即可完成权限分配。若需稍后设置权限，单击**取消修改**即可。
左边导航栏提供完全兼容 MySQL 管理方式的图形化界面，权限管理可以细化到列级。
![](https://qcloudimg.tencent-cloud.cn/raw/b07384986d5e20f768728e712079547e.png)
5. 返回帐号列表，单击**修改权限**可以修改用户权限，单击**克隆帐号**可以完全复制当前帐号权限来新建一个帐号，单击**更多**可以重置密码和删除帐号。
![](https://main.qcloudimg.com/raw/fe697afd0c76d921ce7961f4c57a9017.png)

###### [（可选）开启外网地址](id:waiwang)
1. 登录 [TDSQL MySQL版 控制台](https://console.cloud.tencent.com/tdsqld/instance-tdmysql)，在实例列表，单击实例 ID，进入实例详情页，在基本信息的**外网地址**处，单击**开启**。
![](https://qcloudimg.tencent-cloud.cn/raw/665001ff3241b3a955720d475764ab93.png)
2. 开启后，在**外网地址**处获取外网地址和端口号。TDSQL MySQL版 提供了唯一的 IP、端口供用户访问和使用。


创建帐号和获取内外网地址后，可通过第三方工具和程序驱动进行连接 TDSQL MySQL版。
- Windows 端，以命令行连接、客户端连接和 JDBC 驱动连接三种方式为例。
- Linux 端，以命令行连接为例。

##### 从 Windows 端连接
###### Windows 命令行连接
1. 打开 Windows 命令行，在 MySQL 的正确路径下输入以下命令。
```
mysql -h内外网地址 -P端口号 -u用户名  -p
Enter password: **********（输入密码）
```
2. 将相关代码正确输入后，显示如下信息，成功连接数据库，下一步即可进行数据库内相关操作。
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
```

###### Windows 客户端连接
1. 下载一个标准的 SQL 客户端，例如 MySQL Workbench 、SQLyog 等，本文以 SQLyog 为例。
2. 打开 SQLyog，选择**文件** > **新连接**，输入对应的主机地址、端口、用户名和密码，单击**连接**。
 - 我的SQL主机地址：输入前面获得的内外网地址。
 - 用户名：输入创建的帐号名。
 - 密码：输入帐号对应的密码，如忘记密码，可至 [控制台](https://console.cloud.tencent.com/tdsqld/instance-tdmysql) 进行修改。
 - 端口：输入地址对应的端口。
![](https://main.qcloudimg.com/raw/56645ca6d1c5fe7803e05f7643b833ae.png)
3. 连接成功页面如下图所示，在此页面即可进行数据库内相关操作。
![](https://main.qcloudimg.com/raw/1f492c179e2f604afc02a775d58a2d9c.png)

###### Windows JDBC 驱动连接
TDSQL MySQL版 支持程序驱动连接，本文以 Java 使用 JDBC Driver for MySQL (Connector/J) 连接 TDSQL MySQL版 为例。

1. 在 [MySQL 官网](https://dev.mysql.com/downloads/connector/j/5.0.html) 下载一个 JDBC 的 jar 包，将其导入 Java 引用的 Library 中。
2. 调用 JDBC 代码如下：
```
public static final String url = "内外网地址";
public static final String name = "com.mysql.jdbc.Driver"; //调用 JDBC 驱动
public static final String user = "用户名";
public static final String password = "密码";
//JDBC
Class.forName("com.mysql.jdbc.Driver"); 
Connection conn=DriverManager.getConnection("url, user, password");
//
conn.close();
```
3. 连接成功后，下一步即可进行其他数据库内操作。

##### 从 Linux 端连接
###### Linux 命令行连接
以腾讯云服务器中 CentOS 7.2 64 位系统为例，云服务器购买请参见 [购买方式](https://cloud.tencent.com/document/product/213/506)。
1. 登录 Linux 后，输入命令 `yum install mysql`，利用 CentOS 自带的包管理软件 Yum 在腾讯云镜像源中下载安装 MySQL 客户端。
![](https://main.qcloudimg.com/raw/fe1470a47fd3311460a7cfd24f70a88a.png)
2. 命令行显示 complete 后，表示 MySQL 客户端安装完成。
3. 输入命令 `mysql -h内外网地址 -P端口 -u用户名 -p` 连接 TDSQL MySQL版，下一步即可进行分表操作。
下图以`show databases;`为例。
![](https://main.qcloudimg.com/raw/c094ba46f8a3d321ad4a199d88d9d3e6.png)

#### 管理分表

以下为连接 TDSQL MySQL版 后一些简单的数据库操作介绍，本文以分表为例。

##### 建表
- 分表、单表、广播表的区别详情请参考 [相关表详情文档](https://cloud.tencent.com/document/product/557/8764)。
- 分表键（shardkey）选择的限制请参考 [分表键详情文档](https://cloud.tencent.com/document/product/557/8767)。
- 建分表时，需指明分表键（shardkey），代码示例如下：
```
mysql> create table test1(id int primary key,name varchar(20),addr varchar(20))shardkey=id;
Query OK,0 rows affected(0.15 sec)
```

##### 插入数据
>!insert 字段必须包含分表键，否则会拒绝执行。

向刚刚建立的表中插入数据，代码示例如下：
```
mysql> insert into test1(id,name) VALUES(1,'test');
Query OK,1 rows affected(0.08 sec)
mysql> insert into test3(name,addr) values('example','shenzhen');
ERROR 7013 (HY000): Proxy ERROR:get_shardkeys return error
```

##### 查询数据
>!查询数据时，最好带上分表键，分布式路由将自动跳转到对应分片，此时效率最高。否则，分布式系统会自动全表扫描，然后在网关进行结果聚合，效率较低。

查询数据代码示例如下：
```
mysql> select id from test1 where id=1;
```

##### 删除数据
>!delete 必须带有 where 条件，where 条件建议带上分表键。

删除代码示例如下：
```
mysql> delete from test1 where id=1;
Query OK, 1 row affected (0.02 sec)
```

### 2、TDStore 引擊

#### 创建实例

##### 前提条件
已注册腾讯云账号并完成实名认证。

- 如需注册腾讯云账号：
<div style="background-color:#00A4FF; width: 170px; height: 35px; line-height:35px; text-align:center;"><a href="https://cloud.tencent.com/register?s_url=https%3A%2F%2Fcloud.tencent.com%2F" target="_blank"  style="color: white; font-size:16px;" hotrep="document.guide.3128.btn1">点此注册腾讯云账号</a></div>
- 如需完成实名认证：
<div style="background-color:#00A4FF; width: 170px; height: 35px; line-height:35px; text-align:center;"><a href="https://console.cloud.tencent.com/developer" target="_blank"  style="color: white; font-size:16px;"  hotrep="document.guide.3128.btn2">点此完成实名认证</a></div>

##### 操作步骤
1. 登录 [TDSQL MySQL版（TDStore 引擎）控制台](https://console.cloud.tencent.com/tdsqld/instance-tdstore)，在实例列表，单击**新建**，进入购买页。
2. 在购买页，选择对应配置，单击**立即购买**。
 - **计费模式**：支持按量计费和包年包月。
 - **地域**：选择您业务需要部署 TDSQL MySQL版（TDStore 引擎）的地域。建议您选择与云服务器同一个地域，不同地域的云产品内网不通，购买后不能更换。
 - **网络**：TDSQL MySQL版（TDStore 引擎）所属的网络，建议您选择与云服务器同一个地域下的同一私有网络，否则无法通过内网连接云服务器和数据库。
 - **计算节点数\规格、存储节点数\规格**：根据业务需要选择。
3. 购买成功后，返回 [实例列表](https://console.cloud.tencent.com/tdsqld/instance-tdstore)，即可进行后续操作。

#### 连接实例

##### 连接方式
连接 TDSQL MySQL版（TDStore 引擎）的方式如下：
**内网地址连接**：通过内网地址连接云数据库，使用云服务器 CVM 直接连接云数据库的内网地址，这种连接方式使用内网高速网络，延迟低。
 - 云服务器和数据库须是同一账号，且同一个[ VPC](https://cloud.tencent.com/document/product/215/20046) 内（保障同一个地域）。
 - 内网地址系统默认提供，可在  [TDSQL 控制台](https://console.cloud.tencent.com/tdsqld/instance-tdstore) 的实例列表查看。

>?对于不同的 VPC 下（包括同账号/不同账号，同地域/不同地域）的云服务器和数据库，内网连接方式请参见 [云联网](https://cloud.tencent.com/document/product/877)。

##### 准备工作

###### 创建帐号
1. 登录 [TDSQL 控制台](https://console.cloud.tencent.com/tdsqld/instance-tdstore)，在实例列表，单击实例 ID 或**操作**列的**管理**，进入实例管理页面。
2. 在实例管理页面，选择**帐号管理**页，单击**创建帐号**。
3. 在弹出的对话框，输入帐号名、密码、备注，确认无误后，单击**确认**。
4. 创建成功的账号，将具有以下权限：
```
SELECT,UPDATE,INSERT,ALTER,DELETE,CREATE,CREATE VIEW,SHOW DATABASES,INDEX,SHOW VIEW,PROCESS,DROP
```


<br>创建帐号和获取内网地址后，可通过第三方工具和程序驱动进行连接 TDSQL MySQL版（TDStore 引擎）。
- Windows 端，以命令行连接、客户端连接和 JDBC 驱动连接三种方式为例。
- Linux 端，以命令行连接为例。

##### 从 Windows 端连接

###### Windows 命令行连接
1. 打开 Windows 命令行，在 MySQL 的正确路径下输入以下命令。
```
mysql -h内网地址 -P端口号 -u用户名  -p
Enter password: **********（输入密码）
```
2. 将相关代码正确输入后，显示如下信息，成功连接数据库，下一步即可进行数据库内相关操作。
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
```

###### Windows 客户端连接
1. 下载一个标准的 SQL 客户端，例如 MySQL Workbench 、SQLyog 等，本文以 SQLyog 为例。
2. 打开 SQLyog，选择**文件** > **新连接**，输入对应的主机地址、端口、用户名和密码，单击**连接**。
 - 我的 SQL 主机地址：输入前面获得的内网地址。
 - 用户名：输入创建的帐号名。
 - 密码：输入帐号对应的密码，如忘记密码，可至 [控制台](https://console.cloud.tencent.com/tdsqld/instance-tdstore) 进行修改。
 - 端口：输入地址对应的端口。
![](https://main.qcloudimg.com/raw/56645ca6d1c5fe7803e05f7643b833ae.png)
3. 连接成功页面如下图所示，在此页面即可进行数据库内相关操作。
![](https://main.qcloudimg.com/raw/1f492c179e2f604afc02a775d58a2d9c.png)

###### Windows JDBC 驱动连接
TDSQL MySQL版（TDStore 引擎）支持程序驱动连接，本文以 Java 使用 JDBC Driver for MySQL (Connector/J) 连接为例。
1. 在 [MySQL 官网](https://dev.mysql.com/downloads/connector/j/5.0.html) 下载一个 JDBC 的 jar 包，将其导入 Java 引用的 Library 中。
2. 调用 JDBC 代码如下：
```
public static final String url = "内网地址";
public static final String name = "com.mysql.jdbc.Driver"; //调用 JDBC 驱动
public static final String user = "用户名";
public static final String password = "密码";
//JDBC
Class.forName("com.mysql.jdbc.Driver"); 
Connection conn=DriverManager.getConnection("url, user, password");
//
conn.close();
```
3. 连接成功后，下一步即可进行其他数据库内操作。

##### 从 Linux 端连接
###### Linux 命令行连接
以腾讯云服务器中 CentOS 7.2 64 位系统为例，云服务器购买请参见 [购买方式](https://cloud.tencent.com/document/product/213/506)。

1. [登录 Linux](https://cloud.tencent.com/document/product/213/2936#.E6.AD.A5.E9.AA.A43.EF.BC.9A.E7.99.BB.E5.BD.95.E4.BA.91.E6.9C.8D.E5.8A.A1.E5.99.A8) 后，输入命令 `yum install mysql`，利用 CentOS 自带的包管理软件 Yum 在腾讯云镜像源中下载安装 MySQL 客户端。
![](https://main.qcloudimg.com/raw/fe1470a47fd3311460a7cfd24f70a88a.png)
2. 命令行显示 complete 后，表示 MySQL 客户端安装完成。
3. 输入命令 `mysql -h内网地址 -P端口 -u用户名 -p` 连接数据库，下一步即可进行分表操作。
下图以 `show databases;` 为例。
![](https://main.qcloudimg.com/raw/c094ba46f8a3d321ad4a199d88d9d3e6.png)

## 四、最佳实践

### 1、从单机实例导入到分布式实例

由于分布式数据库的分布式架构对用户透明，一般情况下，只需要预先建好表结构。可以使用 mysqldump、或其他 Navicat、SQLyog 等 MySQL 客户端进行迁移。迁移步骤如下：

第一步：准备导入导出环境
第二步：导出源表的表结构和数据
第三步：修改建表语句并在目的表中创建表结构
第四步：导入数据

#### 步骤1：准备导入导出环境
准备迁移数据前，您需要准备好如下环境：
- 云服务器
	- 建议配置 CPU 2核，内存8GB，磁盘500GB以上（取决于数据量大小）
	- Linux
	- 安装 MySQL 客户端
	- 如果您迁移的数据量较小（< 10GB），也可以通过外网（互联网）直接导入，无需准备
- TDSQL MySQL版
	- 根据预期选择大小，并根据源库字符集，表名大小写，innodb_page_size 大小进行初始化
	- 创建帐号，该帐号建议开启全局所有权限
	- 必要时开启外网 IP

#### 步骤2：导出源表的表结构和数据
##### 演示环境
- 操作库：caccts
- 操作表：t_acct_water_0
- 源库：单实例 MySQL
- 目标库：TDSQL for Percona、MariaDB

##### 在源库导出表结构
通过命令`mysqldump -u username -p password -d dbname tablename > tablename.sql`导出表结构。
```
//命令实例
mysqldump -utest -ptest1234 -d -S /data/4003/prod/mysql.sock caccts t_acct_water_0 > table.sql
```

##### 在源库导出表数据
通过命令`mysqldump -c -u username -p password dbname tablename > tablename.sql`导出表数据。
```
//命令实例
mysqldump -c -t -utest -ptest1234  -S /data/4003/prod/mysql.sock caccts t_acct_water_0 > data.sql
```
>!导出数据必须通过 mysqldump 工具导出，并且加上 -c 参数，因为这样导出的数据行都带有列名字段，不带列名字段的 sql 会被 TDSQL for Percona、MariaDB 拒绝掉。-t 参数的意义是不导出表结构，只导出表数据。

##### 上传文件至云服务器某目录
上传前，您需开启 CVM 外网访问地址，并参见 [Linux 系统通过 SCP 上传文件到 Linux 云服务器](https://cloud.tencent.com/document/product/213/2133) 上传文件，您至少需要上传刚刚导出的：
- 表结构 sql：table.sql
- 数据 sql：data.sql

#### 步骤3：修改建表语句并在目的表中创建表结构
打开刚导出的表结构文件 table.sql，参考如下格式语句添加主键和 shardkey，并另存为 tablenew.sql。
```
CREATE TABLE（
列名称1 数据类型,
列名称2 数据类型,
列名称3 数据类型,
....，
PRIMARY KEY('列名称n')）
ENGINE=INNODB DEFAULT CHARSET=xxxx 
shardkey=keyname
```
>!必须要设置主键，必须指定 shardkey，必须注意表名大小写问题，建议删除多余注释，否则建表可能不成功。
>

![](https://mc.qcloudimg.com/static/img/1cd921ececbacf81226a69a0eb5b919a/image.png)

#### 步骤4：导入数据
##### 连接 TDSQL for Percona、MariaDB 实例
在 CVM 上使用`mysql -u username -p password -h IP -P port `登录 MySQL 服务器，然后使用`use dbname`进入数据库。
>!您可能需要先创建库。

##### 导入表结构
使用刚刚上传的文件，用`source`命令导入数据。
1. 先导入表结构：`source /文件路径/tablenew.sql；`
2. 再导入数据：`source /文件路径/data.sql；`
3. 校验导入情况：`select  count(*) from tablename;`

>!需先导入建表语句，再导入数据。也可以通过 mysql 的 source 命令直接导入 sql。

#### 其他方案
整体来说，只要能够在导入数据前，在目的表先创建对应的表结构（需指定 shardkey），就可以比较顺利的导入数据。

### 2、从分布式实例导入到分布式实例


由于分布式数据库到分布式数据库的数据导入方案与一般情况不同，使用 mysqldump 对分布式实例导入数据到分布式实例的步骤如下：

#### 1. 安装 MariaDB 版本的 mysqldump
购买 Linux 版的云服务器，使用`yum install mariadb-server`即可安装。

#### 2. 导出表结构
```
mysqldump --compact --single-transaction -d -uxxx -pxxx  -hxxx.xxx.xxx.xxx -Pxxxx  db_name table_name   >  schema.sql
```
>?db_name 和 table_name 参数根据实际情况选择。

#### 3. 导出数据
mysqldump 导出数据：
在 [TDSQL MySQL版 控制台](https://console.cloud.tencent.com/dcdb) 的参数设置中设置 net_write_timeout 参数：set global net_write_timeout=28800
```
mysqldump --compact --single-transaction --no-create-info -c -uxxx  -pxxx -hxxx.xxx.xxx.xxx -Pxxxx db_name table_name  > data.sql
```
>?db_name 和 table_name 参数根据实际情况选择，如果导出的数据要导⼊到另外⼀套 TDSQL MySQL版 环境的话，必须加上 -c 选项，-c 与 db_name 之间需添加空格。
>
![](https://main.qcloudimg.com/raw/567f47f17939e809a7e0aa2f06627353.png)

#### 4. 在目标库创建 db
```
mysql --default-character-set=utf8 -uxxx -pxxx -hxxx.xxx.xxx.xxx -Pxxxx -e "create database dbname;";
```
- --default-character-set=utf8：根据您目标表实际情况设定。
- -uxxx：有权限的帐号（-u 是关键字）。
- -pxxx：密码（-p 是关键字）。
- -hxxx.xxx.xxx.xxx -Pxxxx：数据库实例的 IP 和端口。
- dbname：代表 db 的名字。

#### 5. 在目标库上导入表结构
```
mysql --default-character-set=utf8 -uxxx -pxxx -hxxx.xxx.xxx.xxx -Pxxxx dbname < schema.sql
```
- --default-character-set=utf8：根据您目标表实际情况设定。
- -uxxx：有权限的帐号（-u 是关键字）。
- -pxxx：密码（-p 是关键字）。
- -hxxx.xxx.xxx.xxx -Pxxxx：数据库实例的 IP 和端口。
- dbname：代表 db 的名字。

#### 6. 在目标库上导入表数据
```
mysql --default-character-set=utf8 -uxxx -pxxx -hxxx.xxx.xxx.xxx -Pxxxx dbname < data.sql
```
>?如果源表中使用了自增字段，并且导入的时候出现“Column 'xx' specified twice”的错误，则需要对 schema.sql 做处理。
   去掉自增字段的反引号(cat schema.sql | tr "`" " " > schema_tr.sql )，然后 drop database，使用处理后的 schema_tr.sql 重复步骤3 - 5的操作。

![](https://main.qcloudimg.com/raw/b55fe0763f4e8fd1795fec478e9dc0c1.png)

### 3、选择实例配置和分片配置

#### 选型概述

TDSQL MySQL版 由分片（sharding）组成，分片的规格和分片数量决定了 TDSQL MySQL版 实例的处理能力。理论上来讲：

- TDSQL MySQL版 实例读写并发性能 = ∑（某规格分片性能 * 某规格分片数量）
- TDSQL MySQL版 实例事务性能 = ∑（某规格分片事务性能 * 70% * 某规格分片数量）

因此，分片规格越高、分片数量越多，实例的处理能力越强。而分片性能，主要与 CPU / 内存 相关，并以 QPS 为基础衡量指标，我们在分片性能说明章节，给出了大致性能指标。

#### 分片规格的选择

TDSQL MySQL版 分片规格的选择，主要从三个方面需求来决定：1、性能需求；2、容量需求；3、其他要求。

**性能需求**：通过预判至少6个月的性能规模和可能增长，您可以确定您分布式实例所需总 CPU / 内存 规模。
**容量需求**：通过预判至少1年的容量规模和可能增长，您可以确定您分布式实例所需总 磁盘 规模。
**其他要求**：我们建议一个分片**至少存储5000万行数据**，并考虑到业务中所需的 [广播表、单表](https://cloud.tencent.com/document/product/557/8764)，和节点内 join 等业务需求。

>!建议您先确保让单个分片配置较大，而分片数量较少。

综合上述来看，我们预估您可能有如下几种业务特点，推荐策略如下：
- 使用 TDSQL MySQL版 做功能性测试，且对性能没有特别要求：2个分片，每个分片配置为：**内存/磁盘：2GB/25GB**。
- 业务发展初期，总数据规模较小但增长快的选型：2个分片，每个分片配置为：**内存/磁盘：16GB/200GB**。
- 业务发展稳定，根据业务实际情况选型：4个分片，每个分片配置等于：**当前业务峰值*增长率/4**

#### 分片性能测试

数据库基准性能测试为 sysbench 0.5 工具。
修改说明：对 sysbench 自带的 otlp 脚本做了修改，读写比例修改为 1：1，并通过执行测试命令参数 oltp_point_selects 和 oltp_index_updates 控制读写比例，本文测试用例的均采用4个 select 点，1个 update 点，读写比例保持 4:1。

|内存(GB)|存储空间(GB)|数据集(GB)|客户端数|单客户端并发数|QPS|TPS|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|2GB|100GB|46GB|1|128|1880|351|
|4GB|200GB|76GB|1|128|3983|797|
|8GB|200GB|142GB|1|128|6151|1210|
|16GB|400GB|238GB|1|128|10098|2119|
|32GB|700GB|238GB|2|128|20125|3549|
|64GB|1T|378GB|2|128|37956|7002|
|96GB|1.5T|378GB|3|128|51026|10591|
|120GB|2T|378GB|3|128|81050|15013|
|240GB|3T|567GB|4|128|96891|17698|
|480GB|6T|567GB|6|128|140256|26599|


此处 TPS 为单机 TPS，并非测试的是分布式事务的 TPS。
根据运营策略要求，当前 TDSQL MySQL版 的（部分）实例都采用闲时超用技术，所以您可能在您的监控中看到 CPU 利用率超过100%的情况。

## 五、开发指南

### 1、InnoDB 引擎

#### 概述

在 SQL 使用上，分布式实例高度兼容 MySQL 的协议和语法，但由于架构的差异，对于 SQL 有一定的限制，同时为了更好地发挥分布式的优势，建议业务在使用时尽量参考下文的建议。

分布式实例提供水平扩容能力，适合海量数据的场景。具有如下功能特性：
- 提供了灵活的读写分离模式
- 支持全局的 order by、group by、limit 操作
- 聚合函数支持 sum、count、avg、min、max 等
- 支持跨节点（set）的 join、子查询
- 支持预处理协议
- 支持全局唯一字段，支持 sequence
- 支持分布式事务
- 支持两级分区
- 提供特定的 SQL 查询整个集群的配置和状态


分布式实例支持三种不同类型的表：
- **分表**：即水平拆分表，该表从业务视角是一张完整的逻辑表，但后端根据分表键（shardkey）的 HASH 值将数据分布到不同的节点（set）中。
- **单表**：又名 Noshard 表，无需拆分，且没有做任何特殊处理的表，目前分布式实例将该表默认存放在第一个物理节点组（set）中。
- **广播表**：又名小表广播技术，即设置为广播表后，该表的所有操作都将广播到所有节点（set）中，每个 set 都有该表的全量数据，常用于业务系统的配置表等。

>!
>- 在分布式实例中，如果两张表**分表键**相等，这意味着，两张表相同的分表键对应的行，一定存储于相同的物理节点组中。这种场景通常被称为组拆分（groupshard），会极大提高业务联合查询等语句的处理效率。
>- 由于单表默认放置在第一个 set 上，如果在分布式实例中建立了大量的单表，则会导致第一个 set 的负载太大。
>- 除特殊情况外，建议在分布式实例中尽量都使用分表。

#### 使用限制


##### 大特性限制
- 不支持自定义函数、事件、表空间
- 不支持视图、存储过程、触发器、游标
- 不支持外键、自建分区
- 不支持复合语句，如 BEGIN END、LOOP
- 不支持主备同步相关的 SQL

##### 小语法限制
###### DDL
- 不支持 CREATE TABLE ... SELECT
- 不支持 CREATE TEMPORARY TABLE
- 不支持 CREATE/DROP/ALTER SERVER/LOGFILE GROUP
- 不支持 ALTER 对分表键（shardkey）进行改名，但可以修改类型

###### DML
- 不支持 SELECT INTO OUTFILE/INTO DUMPFILE/INTO var_name
- 不支持 query_expression_options，如：HIGH_PRIORITY/STRAIGHT_JOIN/SQL_SMALL_RESULT/SQL_BIG_RESULT/SQL_BUFFER_RESULT/SQL_CACHE/SQL_NO_CACHE/SQL_CALC_FOUND_ROWS
- 不支持不带列名的 INSERT/REPLACE
- 不支持全局的 DELETE/UPDATE 使用 ORDER BY/LIMIT（版本>=1.14.4支持）
- 不支持不带 WHERE 条件的 UPDATE/DELETE
- 不支持 LOAD DATA/XML
- 不支持 SQL 中使用 DELAYED 和 LOW_PRIORITY，没有效果
- 不支持 INSERT ... SELECT（版本>1.14.4支持）
- 不支持 SQL 中对于变量的引用和操作，如：SET @c=1, @d=@c+1; SELECT @c, @d
- 不支持 index_hint
- 不支持 HANDLER/DO

###### 管理 SQL
- 不支持 ANALYZE/CHECK/CHECKSUM/OPTIMIZE/REPAIR TABLE，需要用透传语法
- 不支持 CACHE INDEX
- 不支持 FLUSH
- 不支持 KILL
- 不支持 LOAD INDEX INTO CACHE
- 不支持 RESET
- 不支持 SHUTDOWN
- 不支持 SHOW BINARY LOGS/BINLOG EVENTS
- 不支持 SHOW WARNINGS/ERRORS和LIMIT/COUNT 的组合

###### 其他限制
默认支持最大建表数量为5000，如需超越该限制，可 [提交工单](https://console.cloud.tencent.com/workorder/category) 申请。

#### 兼容性

##### 语言结构

分布式实例支持所有 MySQL 使用的文字格式，包括：
```
String Literals
Numeric Literals
Date and Time Literals
Hexadecimal Literals
Bit-Value Literals
Boolean Literals
NULL Values
```

###### String Literals
String Literals 是一个 bytes 或者 characters 的序列，两端被单引号`'`或者双引号`"`包围，TDSQL MySQL版 目前不支持 ANSI_QUOTES SQL MODE，双引号`"`包围的始终认为是 String Literals，而不是 identifier。

不支持 character set introducer，即`[_charset_name]'string' [COLLATE collation_name]`这种格式。

支持的转义字符：
```
\0: ASCII NUL (X’00’) 字符
\‘: 单引号
\“: 双引号
\b: 退格符号
\n: 换行符
\r: 回车符
\t: tab 符（制表符）
\z: ASCII 26 (Ctrl + Z)
\\: 反斜杠 \
\%: \%
\_: _
```

###### Numeric Literals
数值字面值包括 integer、Decimal 类型、浮点数字面值。
integer 可以包括`.`作为小数点分隔，数字前可以有`-`或者`+`来表示正数或者负数。
精确数值字面值可以表示为如下格式：`1, .2, 3.4, -5, -6.78, +9.10`。
科学记数法，如下格式：`1.2E3, 1.2E-3, -1.2E3, -1.2E-3`。

###### Date and Time Literals
DATE 支持如下格式：
```
'YYYY-MM-DD' or 'YY-MM-DD'
'YYYYMMDD' or 'YYMMDD'
YYYYMMDD or YYMMDD

如：'2012-12-31', '2012/12/31', '2012^12^31',  '2012@12@31'  '20070523' , '070523' 
```

DATETIME，TIMESTAMP 支持如下格式：
```
'YYYY-MM-DD HH:MM:SS' or 'YY-MM-DD HH:MM:SS'
'YYYYMMDDHHMMSS' or 'YYMMDDHHMMSS'
YYYYMMDDHHMMSS or YYMMDDHHMMSS 

如'2012-12-31 11:30:45', '2012^12^31 11+30+45', '2012/12/31 11*30*45',  '2012@12@31 11^30^45'，19830905132800 
```

###### Hexadecimal Literals
支持格式如下：
```
X'01AF'
X'01af'
x'01AF'
x'01af'
0x01AF
0x01af
```

###### Bit-Value Literals
支持格式如下：
```
b'01'
B'01'
0b01
```

###### Boolean Literals
常量 TRUE 和 FALSE 等于1和0，大小写不敏感。
```
mysql>  SELECT TRUE, true, FALSE, false;
+------+------+-------+-------+
| TRUE | TRUE | FALSE | FALSE |
+------+------+-------+-------+
|    1 |    1 |     0 |     0 |
+------+------+-------+-------+
1 row in set (0.03 sec)
```

###### NULL Values
NULL 代表数据为空，大小写不敏感，与 \N（大小写敏感）同义。
需要注意的是 NULL 跟0并不一样，跟空字符串`''`也不一样。

##### 字符集和时区
支持 MySQL 的所有字符集和字符序：
```
mysql> show character set;
+----------+---------------------------------+---------------------+--------+
| Charset  | Description                     | Default collation   | Maxlen |
+----------+---------------------------------+---------------------+--------+
| big5     | Big5 Traditional Chinese        | big5_chinese_ci     |      2 |
| dec8     | DEC West European               | dec8_swedish_ci     |      1 |
| cp850    | DOS West European               | cp850_general_ci    |      1 |
| hp8      | HP West European                | hp8_english_ci      |      1 |
| koi8r    | KOI8-R Relcom Russian           | koi8r_general_ci    |      1 |
| latin1   | cp1252 West European            | latin1_swedish_ci   |      1 |
| latin2   | ISO 8859-2 Central European     | latin2_general_ci   |      1 |
| swe7     | 7bit Swedish                    | swe7_swedish_ci     |      1 |
| ascii    | US ASCII                        | ascii_general_ci    |      1 |
| ujis     | EUC-JP Japanese                 | ujis_japanese_ci    |      3 |
| sjis     | Shift-JIS Japanese              | sjis_japanese_ci    |      2 |
| hebrew   | ISO 8859-8 Hebrew               | hebrew_general_ci   |      1 |
| tis620   | TIS620 Thai                     | tis620_thai_ci      |      1 |
| euckr    | EUC-KR Korean                   | euckr_korean_ci     |      2 |
| koi8u    | KOI8-U Ukrainian                | koi8u_general_ci    |      1 |
| gb2312   | GB2312 Simplified Chinese       | gb2312_chinese_ci   |      2 |
| greek    | ISO 8859-7 Greek                | greek_general_ci    |      1 |
| cp1250   | Windows Central European        | cp1250_general_ci   |      1 |
| gbk      | GBK Simplified Chinese          | gbk_chinese_ci      |      2 |
| latin5   | ISO 8859-9 Turkish              | latin5_turkish_ci   |      1 |
| armscii8 | ARMSCII-8 Armenian              | armscii8_general_ci |      1 |
| utf8     | UTF-8 Unicode                   | utf8_general_ci     |      3 |
| ucs2     | UCS-2 Unicode                   | ucs2_general_ci     |      2 |
| cp866    | DOS Russian                     | cp866_general_ci    |      1 |
| keybcs2  | DOS Kamenicky Czech-Slovak      | keybcs2_general_ci  |      1 |
| macce    | Mac Central European            | macce_general_ci    |      1 |
| macroman | Mac West European               | macroman_general_ci |      1 |
| cp852    | DOS Central European            | cp852_general_ci    |      1 |
| latin7   | ISO 8859-13 Baltic              | latin7_general_ci   |      1 |
| utf8mb4  | UTF-8 Unicode                   | utf8mb4_general_ci  |      4 |
| cp1251   | Windows Cyrillic                | cp1251_general_ci   |      1 |
| utf16    | UTF-16 Unicode                  | utf16_general_ci    |      4 |
| utf16le  | UTF-16LE Unicode                | utf16le_general_ci  |      4 |
| cp1256   | Windows Arabic                  | cp1256_general_ci   |      1 |
| cp1257   | Windows Baltic                  | cp1257_general_ci   |      1 |
| utf32    | UTF-32 Unicode                  | utf32_general_ci    |      4 |
| binary   | Binary pseudo charset           | binary              |      1 |
| geostd8  | GEOSTD8 Georgian                | geostd8_general_ci  |      1 |
| cp932    | SJIS for Windows Japanese       | cp932_japanese_ci   |      2 |
| eucjpms  | UJIS for Windows Japanese       | eucjpms_japanese_ci |      3 |
| gb18030  | China National Standard GB18030 | gb18030_chinese_ci  |      4 |
+----------+---------------------------------+---------------------+--------+
41 rows in set (0.02 sec)
```

查看当前连接的字符集：
```
mysql> show variables like "%char%";
+--------------------------+-----------------------------------------------------+
| Variable_name            | Value                                               |
+--------------------------+-----------------------------------------------------+
| character_set_client     | latin1                                              |
| character_set_connection | latin1                                              |
| character_set_database   | utf8                                                |
| character_set_filesystem | binary                                              |
| character_set_results    | latin1                                              |
| character_set_server     | utf8                                                |
| character_set_system     | utf8                                                |
| character_sets_dir       | /data/tdsql_run/8812/percona-5.7.17/share/charsets/ |
+--------------------------+-----------------------------------------------------+
```

设置当前连接相关的字符集：
```
mysql> set names utf8;
Query OK, 0 rows affected (0.03 sec)

mysql> show variables like "%char%";
+--------------------------+-----------------------------------------------------+
| Variable_name            | Value                                               |
+--------------------------+-----------------------------------------------------+
| character_set_client     | utf8                                                |
| character_set_connection | utf8                                                |
| character_set_database   | utf8                                                |
| character_set_filesystem | binary                                              |
| character_set_results    | utf8                                                |
| character_set_server     | utf8                                                |
| character_set_system     | utf8                                                |
| character_sets_dir       | /data/tdsql_run/8811/percona-5.7.17/share/charsets/ |
+--------------------------+-----------------------------------------------------+
```

>?分布式实例不支持设置全局参数，需要调用前台接口设置。

支持通过设置 time_zone 变量修改时区相关的属性：
```
mysql> show variables like '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | SYSTEM |
+------------------+--------+
2 rows in set (0.00 sec)

mysql> create table test.tt (ts timestamp, dt datetime,c int) shardkey=c;
Query OK, 0 rows affected (0.49 sec)

mysql>  insert into test.tt (ts,dt,c)values ('2017-10-01 12:12:12', '2017-10-01 12:12:12',1);
Query OK, 1 row affected (0.09 sec)

mysql> select * from test.tt;
+---------------------+---------------------+------+
| ts                  | dt                  | c    |
+---------------------+---------------------+------+
| 2017-10-01 12:12:12 | 2017-10-01 12:12:12 |    1 |
+---------------------+---------------------+------+
1 row in set (0.04 sec)

mysql> set time_zone = '+12:00';
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | +12:00 |
+------------------+--------+
2 rows in set (0.00 sec)

mysql> select * from test.tt;
+---------------------+---------------------+------+
| ts                  | dt                  | c    |
+---------------------+---------------------+------+
| 2017-10-01 16:12:12 | 2017-10-01 12:12:12 |    1 |
+---------------------+---------------------+------+
1 row in set (0.06 sec)
```

##### 数据类型
支持 MySQL 的所有数据类型，包括数字，字符，日期，空间类型，JSON。

###### 数字
整型支持 INTEGER， INT，SMALLINT，TINYINT，MEDIUMINT，BIGINT

| 类型      | 字节数 | 最小值（有符号/无符号）  | 最大值（有符号/无符号）                 |
| --------- | ------ | ---------------------- | ---------------------------------------- |
| TINYINT   | 1      | -128/0                 | 127/255                                  |
| SMALLINT  | 2      | -32768/0               | 32767/65535                              |
| MEDIUMINT | 3      | -8388608/0             | 8388607/16777215                         |
| INT       | 4      | -2147483648/0          | 2147483647/4294967295                    |
| BIGINT    | 8      | -9223372036854775808/0 | 9223372036854775807/18446744073709551615 |

浮点类型支持 FLOAT 和 DOUBLE，格式 FLOAT(M,D) 、REAL(M,D) 或 DOUBLE PRECISION(M,D)

定点类型支持 DECIMAL和NUMERIC，格式 DECIMAL(M,D)

###### 字符
支持如下字符类型：
```
CHAR 和 VARCHAR Types
BINARY 和 VARBINARY Types
BLOB 和 TEXT Types
	TINYBLOB，TINYTEXT，MEDIUMBLOB,MEDIUMTEXT，LONGBLOB，LONGTEXT
ENUM Type
SET Type
```

###### 日期
支持如下时间类型：
```
DATE, DATETIME,  TIMESTAMP Types
TIME Type
YEAR Type
```

###### 空间
支持如下空间类型：
```
GEOMETRY
POINT
LINESTRING
POLYGON

MULTIPOINT
MULTILINESTRING
MULTIPOLYGON
GEOMETRYCOLLECTION
```

###### JSON
支持存储 JSON 格式的数据，使得对 JSON 处理更加有效，同时又能提早检查错误：
>!对 JSON 类型的字段进行排序时，不支持混合类型排序。如不能将 string 类型和 int 类型做比较，同类型排序只支持数值类型，string 类型，其它类型排序不处理。对下表来说，不支持`select * from t1 order by t1->"$.key2"`，因为排序列中包含了数值和字符串类型。

```sql
mysql>  CREATE TABLE t1 (jdoc JSON,a int) shardkey=a;
Query OK, 0 rows affected (0.30 sec)

mysql> INSERT INTO t1 (jdoc,a)VALUES('{"key1": "value1", "key2": "value2"}',1);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO t1 (jdoc,a)VALUES('{"key1": "value1", "key2": 2}',2);

mysql> INSERT INTO t1 (jdoc,a)VALUES('[1, 2,',5);
ERROR 3140 (22032): Invalid JSON text: "Invalid value." at position 6 in value for column 't1.jdoc'.

mysql> select * from t1;
+--------------------------------------+---+
| jdoc                                 | a |
+--------------------------------------+---+
| {"key1": "value1", "key2": "value2"} | 1 |
| {"key1": "value1", "key2": 2}        | 2 |
+--------------------------------------+---+
2 rows in set (0.00 sec)

```



##### 函数支持
Control Flow Functions

| Name     | Description                  |
| -------- | ---------------------------- |
| CASE     | Case operator                |
| IF()     | If/else construct            |
| IFNULL() | Null if/else construct       |
| NULLIF() | Return NULL if expr1 = expr2 |


String Functions

| Name               | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| ASCII()            | Return numeric value of left-most character                  |
| BIN()              | Return a string containing binary representation of a number |
| BIT_LENGTH()       | Return length of argument in bits                            |
| CHAR()             | Return the character for each integer passed                 |
| CHAR_LENGTH()      | Return number of characters in argument                      |
| CHARACTER_LENGTH() | Synonym for CHAR_LENGTH()                                    |
| CONCAT()           | Return concatenated string                                   |
| CONCAT_WS()        | Return concatenate with separator                            |
| ELT()              | Return string at index number                                |
| EXPORT_SET()       | Return a string such that for every bit set in the value bits, you get an on string and for every unset bit, you get an off string |
| FIELD()            | Return the index (position) of the first argument in the subsequent arguments |
| FIND_IN_SET()      | Return the index position of the first argument within the second argument |
| FORMAT()           | Return a number formatted to specified number of decimal places |
| FROM_BASE64()      | Decode to a base-64 string and return result                 |
| HEX()              | Return a hexadecimal representation of a decimal or string value |
| INSERT()           | Insert a substring at the specified position up to the specified number of characters |
| INSTR()            | Return the index of the first occurrence of substring        |
| LCASE()            | Synonym for LOWER()                                          |
| LEFT()             | Return the leftmost number of characters as specified        |
| LENGTH()           | Return the length of a string in bytes                       |
| LIKE               | Simple pattern matching                                      |
| LOAD_FILE()        | Load the named file                                          |
| LOCATE()           | Return the position of the first occurrence of substring     |
| LOWER()            | Return the argument in lowercase                             |
| LPAD()             | Return the string argument, left-padded with the specified string |
| LTRIM()            | Remove leading spaces                                        |
| MAKE_SET()         | Return a set of comma-separated strings that have the corresponding bit in bits set |
| MATCH              | Perform full-text search                                     |
| MID()              | Return a substring starting from the specified position      |
| NOT LIKE           | Negation of simple pattern matching                          |
| NOT REGEXP         | Negation of REGEXP                                           |
| OCT()              | Return a string containing octal representation of a number  |
| OCTET_LENGTH()     | Synonym for LENGTH()                                         |
| ORD()              | Return character code for leftmost character of the argument |
| POSITION()         | Synonym for LOCATE()                                         |
| QUOTE()            | Escape the argument for use in an SQL statement              |
| REGEXP             | Pattern matching using regular expressions                   |
| REPEAT()           | Repeat a string the specified number of times                |
| REPLACE()          | Replace occurrences of a specified string                    |
| REVERSE()          | Reverse the characters in a string                           |
| RIGHT()            | Return the specified rightmost number of characters          |
| RLIKE              | Synonym for REGEXP                                           |
| RPAD()             | Append string the specified number of times                  |
| RTRIM()            | Remove trailing spaces                                       |
| SOUNDEX()          | Return a soundex string                                      |
| SOUNDS LIKE        | Compare sounds                                               |
| SPACE()            | Return a string of the specified number of spaces            |
| STRCMP()           | Compare two strings                                          |
| SUBSTR()           | Return the substring as specified                            |
| SUBSTRING()        | Return the substring as specified                            |
| SUBSTRING_INDEX()  | Return a substring from a string before the specified number of occurrences of the delimiter |
| TO_BASE64()        | Return the argument converted to a base-64 string            |
| TRIM()             | Remove leading and trailing spaces                           |
| UCASE()            | Synonym for UPPER()                                          |
| UNHEX()            | Return a string containing hex representation of a number    |
| UPPER()            | Convert to uppercase                                         |
| WEIGHT_STRING()    | Return the weight string for a string                        |


Numeric Functions and Operators

| Name            | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| ABS()           | Return the absolute value                                    |
| ACOS()          | Return the arc cosine                                        |
| ASIN()          | Return the arc sine                                          |
| ATAN()          | Return the arc tangent                                       |
| ATAN2(), ATAN() | Return the arc tangent of the two arguments                  |
| CEIL()          | Return the smallest integer value not less than the argument |
| CEILING()       | Return the smallest integer value not less than the argument |
| CONV()          | Convert numbers between different number bases               |
| COS()           | Return the cosine                                            |
| COT()           | Return the cotangent                                         |
| CRC32()         | Compute a cyclic redundancy check value                      |
| DEGREES()       | Convert radians to degrees                                   |
| DIV             | Integer division                                             |
| /               | Division operator                                            |
| EXP()           | Raise to the power of                                        |
| FLOOR()         | Return the largest integer value not greater than the argument |
| LN()            | Return the natural logarithm of the argument                 |
| LOG()           | Return the natural logarithm of the first argument           |
| LOG10()         | Return the base-10 logarithm of the argument                 |
| LOG2()          | Return the base-2 logarithm of the argument                  |
| -               | Minus operator                                               |
| MOD()           | Return the remainder                                         |
| %, MOD          | Modulo operator                                              |
| PI()            | Return the value of pi                                       |
| +               | Addition operator                                            |
| POW()           | Return the argument raised to the specified power            |
| POWER()         | Return the argument raised to the specified power            |
| RADIANS()       | Return argument converted to radians                         |
| RAND()          | Return a random floating-point value                         |
| ROUND()         | Round the argument                                           |
| SIGN()          | Return the sign of the argument                              |
| SIN()           | Return the sine of the argument                              |
| SQRT()          | Return the square root of the argument                       |
| TAN()           | Return the tangent of the argument                           |
| *               | Multiplication operator                                      |
| TRUNCATE()      | Truncate to specified number of decimal places               |
| -               | Change the sign of the argument                              |


Date and Time Functions

| Name                                   | Description                                                  |
| -------------------------------------- | ------------------------------------------------------------ |
| ADDDATE()                              | Add time values (intervals) to a date value                  |
| ADDTIME()                              | Add time                                                     |
| CONVERT_TZ()                           | Convert from one time zone to another                        |
| CURDATE()                              | Return the current date                                      |
| CURRENT_DATE(), CURRENT_DATE           | Synonyms for CURDATE()                                       |
| CURRENT_TIME(), CURRENT_TIME           | Synonyms for CURTIME()                                       |
| CURRENT_TIMESTAMP(), CURRENT_TIMESTAMP | Synonyms for NOW()                                           |
| CURTIME()                              | Return the current time                                      |
| DATE()                                 | Extract the date part of a date or datetime expression       |
| DATE_ADD()                             | Add time values (intervals) to a date value                  |
| DATE_FORMAT()                          | Format date as specified                                     |
| DATE_SUB()                             | Subtract a time value (interval) from a date                 |
| DATEDIFF()                             | Subtract two dates                                           |
| DAY()                                  | Synonym for DAYOFMONTH()                                     |
| DAYNAME()                              | Return the name of the weekday                               |
| DAYOFMONTH()                           | Return the day of the month (0-31)                           |
| DAYOFWEEK()                            | Return the weekday index of the argument                     |
| DAYOFYEAR()                            | Return the day of the year (1-366)                           |
| EXTRACT()                              | Extract part of a date                                       |
| FROM_DAYS()                            | Convert a day number to a date                               |
| FROM_UNIXTIME()                        | Format Unix timestamp as a date                              |
| GET_FORMAT()                           | Return a date format string                                  |
| HOUR()                                 | Extract the hour                                             |
| LAST_DAY                               | Return the last day of the month for the argument            |
| LOCALTIME(), LOCALTIME                 | Synonym for NOW()                                            |
| LOCALTIMESTAMP, LOCALTIMESTAMP()       | Synonym for NOW()                                            |
| MAKEDATE()                             | Create a date from the year and day of year                  |
| MAKETIME()                             | Create time from hour, minute, second                        |
| MICROSECOND()                          | Return the microseconds from argument                        |
| MINUTE()                               | Return the minute from the argument                          |
| MONTH()                                | Return the month from the date passed                        |
| MONTHNAME()                            | Return the name of the month                                 |
| NOW()                                  | Return the current date and time                             |
| PERIOD_ADD()                           | Add a period to a year-month                                 |
| PERIOD_DIFF()                          | Return the number of months between periods                  |
| QUARTER()                              | Return the quarter from a date argument                      |
| SEC_TO_TIME()                          | Converts seconds to 'HH:MM:SS' format                        |
| SECOND()                               | Return the second (0-59)                                     |
| STR_TO_DATE()                          | Convert a string to a date                                   |
| SUBDATE()                              | Synonym for DATE_SUB() when invoked with three arguments     |
| SUBTIME()                              | Subtract times                                               |
| SYSDATE()                              | Return the time at which the function executes               |
| TIME()                                 | Extract the time portion of the expression passed            |
| TIME_FORMAT()                          | Format as time                                               |
| TIME_TO_SEC()                          | Return the argument converted to seconds                     |
| TIMEDIFF()                             | Subtract time                                                |
| TIMESTAMP()                            | With a single argument, this function returns the date or datetime expression; with two arguments, the sum of the arguments |
| TIMESTAMPADD()                         | Add an interval to a datetime expression                     |
| TIMESTAMPDIFF()                        | Subtract an interval from a datetime expression              |
| TO_DAYS()                              | Return the date argument converted to days                   |
| TO_SECONDS()                           | Return the date or datetime argument converted to seconds since Year 0 |
| UNIX_TIMESTAMP()                       | Return a Unix timestamp                                      |
| UTC_DATE()                             | Return the current UTC date                                  |
| UTC_TIME()                             | Return the current UTC time                                  |
| UTC_TIMESTAMP()                        | Return the current UTC date and time                         |
| WEEK()                                 | Return the week number                                       |
| WEEKDAY()                              | Return the weekday index                                     |
| WEEKOFYEAR()                           | Return the calendar week of the date (1-53)                  |
| YEAR()                                 | Return the year                                              |
| YEARWEEK()                             | Return the year and week                                     |

Aggregate (GROUP BY) Functions

| Name    | Description                                   |
| ------- | --------------------------------------------- |
| AVG()   | Return the average value of the argument      |
| COUNT() | Return a count of the number of rows returned |
| MAX()   | Return the maximum value                      |
| MIN()   | Return the minimum value                      |
| SUM()   | Return the sum                                |


Bit Functions and Operators

| Name        | Description                            |
| ----------- | -------------------------------------- |
| BIT_COUNT() | Return the number of bits that are set |
| &           | Bitwise AND                            |
| ~           | Bitwise inversion                      |
| \|     | Bitwise OR                             |
| ^           | Bitwise XOR                            |
| <<          | Left shift                             |
| >>          | Right shift                            |


Cast Functions and Operators

| Name      | Description                      |
| --------- | -------------------------------- |
| BINARY    | Cast a string to a binary string |
| CAST()    | Cast a value as a certain type   |
| CONVERT() | Cast a value as a certain type   |

#### 连接数据库

##### 客户端连接
TDSQL MySQL版 提供和 MySQL 兼容的连接方式，用户可通过 IP 地址、端口号以及用户名、密码连接 TDSQL MySQL版：
```
mysql -hxxx.xxx.xxx.xxx -Pxxxx -uxxx -pxxx -c
```
>!TDSQL MySQL版 不支持4.0以下的版本以及压缩协议，建议在使用客户端的时候增加`-c`选项，以便于使用某些高级功能。

##### PHP MySQLli 连接
PHP 需要开启 MySQLli 扩展连接数据库，具体 demo 如下：
```
header("Content-Type:text/html;charset=utf-8");
$host="10.10.10.10";  //实例的 proxy_host_ip
$user="test";  //实例用户
$pwd="test";  //实例用户密码
$db="aaa";  //数据库名
$port="15002";  //proxy_host 端口号
$sqltool=new MySQLli($host,$user,$pwd,$db,$port);
//其他必要代码
$sqltool->close();
echo "ok"."\n";
```

##### JDBC 连接
您也可以使用 JDBC 连接 TDSQL MySQL版，例如：
```
private final String USERNAME = "test";  
private final String PASSWORD = "123456"; 
private final String DRIVER = "com.mysql.jdbc.Driver";   
private final String URL = "jdbc:mysql://10.10.10.10:3306?userunicode=true&characterEncoding=utf8mb4";  
private Connection connection;  
private PreparedStatement pstmt;  
private ResultSet resultSet;  
```

##### 其他连接方式
您也可以选择其他兼容 MySQL 的连接方式，例如 navicat、odbc 等。

#### JOIN 和子查询

对于分布式实例，数据水平拆分在各个节点，为提高性能，建议优先优化表结构和 SQL，尽量使用不跨节点的方式。

##### 推荐方式
###### 多个分表，带有分表键相等的条件
```sql
MySQL > select * from test1 join test2 where test1.a=test2.a;     
+---+------+---------+---+------+---------------+
| a | b    | c       | a | d    | e             |
+---+------+---------+---+------+---------------+
| 1 |    2 | record1 | 1 |    3 | test2_record1 |
| 2 |    3 | record2 | 2 |    3 | test2_record2 |
+---+------+---------+---+------+---------------+
2 rows in set (0.00 sec)

MySQL > select * from test1 left join test2 on test1.a<test2.a where test1.a=1;
+---+------+---------+------+------+---------------+
| a | b    | c       | a    | d    | e             |
+---+------+---------+------+------+---------------+
| 1 |    2 | record1 |    2 |    3 | test2_record2 |
+---+------+---------+------+------+---------------+
1 row in set (0.00 sec)

MySQL> select * from test1 where test1.a in (select a from test2);        
+---+------+---------+
| a | b    | c       |
+---+------+---------+
| 1 |    2 | record1 |
| 2 |    3 | record2 |
+---+------+---------+
2 rows in set (0.00 sec)

MySQL> select a, count(1) from test1 where exists (select * from test2 where test2.a=test1.a) group by a;        
+---+----------+
| a | count(1) |
+---+----------+
| 1 |        1 |
| 2 |        1 |
+---+----------+
2 rows in set (0.00 sec)

MySQL> select distinct count(1) from test1 where exists (select * from test2 where test2.a=test1.a) group by a;   
+----------+
| count(1) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)

MySQL> select count(distinct a) from test1 where exists (select * from test2 where test2.a=test1.a);        
+-------------------+
| count(distinct a) |
+-------------------+
|                 2 |
+-------------------+
1 row in set (0.00 sec)
```

###### 均为单表
```sql
mysql> create table noshard_table ( a int, b int key);
Query OK, 0 rows affected (0.02 sec)

mysql> create table noshard_table_2 ( a int, b int key);
Query OK, 0 rows affected (0.00 sec)

mysql> select * from noshard_table,noshard_table_2;
Empty set (0.00 sec)

mysql> insert into noshard_table (a,b) values(1,2),(3,4);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> insert into noshard_table_2 (a,b) values(10,20),(30,40);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from noshard_table,noshard_table_2;
+------+---+------+----+
| a    | b | a    | b  |
+------+---+------+----+
|    1 | 2 |   10 | 20 |
|    3 | 4 |   10 | 20 |
|    1 | 2 |   30 | 40 |
|    3 | 4 |   30 | 40 |
+------+---+------+----+
4 rows in set (0.00 sec)
```

###### 广播表
```sql
 MySQL> create table global_test(a int key, b int)shardkey=noshardkey_allset;
Query OK, 0 rows affected (0.00 sec)

MySQL> insert into global_test(a, b) values(1,1),(2,2);
Query OK, 2 rows affected (0.00 sec)

MySQL> select * from test1, global_test;
+---+------+---------+---+------+
| a | b    | c       | a | b    |
+---+------+---------+---+------+
| 1 |    2 | record1 | 1 |    1 |
| 2 |    3 | record2 | 1 |    1 |
| 1 |    2 | record1 | 2 |    2 |
| 2 |    3 | record2 | 2 |    2 |
+---+------+---------+---+------+
4 rows in set (0.00 sec)
```

###### 子查询带有 shardkey 的 derived table
```
mysql> select a from (select * from test1 where a=1) as t;
+---+
| a |
+---+
| 1 |
+---+
1 row in set (0.00 sec)
```
>?子查询时不指定 shardkey，即可查询结果。

##### 复杂 SQL
对于不能满足推荐方式的 SQL，由于需要做跨节点的数据交互，所以性能会差一些。
包括：
- 包含子查询的查询。
- 多表的 join 查询，且参与查询的各表的分区字段（shardkey）不相等，或者同时涉及不同类型的表，例如单表和分表。

对于这类复杂查询，通过条件下推，将真正参与查询的数据从后端数据库中抽取出来，存放在本地的临时表中，然后对临时表中的数据进行计算。

因此用户需要明确指定参与查询的表的条件，避免因为抽取大量数据而性能受损。
```sql
mysql> create table test1 ( a int key, b int, c char(20) ) shardkey=a;
Query OK, 0 rows affected (1.56 sec)

mysql> create table test2 ( a int key, d int, e char(20) ) shardkey=a;
Query OK, 0 rows affected (1.46 sec)

mysql> insert into test1 (a,b,c) values(1,2,"record1"),(2,3,"record2");
Query OK, 2 rows affected (0.02 sec)

mysql> insert into test2 (a,d,e) values(1,3,"test2_record1"),(2,3,"test2_record2");
Query OK, 2 rows affected (0.02 sec)

mysql> select * from test1 join test2 on test1.b=test2.d;
+---+------+---------+---+------+---------------+
| a | b    | c       | a | d    | e             |
+---+------+---------+---+------+---------------+
| 2 |    3 | record2 | 1 |    3 | test2_record1 |
| 2 |    3 | record2 | 2 |    3 | test2_record2 |
+---+------+---------+---+------+---------------+
2 rows in set (0.00 sec)

MySQL> select * from test1 where exists (select * from test2 where test2.a=test1.b);
+---+------+---------+
| a | b    | c       |
+---+------+---------+
| 1 |    2 | record1 |
+---+------+---------+
1 row in set (0.00 sec)
```

分布式实例还支持丰富的复杂 update/delete/insert 操作。

需要注意的是，这类查询是在与之对应的 select 基础上实现的，因此也需要将数据加载至网关临时表，建议用户尽量在查询中指定明确的查询条件，避免大量数据的加载带来性能损耗。
另外，网关在加载数据时默认不会对加载的数据进行上锁，这与官方的 MySQL 行为存在略微的差异；如需加锁可以通过修改 proxy 配置来实现。

```sql
MySQL [th]> update test1 set test1.c="record" where exists(select 1 from test2 where test1.b=test2.d);
Query OK, 1 row affected (0.00 sec)

MySQL [th]> update test1, test2 set test1.b=2 where test1.b=test2.d;
Query OK, 1 row affected (0.00 sec)

MySQL [th]> insert into test1 select cast(rand()*1024 as unsigned), d, e from test2;
Query OK, 2 rows affected (0.00 sec)

MySQL [th]> delete from test1 where b in (select b from test2);
Query OK, 6 rows affected (0.00 sec)

MySQL [th]> delete from test2.* using test1 right join test2 on test1.a=test2.a where test1.a is null;
Query OK, 2 rows affected (0.00 sec)
```


#### Sequence

关键字 Sequence 语法和 MariaDB/Oracle 兼容，但是保证分布式全局递增且唯一，具体使用如下：

>?
>- 在 TDSQL MySQL版 分布式数据库当中使用 Sequence 时，须在该关键字前面加 `tdsql_` 前缀，且要求 proxy 版本最低为1.19.5-M-V2.0R745D005；可通过数据库管理语句 `/*Proxy*/show status` 查询 proxy 版本，若 proxy 版本较老可以 [提交工单](https://console.cloud.tencent.com/workorder/category) 进行升级。
>- 目前 Sequence 为保证分布式全局数值唯一，导致性能较差，主要适用于并发不高的场景。

创建序列需要 CREATE SEQUENCE 系统权限。序列的创建语法如下：
```
　　CREATE TDSQL_SEQUENCE 序列名
　　[START WITH n]
　　[{TDSQL_MINALUE/ TDSQL_MAXMINVALUE n| TDSQL_NOMAXVALUE}]
　　[TDSQL_INCREMENT BY n]
　　[{TDSQL_CYCLE|TDSQL_NOCYCLE}]
```

##### 创建 Sequence
```
create tdsql_sequence test.s1 start with 12 tdsql_minvalue 10 maxvalue 50000 tdsql_increment by 5 tdsql_nocycle
create tdsql_sequence test.s2 start with 12 tdsql_minvalue 10 maxvalue 50000 tdsql_increment by 1 tdsql_cycle
```
- 以上SQL语句包含开始值、最小值、最大值、步长、缓存大小及是否回绕6个参数，参数都应为正整数。
- 参数默认值，开始值（1）、最小值（1）、最大值（LONGLONG_MAX-1）、步长（1）、是否回绕（0）。

##### 删除 Sequence
```
drop tdsql_sequence test.s1
```

##### 查询 Sequence
```
show create tdsql_sequence test.s2
```

##### 使用 Sequence
####### 使用 Sequence 获取下一个数值
```
select tdsql_nextval(test.s2)
select next value for test.s2
```

```
mysql> select tdsql_nextval(test.s1);
+----+
| 12 |
+----+
| 12 |
+----+
1 row in set (0.18 sec)

mysql> select tdsql_nextval(test.s2);
+----+
| 12 |
+----+
| 12 |
+----+
1 row in set (0.13 sec)

mysql> select tdsql_nextval(test.s1);
+----+
| 17 |
+----+
| 17 |
+----+
1 row in set (0.01 sec)

mysql> select tdsql_nextval(test.s2);
+----+
| 13 |
+----+
| 13 |
+----+
1 row in set (0.00 sec)

mysql> select next value for test.s1;
+----+
| 22 |
+----+
| 22 |
+----+
1 row in set (0.01 sec)
```

####### nextval 可以用在 insert 等地方
```
mysql> select * from test.t1;
+----+------+
| a  | b    |
+----+------+
| 11 |    2 |
+----+------+
1 row in set (0.00 sec)

mysql> insert into test.t1(a,b) values(tdsql_nextval(test.s2),3);
Query OK, 1 row affected (0.01 sec)

mysql> select * from test.t1;
+----+------+
| a  | b    |
+----+------+
| 11 |    2 |
| 14 |    3 |
+----+------+
2 rows in set (0.00 sec)
```

如需获取上一次的值以连接相关数据：如果之前没有用 nextval 命令获取过数据，数值将返回为0。
```
select tdsql_lastval(test.s1)
select tdsql_previous value for test.s1;
```

```
mysql> select tdsql_lastval(test.s1);
+----+
| 22 |
+----+
| 22 |
+----+
1 row in set (0.00 sec)

mysql> select tdsql_previous value for test.s1;
+----+
| 22 |
+----+
| 22 |
+----+
1 row in set (0.00 sec)
```

设置下一个序列数值，只能比当前数值大，否则将返回数值为0。
```
select tdsql_setval(test.s2,1000,bool use)  //  use 默认为1，表示1000这个值用过了，下一次不包含1000，如果为0，则下一个从1000开始。
```

设置下一个序列数值时，如果比当前数值小，则系统将没有反应。
```
mysql> select tdsql_nextval(test.s2);
+----+
| 15 |
+----+
| 15 |
+----+
1 row in set (0.01 sec)

mysql> select tdsql_setval(test.s2,10);
+---+
| 0 |
+---+
| 0 |
+---+
1 row in set (0.03 sec)

mysql> select tdsql_nextval(test.s2);
+----+
| 16 |
+----+
| 16 |
+----+
```

如果比当前数值大，成功返回当前设置的值。
```
mysql> select tdsql_setval(test.s2,20);
+----+
| 20 |
+----+
| 20 |
+----+
1 row in set (0.02 sec)
mysql> select tdsql_nextval(test.s2);
+----+
| 21 |
+----+
| 21 |
+----+
1 row in set (0.01 sec)
```

强制设置下一个序列数值，允许设置比当前数值小的值。
```
select tdsql_resetval(test.s2,1000)
```

如果强制设置成功，返回当前设置的值，下一个序列数值从该值开始。
```
mysql> select tdsql_resetval(test.s2,14);
+----+
| 14 |
+----+
| 14 |
+----+
1 row in set (0.00 sec)

mysql> select tdsql_nextval(test.s2);
+----+
| 14 |
+----+
| 14 |
+----+
1 row in set (0.01 sec)
```

需要注意，Sequence 的部分关键字以 `TDSQL_` 前缀开始：
```
 TDSQL_CYCLE
 TDSQL_INCREMENT
 TDSQL_LASTVAL  
 TDSQL_MINVALUE 
 TDSQL_NEXTVAL  
 TDSQL_NOCACHE  
 TDSQL_NOCYCLE  
 TDSQL_NOMAXVALUE
 TDSQL_NOMINVALUE
 TDSQL_PREVIOUS 
 TDSQL_RESTART  
 TDSQL_REUSE    
 TDSQL_SEQUENCE 
 TDSQL_SETVAL   
```



#### 常用 DML

##### select
建议带上 shardkey 字段，proxy 根据该字段的 hash 值直接将 SQL 请求路由至对应的数据库实例进行处理；否则就需要发送给集群中所有的数据库实例执行，然后 proxy 根据数据库返回的结果集进行聚合，影响执行效率：
```
mysql> select * from test1 where a=2;
+------+------+---------+
| a    | b    | c       |
+------+------+---------+
|    2 |    3 | record2 |
|    2 |    4 | record3 |
+------+------+---------+
2 rows in set (0.00 sec)
```

##### insert/replace
字段必须包含 shardkey，否则会拒绝执行该 SQL，因为 proxy 不知道将该 SQL 发往哪个后端数据库：
```
mysql> insert into test1 (b,c) values(4,"record3");
ERROR 810 (HY000): Proxy ERROR:sql is too complex,need to send to only noshard table.
	Shard table insert must has field spec

mysql> insert into test1 (a,c) values(4,"record3");
Query OK, 1 row affected (0.01 sec)
```

##### delete/update
使用分表时，为安全考虑，执行该类 SQL 时，必须带有 where 条件，否则拒绝执行该 SQL 命令：
```
mysql> delete from test1;
ERROR 810 (HY000): Proxy ERROR:sql is too complex,need to send to only noshard table.
	Shard table delete/update must have a where clause

mysql> delete from test1 where a=1;
Query OK, 1 row affected (0.01 sec)
```



#### 读写分离


TDSQL MySQL版 实例支持下列几种模式的读写分离：
- 通过增加 slave 注释标记，即在 SQL 中添加 /`*slave*/` 这样的标记，并且 mysql 后面增加 -c 参数来解析注释 `mysql -c -e "/*slave*/sql"`，该 SQL 会发送给备机。
>?支持` /*slave:slaveonly*/`、` /*slave:20*/`、` /*slave:slaveonly,20*/`这几种形式，数值表示 slave 应该满足的延迟，slaveonly 表示在没有符合条件的 slave 时，不会将查询发送给主节点。
>
```
//主机读//
select * from emp order by sal，deptno desc；
//从机读//
/*slave*/ select * from emp order by sal，deptno desc；
```
- 由只读帐号发送的请求会根据配置的属性发给备机。
  ![](https://qcloudimg.tencent-cloud.cn/raw/bc5207ff544069ce434b6b2ea45193ae.png)

  
#### 分布式事务


由于事务操作的数据通常跨多个物理节点，在分布式数据库中，类似方案即称为分布式事务。
TDSQL MySQL版 支持普通分布式事务协议和 XA 分布式事务协议。TDSQL MySQL版（内核5.7或以上版本）默认支持分布式事务，且对客户端透明，像使用单机事务一样方便。
TDSQL MySQL版 分布式事务采用两阶段提交算法（2PC）保证事务的原子性（Atomicity）和一致性（Consistency），隔离级别配置为 Read committed、Repeatable read 或 Serializable。

##### 普通分布式事务
```
begin; 	# 开启事务
... 	   # 跨 set 的增删改查等非 DDL 操作
commit;    # 提交事务
```

##### XA 分布式事务
XA 分布式事务是指跨实例的事务：
```
xa begin ''; 		# 开启 XA 事务，事务标识由系统内部生成，因此传入空字符串
...			      # 跨 set 的增删改查等非 DDL 操作
select gtid();   	# 获取当前 XA 事务的标识，下面假定为'xid'
xa prepare 'xid';	# 准备事务
xa commit/rollback 'xid'; # 提交或回滚事务
```

##### 新增事务接口
- `select gtid()` ：获取当前分布式事务的全局唯一标识。如果为空，则该事务不是分布式事务。
- 普通分布式事务标识的格式为：‘网关id’-‘proxy随机值’-‘序列号’-‘时间戳’-‘分区号’，例如 c46535fe-b6-dd-595db6b8-25。
- XA 分布式事务标识的格式为：‘ex’-‘网关id’-‘proxy随机值’-‘序列号’-‘时间戳’-‘分区号’，例如 ex-c46535fe-b6-dd-595db6b8-25。

- `select gtid_state(“当前分布式事务的全局唯一标识”)`：在事务提交异常之后（默认3秒后）用来获取事务的状态。可能的结果有：
- COMMIT：标识该事务已经或者最终会被提交。
- ABORT：标识该事务最终会被回滚。
- 空：由于事务的状态会在一个小时之后清除，因此有以下两种可能：
	- 一个小时之后查询，标识事务状态已经清除,
	- 一个小时以内查询，标识事务最终会被回滚。

- `xa boost ‘当前分布式事务的全局唯一标识’ `：普通事务提交（commit）发送异常之后，事务在一段时间内（默认30秒）由后台组件自动提交或者回滚掉。如果用户不愿意等待这么长的时间，可以反复调用该接口，促使系统及时地提交或回滚掉事务。该接口会返回事务的状态，即提交或者回滚。

- `xa lockwait`：显示当前分布式事务的等待关系。用户可以通过 dot 工具，将其转化为图片。

- `xa show`：显示当前 proxy 上处于活跃状态的事务。


#### 建表


##### 建分表
分表创建时必须在最后面指定分表键（shardkey）的值，该值为表中的一个字段名字，会用于后续 SQL 的路由选择：
```
mysql> create table test1 ( a int, b int, c char(20),primary key (a,b),unique key u_1(a,c) ) shardkey=a;
Query OK, 0 rows affected (0.07 sec)
```

在分布式实例中，shardkey 对应后端数据库的分区字段，因此每一个唯一索引和主键都必须要包含这个 shardkey，否则无法创建表。
场景：存在多个唯一索引时报错。
```
mysql> create table test1 ( a int, b int, c char(20),primary key (a,b),unique key u_1(a,c),unique key u_2(b,c) ) shardkey=a;
```
此时有一个唯一索引`u_2`不包含 shardkey，无法创建表，会报如下错误：
```
ERROR 1105 (HY000): A UNIQUE INDEX must include all columns in the table's partitioning function
```
因为主键索引或者 unique key 索引意味着需要全局唯一，而要实现全局唯一索引，则必须包含 shardkey 字段。


除上面的限制外，shardkey 字段还有如下要求：
- shardkey 字段的类型必须是 int、bigint、smallint、char、varchar。
- shardkey 字段类型为 char、varchar 时需定义字段长度。
- shardkey 字段的值不能有中文，proxy 不会转换字符集，因此不同字符集可能会路由到不同的分区。
- 不能 update shardkey 字段的值。
- shardkey=a 放在 SQL 的最后面。
- 访问数据尽量都带上 shardkey 字段，非强制要求，但是不带 shardkey 的 SQL 会路由到所有节点，消耗较多资源。

##### 建广播表
支持建小表（广播表），此时该表在所有 set 中都是全量数据，主要方便于跨 set 的 join 操作，同时通过分布式事务保证修改操作的原子性，使得所有 set 的数据完全一致。
```
mysql> create table global_table ( a int, b int key) shardkey=noshardkey_allset;
Query OK, 0 rows affected (0.06 sec)
```

##### 建单表
支持建立普通的表，语法和 MySQL 完全一致，此时该表的数据全量存在第一个 set 中，所有该类型的表都放在第一个 set 中：
```
mysql> create table noshard_table ( a int, b int key);
Query OK, 0 rows affected (0.02 sec)
```


#### 连接保护

连接保护用于 proxy 与后端数据库连接断开时（如后端数据库发生了异常，但异常未影响 proxy），保持客户端和 proxy 之间的连接不断开。

proxy 正在执行 SQL 时，如果与数据库的连接断开，则 proxy 断开与该 SQL 相关的所有连接（除 proxy 与客户端之间的连接），并告知用户错误：
```c++
ER_PROXY_TRANSACTION_ERROR // 在事务中
ER_PROXY_CONN_BROKEN_ERROR // 非事务中
```

##### 处理方式

- 如果 proxy 与后端数据库连接断开时，用户 session 处于`普通事务`中，处理如下（图中错误码均为 ER_PROXY_TRANSACTION_ERROR）：
  ![](https://main.qcloudimg.com/raw/a758c8c5d73ab6a54c62bb86d971131b.png)

- 如果 proxy 与后端数据库连接断开时，用户正处于`XA 事务`中，处理如下（图中错误码均为 ER_PROXY_TRANSACTION_ERROR）：
  ![](https://main.qcloudimg.com/raw/e15b45ae460c0ddfc7a60fa3c21cf3c4.png)

##### 超时配置
>?用户在事务中 proxy 与后端数据库发生连接断开事件，如果用户在超时之前还没有回滚事务，则 proxy 断开与用户的连接。
>
超时参数在 proxy 配置文件中为：
```json
<server_close timeout="60"/>
```


#### 全局唯一字段

关键字`auto_increment`，即支持一个全局的自增字段，auto_increment 可以保证该表某个字段全局唯一，但不保证单调递增，具体使用方法如下：

##### 创建
```
mysql> create table auto_inc (a int,b int,c int auto_increment,d int,key auto(c),primary key p(a,d)) shardkey=d;
Query OK, 0 rows affected (0.12 sec)
```

##### 插入
```
mysql>  insert into shard.auto_inc ( a,b,d,c) values(1,2,3,0),(1,2,4,0);
Query OK, 2 rows affected (0.05 sec)
Records: 2  Duplicates: 0  Warnings: 0
	
mysql> select * from shard.auto_inc;
+---+------+---+---+
| a | b    | c | d |
+---+------+---+---+
| 1 |    2 | 2 | 4 |
| 1 |    2 | 1 | 3 |
+---+------+---+---+
2 rows in set (0.03 sec)
```

如果发生切换、重启等过程，自增长字段中间会有空洞，例如：
```
mysql> insert into shard.auto_inc ( a,b,d,c) values(11,12,13,0),(21,22,23,0);
Query OK, 2 rows affected (0.03 sec)
mysql> select * from shard.auto_inc;
+‐‐‐‐‐‐+‐‐‐‐‐‐+‐‐‐‐‐‐+‐‐‐‐‐‐+
| a | b | c | d |
+‐‐‐‐‐‐+‐‐‐‐‐‐+‐‐‐‐‐‐+‐‐‐‐‐‐+
| 21 | 22 | 2002 | 23 |
| 1 | 2 | 2 | 4 |
| 1 | 2 | 1 | 3 |
| 11 | 12 | 2001 | 13 |
+‐‐‐‐‐‐+‐‐‐‐‐‐+‐‐‐‐‐‐+‐‐‐‐‐‐+
4 rows in set (0.01 sec)
```

更改当前值：
```
alter table auto_inc auto_increment=100
```

通过`select last_insert_id()`获取最近一个自增值：
```	
mysql> insert into auto_inc ( a,b,d,c) values(1,2,3,0),(1,2,4,0);
Query OK, 2 rows affected (0.73 sec)
		
mysql> select * from auto_inc;
+---+------+------+---+
| a | b    | c    | d |
+---+------+------+---+
| 1 |    2 | 4001 | 3 |
| 1 |    2 | 4002 | 4 |
+---+------+------+---+
2 rows in set (0.00 sec)
	
mysql> select last_insert_id();
+------------------+
| last_insert_id() |
+------------------+
| 4001             |
+------------------+
1 row in set (0.00 sec)
```

目前 select last_insert_id() 只能跟 shard 表和广播表的自增字段一起使用，不支持 noshard 表。



#### 数据导出导入


##### 导出数据
TDSQL MySQL版 支持通过 mysqldump 导出数据，导出前须设置 net_write_timeout 参数：`set global net_write_timeout=28800`，命令行有权限限制，请通过 [TDSQL MySQL版 控制台](https://console.cloud.tencent.com/dcdb) 操作。
```
mysqldump --compact --single-transaction --no-create-info -c db_name table_name  -utest -h10.xx.xx.34 -P3336  -ptest123
```

>?
>- db 和 table 名参数根据实际情况选择，如果导出的数据要导入到另外一套 TDSQL MySQL版 环境的话，必须加上 -c 选项。
>- 导出帐号需拥有 `select on *.*` 的权限。

##### 导入数据
TDSQL MySQL版 提供专门的导入数据工具，完成 load data outfile 对应数据的导入，该工具的原理是把源文件按照 shardkey 的路由规则，切分成多个文件，然后把每个单独透传到对应的后端数据库。

[下载工具](https://tdsqlfilebackup-1252014656.cos.ap-chengdu.myqcloud.com/load_data.tar)

```
[tdengine@TENCENT64 ~/]$./load_data

format:./load_data mode0/mode1 proxy_host proxy_port user password db_table shardkey_index file field_terminate filed_enclosed

example:./load_data mode1 10.xx.xx.10 3336 test test123 shard.table  1 '/tmp/datafile'  ' ' ''
```

>!
>- 源文件必须以 '\n' 作为换行符。
>- mode0 只切分源文件，不做数据导入，一般用于调试，正式导入数据使用 mode1。
>- shardkey_index 从0开始，如果 shardkey 在第2个字段，则 shardkey_index 为1。



#### 数据库管理语句

##### 状态查询
通过 SQL 可以查看 proxy 的配置以及状态信息，目前支持如下命令：
- `/*proxy*/help;`
- `/*proxy*/show config;`
- `/*proxy*/show status;`

>?如果使用 MySQL 客户端，需要在使用客户端时增加`-c`选项，如 mysql -hxxx.xxx.xxx.xxx -Pxxxx -uxxx -pxxx -c。

示例如下：
```
	mysql> /*proxy*/help;
	+-----------------------+-------------------------------------------------------+
	| command               | description                                           |
	+-----------------------+-------------------------------------------------------+
	| show config           | show config from conf                                 |
	| show status           | show proxy status,like route,shardkey and so on       |
	| set sys_log_level=N   | change the sys debug level N should be 0,1,2,3        |
	| set inter_log_level=N | change the interface debug level N should be 0,1      |
	| set inter_time_open=N | change the interface time debug level N should be 0,1 |
	| set sql_log_level=N   | change the sql debug level N should be 0,1            |
	| set slow_log_level=N  | change the slow debug level N should be 0,1           |
	| set slow_log_ms=N     | change the slow ms                                    |
	| set log_clean_time=N  | change the log clean days                             |
	| set log_clean_size=N  | change the log clean size in GB                       |
	+-----------------------+-------------------------------------------------------+
	10 rows in set (0.00 sec)
```

```
	mysql> /*proxy*/show config;
	+-----------------+--------------------+
	| config_name     | value              |
	+-----------------+--------------------+
	| version         | V2R120D001         |
	| mode            | group shard        |
	| rootdir         | /shard_922         |
	| sys_log_level   | 0                  |
	| inter_log_level | 0                  |
	| inter_time_open | 0                  |
	| sql_log_level   | 0                  |
	| slow_log_level  | 0                  |
	| slow_log_ms     | 1000               |
	| log_clean_time  | 1                  |
	| log_clean_size  | 1                  |
	| rw_split        | 1                  |
	| ip_pass_through | 0                  |
	+-----------------+--------------------+
	14 rows in set (0.00 sec)
```

```
	mysql> /*proxy*/show status;
	+-----------------------------+------------------------------------------------------------------------------+
	| status_name                 | value                                                                        |
	+-----------------------------+------------------------------------------------------------------------------+
	| cluster                     | group_1499858910_79548                                                       |
	| set_1499859173_1:ip         | 10.49.118.165:5025;10.175.98.109:5025@1@IDC_4@0,10.231.23.241:5025@1@IDC_2@0 |
	| set_1499859173_1:hash_range | 0---31                                                                       |
	| set_1499911640_3:ip         | 10.49.118.165:5026;10.175.98.109:5026@1@IDC_4@0,10.231.23.241:5026@1@IDC_2@0 |
	| set_1499911640_3:hash_range | 32---63                                                                      |
	| set                         | set_1499859173_1,set_1499911640_3                                            |

```

同时 proxy 增强了 explain 的返回结果，显示 proxy 修改后的 SQL。
```
	mysql> explain select * from test1;
	+------+-------------+-------+------+---------------+------+---------+------+------+-------+-----------------------------------------+
	| id   | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra | info                                    |
	+------+-------------+-------+------+---------------+------+---------+------+------+-------+-----------------------------------------+
	|    1 | SIMPLE      | test1 | ALL  | NULL          | NULL | NULL    | NULL |   16 |       | set_2,explain select * from shard.test1 |
	|    1 | SIMPLE      | test1 | ALL  | NULL          | NULL | NULL    | NULL |   16 |       | set_1,explain select * from shard.test1 |
	+------+-------------+-------+------+---------------+------+---------+------+------+-------+-----------------------------------------+
	2 rows in set (0.03 sec)
```


#### 透传 SQL


TDSQL MySQL版 实例会对 SQL 进行语法解析，有一定的限制，如果用户想在某个节点（set）中执行 MySQL 支持，但分布式实例不支持的 SQL 时，可以使用透传 SQL 的功能。
>?
>- 透传 SQL 时，proxy 不会解析 SQL，如果是往两个 set 进行透传写操作，不会使用分布式事务，特殊情况下会发生不一致问题，因此对于写操作建议一次透传一个 set。
>- 为保证透传语法生效，连接 MySQL 时请使用 -c 参数。


```
MySQL [test]> repair table test.t1;
ERROR 664 (HY000): Proxy ERROR:SQL is too complex, only applicable to noshard table: Shard table do not support repair
MySQL [test]> /*sets:allsets*/repair table test.t1;
+---------+--------+----------+----------+------------------+
| Table   | Op     | Msg_type | Msg_text | info             |
+---------+--------+----------+----------+------------------+
| test.t1 | repair | status   | OK       | set_1544429866_3 |
| test.t1 | repair | status   | OK       | set_1544429718_1 |
+---------+--------+----------+----------+------------------+
2 rows in set (0.01 sec)
```

具体语法：
- sets:set_1,set_2：代表指定某几个 set，set 名字可以通过`/*proxy*/show status`查询。
- sets:allsets：代表指定全部 set。
- shardkey:10：代表支持透传 SQL 到 shardkey 对应值上的 set。
- shardkey_hash:10：透传到负责 hash 值为10的 set，如果为0，则发送到第一个 set 上。



#### 预处理

SQL 类型的支持：
- PREPARE Syntax
- EXECUTE Syntax

二进制协议的支持：
- COM_STMT_PREPARE
- COM_STMT_EXECUTE

示例：
```
	mysql> select * from test1;
	+---+------+
	| a | b    |
	+---+------+
	| 5 |    6 |
	| 3 |    4 |
	| 1 |    2 |
	+---+------+
	3 rows in set (0.03 sec)
```

```
	mysql> prepare ff from "select * from test1 where a=?";
	Query OK, 0 rows affected (0.00 sec)
	Statement prepared
	
	mysql> set @aa=3;
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> execute ff using @aa;
	+---+------+
	| a | b    |
	+---+------+
	| 3 |    4 |
	+---+------+
	1 row in set (0.06 sec)
```

#### 二级分区

TDSQL MySQL版 目前支持 Range 和 List 两种格式的二级分区，具体建表语法和 MySQL 分区语法类似。

##### 二级分区语法
一级 Hash，二级 List 分区示例如下：
```
MySQL [test]> CREATE TABLE customers_1 (
  first_name VARCHAR(25) key,
  last_name VARCHAR(25),
  street_1 VARCHAR(30),
  street_2 VARCHAR(30),
  city VARCHAR(15),
  renewal DATE
) shardkey=first_name

PARTITION BY LIST (city) (
  PARTITION pRegion_1 VALUES IN('Beijing', 'Tianjin', 'Shanghai'),
  PARTITION pRegion_2 VALUES IN('Chongqing', 'Wulumuqi', 'Dalian'),
  PARTITION pRegion_3 VALUES IN('Suzhou', 'Hangzhou', 'Xiamen'),
  PARTITION pRegion_4 VALUES IN('Shenzhen', 'Guangzhou', 'Chengdu')
);
```

一级 Range，二级 List 创建语法如下：
```
MySQL [test]> CREATE TABLE tb_sub_r_l (
   id int(11) NOT NULL,
   order_id bigint NOT NULL,
   PRIMARY KEY (id,order_id)) 
   PARTITION BY list(order_id)
   (PARTITION p0 VALUES in (2121122),
   PARTITION p1 VALUES in (38937383))
   TDSQL_DISTRIBUTED BY RANGE(id) (s1 values less than (100),s2 values less than (1000));
Query OK, 0 rows affected, 1 warning (0.35 sec)
```

####### Range 支持类型
- DATE，DATETIME，TIMESTAMP。
- 支持 year、month、day 函数，函数为空和 day 函数一样。
- TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT。
- 支持 year、month、day 函数，此时传入的值转换为年月日，然后和分表信息进行对比。

####### List 支持类型
- DATE、DATETIME、TIMESTAMP。
- 支持年月日函数。
- TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT、VARCHAR。

<dx-alert infotype="alarm" title="警告">
<li>建议不要使用 TIMESTAMP 类型作为分区键，因为 TIMESTAMP 受到时区的影响，同时只能使用到2038年。</li>
<li>如果分区键是 char 或者 varchar 类型，建议长度不超255。</li>
</dx-alert>

##### 使用场景和方法建议
建议业务尽量都使用一级分区表。
- 使用前根据业务长期场景合理设计表结构，二级分区适用于表结构创建后长期都不需要 DDL 变更、需要定期进行分区数据清理和裁剪的场景，如日志流水表。
- 合理设计二级分区的粒度，二级分区的粒度建议不要划分得太细，避免产生过多的二级子表。如流水表按月进行二级分区，而不是按天/小时进行分区，避免文件系统上数据文件个数过多。
- 在对二级分区表进行 SQL 查询时，查询条件需要尽量带上一级分区和二级分区的键值，避免执行查询时需要打开很多的数据文件进行搜索。
- 在对二级分区表进行 join 查询时，如果查询条件未能带上一级分区和二级分区的键值，操作性能效率较低，建议不要使用。
- 表的主键或唯一索引需要包含分区键，否则无法保证数据唯一性。


#### 错误码和错误信息


proxy 增加如下错误编码：

	#define ER_PROXY_GRAM_ERROR_BEGIN 600
	
	#define ER_PROXY_SANITY_ERROR 601           // "Sanity error: %s"
	#define ER_PROXY_GS_NOT_SUPPORT 602         // sql 类型不支持
	#define ER_PROXY_ORDERBY_INDEX_NEG 603      // order by index is negative
	#define ER_PROXY_ORDERBY_INDEX_TOO_BIG 604  // order by index is too big
	#define ER_PROXY_ORDERBY_TYPE_UNSUPPORT 605 // 不支持到 order by 用法
	#define ER_PROXY_GROUPBY_INDEX_NEG 606      // group by index is negative
	#define ER_PROXY_GROUPBY_INDEX_TOO_BIG 607  // group by index is too big
	#define ER_PROXY_GROUPBY_TYPE_UNSUPPORT 608 // 不支持的 group by 用法
	#define ER_PROXY_GET_AUTO_ID_FAILED 609     // 获取自增 id 失败
	#define ER_PROXY_TEANS_ROLLED_BACK 610      // 事务已经被回滚
	#define ER_PROXY_ONE_SET 611                // 当前 sql 应该被发往一个后端，但是不是
	#define ER_PROXY_CLIENT_HS_ERROR 612        // 解析客户端握手包出错
	#define ER_PROXY_ACCESS_DENIED_ERROR 613    // the length of readu_auth_switch_result is not 20，不应该出现
	#define ER_PROXY_TRANS_NOT_ALLOWED 614      // 事务中不允许执行的命令
	#define ER_PROXY_TRANS_READ_ONLY 615        // 只读事务中不允许执行的命令
	#define ER_PROXY_TRANS_ERROR_DIFFENT_SET 616    // 非 xa 事务中，只读 sql 使用了多个后端
	#define ER_PROXY_STRICT_ERROR 617           // strict 模式下，一次仅允许修改一个 set
	#define ER_PROXY_SC_TOO_LONG 618            // 后端断开时间过长，链接断开
	#define ER_PROXY_START_TRANS_FAILED 619     // 开启新的 xa 事务失败
	#define ER_PROXY_SC_RETRY 620               // server 已经 close，请重试上一条 sql
	#define ER_PROXY_SC_TRANS_IN_ROLLBACK_ONLY 621  // server 已经 close，当前事务处于 rollback
	#define ER_PROXY_SC_COMMIT_LATER 622        // server 已经 close，事务会在稍后提交
	#define ER_PROXY_SC_ROLLBACL_LATER 623      // server 已经 close，事务会在稍后回滚
	#define ER_PROXY_SC_IN_COMMIT_OR_ROLLBACK 624   //  server 在事务提交/回滚阶段 close
	#define ER_PROXY_SC_NEED_ROLLBACK 625       // server 已经 close，需要回滚当前事务
	#define ER_PROXY_SC_STATE_WILL_ROLLBACK 626 // server 已经 close，将会会滚
	#define ER_PROXY_XA_UNSUPPORT 627           // xa 目前不支持的命令
	#define ER_PROXY_XA_INVALID_COMMAND 628     // xa 命令不合法
	#define ER_PROXY_XA_GTID_INIT_ERROR 629     // gtid log 初始化失败
	#define ER_PROXY_XA_GET_SET_IP_PORT_FAILED 630  // 获取 set 地址失败
	#define ER_PROXY_XA_UPDATE_GTID_LOG_FAILED 631  // 更新 gtid log 失败
	#define ER_PROXY_MYSQL_PARSER_ERROR 632     // 嵌入式库 sql 解析失败
	#define ER_PROXY_ILLEGAL_ID 633             // kill id 不合法
	#define ER_PROXY_NOT_SUPPORT_CURSOR 634     // CURSOR_TYPE_READ_ONLY 暂不支持
	#define ER_PROXY_UNKNOWN_PREPARE_HANDLER 635    // 执行的 prepare 不明确
	#define ER_PROXY_SET_PARA_FAIL 636          // Set parameters failed
	#define ER_PROXY_SUBPARTITION_DEAY 637      // 处理二级分区发生错误
	#define ER_PROXY_NO_SUBPARTITION_ROUTE 638  // 没有获取到二级分区表的路由信息
	#define ER_PROXY_LOCK_MORE_TABLE 639        // 一次只可以锁定一张二级分区表
	#define ER_PROXY_GET_ROUTER_LOCK_FAIL 640   // 获取路由锁失败
	#define ER_PROXY_PART_NAME_EMPTY 641        // 分区名称为空
	#define ER_PROXY_SUB_PART_TABLE_IS_NONE 642 // 没有二级分区
	#define ER_PROXY_PART_TYPE_DENY 643         // 二级分区类型不支持
	#define ER_PROXY_PART_NAME_ILLEGAL 644      // 分区名不合法
	#define ER_PROXY_DROP_ALL_PARTITION_FAIL 645    //  删除所有分区失败，尝试直接删除表
	#define ER_PROXY_GET_OLD_PART_NUM_FAIL 646  // 获取表的分片数失败
	#define ER_PROXY_EMPTY_SQL 647              // empty sql，不会返回给客户端
	#define ER_PROXY_ERROR_SHARDKEY 648         // sk 必须为某一列
	#define ER_PROXY_ERROR_SUB_SHARDKEY 649     // 二级分区键失败
	#define ER_PROXY_SQLUSE_NOT_SUPPORT 650     // proxy 不支持这种用法
	#define ER_PROXY_DBFW_WHITE_LIST_DENY 651   // 不在白名单，被防火墙拒绝
	#define ER_PROXY_DBFW_DENY 652              // 防火墙拒绝
	#define ER_PROXY_INCORRECT_ARGS 653         // stmt 参数不正确
	#define ER_PROXY_SYSTABLE_UNSUPPORT_NON_READ_SQL 654    // 不支持非只读 sql 访问系统表
	#define ER_PROXY_TABLE_NOT_EXIST 655        // 表不存在
	#define ER_PROXY_SHARD_JOIN_UNSUPPORT_TYPE 656  // shard join 暂不支持的用法
	#define ER_PROXY_RECURSIVE_JOIN_DENY 657    // 递归 join 不支持
	#define ER_PROXY_JOIN_INTERNAL_ERROR 658    // join 异常
	#define ER_PROXY_SQL_TOO_COMPLEX 659        // sql 太复杂，groupshard 暂不支持
	#define ER_PROXY_INVALID_ARG_FOR_GTID_STATE 660 // gtid_state() 参数不合法
	#define ER_PROXY_CANT_SET_GLOBAL_AUTOCOMMIT_GS 661  // Global autocommit cannot be set in groupshard
	#define ER_PROXY_INVALID_VALUE_FOR_AUTOCOMMIT 662   // autocommit 值设置不合法
	#define ER_PROXY_XID_ERROR 663              // xid 不合法
	#define ER_PROXY_XID_GENERAT_FAILED 664     // xid 不能由用户指定
	#define ER_PROXY_CANT_EXEC_IN_INTER_TRANS 665   // "The command cannot be executed in internal transction"
	#define ER_PROXY_XID_TIME_ERROR 666         // "Unexpected time part of xid"
	#define ER_PROXY_XID_TIMEDIFF_TOO_LONG 667  // "timediff > 1800s, it's not safe to execute boost"
	#define ER_PROXY_SAVEPOINT_NOT_EXIST 668    // SAVEPOINT 不存在
	#define ER_PROXY_SC_TRANS_IN_ROLLED 669     // 事务已经会滚，由于 serevr 已经 close
	#define ER_PROXY_CANT_BOOST_IN_TRANS 670    // 事务中不允许执行 SQLCOM_BOOST
	#define ER_PROXY_TRANS_EXPECTED 671         // "A transaction is expected, this maybe a bug"
	#define ER_PROXY_EXTERNAL_TRANS 672         // 外部 xa 中不允许执行
	#define ER_PROXY_AUTO_INC_FAIL 673          // "Deal auto inc failed"
	#define ER_PROXY_CHECK_JOIN_FAIL 674        // "Check join failed"
	#define ER_PROXY_TABLE_TYPE_NOT_MATCH 675   // "Do not support shard-table operations in noshard instance"
	#define ER_PROXY_UNSUPPORT_NS_IN_INSERT 676 // "Do not support noshard and noshard_allset in insert sql"
	#define ER_PROXY_ALTER_SEQ_ID_FAIL 677      // Alter seq id failed
	#define ER_PROXY_ALTER_ID_ILLEGAL 678       // Alter seq id is illegal
	#define ER_PROXY_CANT_CHANGE_STEP 679       // "Current table use zk to get auto inc, do not support to change step: \'%s\'"
	#define ER_PROXY_ALTER_STEP_FAIL 680        // Alter step failed
	#define ER_PROXY_TOO_MUCH_TABLES 681        // 表的数量超过限制
	#define ER_PROXY_TABLE_EXISTED 682          // 表已经存在
	#define ER_PROXY_CREATE_STABLE_FAILED 683   // 创建shard表失败，复杂 sql 不能用来创建 shard 表
	#define ER_PROXY_DDL_DENY 684               // ddl 不允许的操作
	#define ER_PROXY_SHADKEY_ERROR 685          // SQL should not relate to subpartition tables
	#define ER_PROXY_NO_SK 686                  // reject nosk
	#define ER_PROXY_COMBINE_SQL_KEY 687        // Something went wrong
	#define ER_PROXY_GET_SK_ERROR 688           // sk 获取失败
	#define ER_PROXY_SHOW_FAILED 689            // proxy show 命令错误
	#define ER_PROXY_SET_FAILED 690             // proxy set 命令错误
	#define ER_PROXY_UNLOCK_FORMAT_ERROR 691    // sql 格式不正确
	#define ER_PROXY_UNLOCK_ROUTER_FAIL 692     // 释放路由锁失败
	#define ER_PROXY_LOCK_ROUTER_FAIL 693       // 加路由锁失败
	#define ER_PROXY_PROXY_CMD_FAIL 694         // 不支持的/*proxy*/ 命令
	#define ER_PROXY_PROCESS_RULE_FILE_FAILED 695   // dump_error
	#define ER_PROXY_GET_AUTO_NUM_ERROR 696     // 获取自增值失败
	#define ER_PROXY_SEQUENCE_NOT_EXIST 697     // sequence 不存在
	#define ER_PROXY_SEQUENCE_ERROR 698         // sequence 不合法
	#define ER_PROXY_SEQUENCE_ALREADY_EXIST 699 // sequence 已经存在
	#define ER_PROXY_SQL_RETRY 700              // sql 还未提交或回滚
	#define ER_PROXY_XA2PC_ABORT 701            // 2pc 失败，事务将会会滚
	#define ER_PROXY_XA2PC_COMMIT 702           // 2pc 失败，后续提交
	#define ER_PROXY_XA2PC_UNCERTAIN 703        // 2pc 失败，结果未知
	#define ER_PROXY_KILL_ERROR 704             // kill 失败
	#define ER_PROXY_TRACE_DENY	705		  // trace 模式下不允许执行的sql
	#define ER_PROXY_SQL_IMCOMPLETE 706		 // 事务状态不完整
	#define ER_PROXY_SHARDKEY_HASH_ERROR 709	// sk hash错误
	
	#define ER_PROXY_GRAM_ERROR_END 799         
	
	// system error -----------------------------------------------------------------------
	#define ER_PROXY_SYSTEM_ERROR_BEGIN 900
	
	#define ER_PROXY_SLICING 901                // slice 被修改，可能在扩容阶段，拒掉当前 sql
	#define ER_PROXY_NO_DEFAULT_SET 902         // set 为空
	#define ER_PROXY_GET_ADDRESS_FAILED 903     // 还未初始化完成，获取后端地址失败，稍后重试
	#define ER_PROXY_SQL_SIZE_ERROR_IN_GET_CANDIDATE_ADDRESS 904 // 获取后端地址出错（发往后端个数不正确）
	#define ER_PROXY_GET_ADDRESS_ERROR 905      // 获取后端地址出错
	#define ER_PROXY_CANDIDATE_ADDRESS_EMPTY 906 // 未获取到后端地址
	#define ER_PROXY_CANT_GET_SOCK 907          // socket 获取失败	
	#define ER_PROXY_GET_SET_SOCK_FAIL 908      // socket 获取失败
	#define ER_PROXY_CONNECT_ERROR 909          // 后端连接失败
	#define ER_PROXY_NO_SQL_ASSIGN_TO_SET 910   // 分发 sql 异常
	#define ER_PROXY_STATUS_ERROR 911           // group 状态异常，断开链接
	#define ER_PROXY_CONN_BROKEN_ERROR 912      // server close，sql 状态不正常
	#define ER_PROXY_UNKNOWN_ERROR 913          // proxy 未知错误
	#define ER_PROXY_ALL_SLAVES_UNAVAILABLE 914 // 所有备机不可用
	#define ER_PROXY_ALL_SLAVES_CHANGE 915      // 备机异常
	
	#define ER_PROXY_ERROR_END 916


>?其中900以上为系统错误，会通过 [云监控平台](https://console.cloud.tencent.com/monitor/overview) 进行告警。

### 2、TDStore 引擎

#### 使用说明

TDSQL MySQL版（TDStore 引擎）在使用上与 MySQL 8.0 高度一致，用户可以将其视为一个 MySQL 8.0 实例来使用，但请注意以下几点：

##### TDStore 引擎暂不支持的语法特性
- 不支持下列对象的 CREATE/ALTER/DROP 语法。
	- event
	- resource group
	- instance
	- server
	- tablespace
	- spatial
- SET GLOBAL log_output=TABLE：不支持 general_log 和 slow_log 输出到表，只能输出到 file（也就是常说的慢查询日志文件）。
- 暂不支持 LOCK TABLE 语法（默认依赖 LOCK TABLE 语法的数据导入导出工具建议通过参数设置跳过 LOCK TABLE 的执行避免报错，例如 mysqldump --lock-tables=false）。
- 禁止对 MySQL 库下的任何系统表进行 DDL 操作。
- 暂不支持外键。
- 暂不支持 binlog。
- 不允许修改表的存储引擎。
- 暂不支持虚拟列、GEOMETRY 类型、降序索引、全文索引。
- **实验特性：如下功能在当前版本为实验特性，不建议用户在正式生产环境中使用。**
	- view：视图
	- trigger：触发器
	- procedure：存储过程
	- function：函数
>?每个实验特性各自拥有独立开关和默认值。用户如需开启/关闭，请在连接到 TDStore 实例后通过如下 SQL 语句开启（以 tdsql_enable_trigger 为例）：`SET PERSIST tdsql_enable_trigger=ON;`。
>
<table>
<thead><tr><th>特性名</th><th>开关</th><th>默认值</th></tr></thead>
<tbody><tr>
<td>视图</td>
<td>tdsql_enable_view</td>
<td>ON</td></tr>
<tr>
<td>触发器</td>
<td>tdsql_enable_trigger</td>
<td>OFF</td></tr>
<tr>
<td>存储过程</td>
<td>tdsql_enable_procedure</td>
<td>ON</td></tr>
<tr>
<td>函数</td>
<td>tdsql_enable_function</td>
<td>ON</td></tr>
</tbody></table>

##### TDStore 引擎与 MySQL 8.0 的使用差异
- TDStore 自增字段暂时只保证全局唯一，不保证全局自增。
- alter table 除了 add column，add/drop index 支持 online 不阻塞读写以外，其他 ddl 均会在执行 ddl 期间锁表。
- set global xxx 只会设置某个计算节点上的变量，需要用广播 hint /\*##all_nodes\*/set global xxx 广播到所有计算节点才能使其在全局生效。
- show variables，show global status 展示的是当前连接的计算节点中的状态信息，连到不同的计算节点展示的信息可能不同。
- show processlist 展示的是当前连接的计算节点的 processlist，需要用 show full processlist 展示所有计算节点的 processlist。


#### 错误码信息


TDStore 引擎错误码及说明如下：

>?当您遇到如下错误码需要处理时，您可以通过 [在线支持](https://cloud.tencent.com/act/event/Online_service) 进行处理。

| **错误码** | **说明**                                                     |
| ---------- | ------------------------------------------------------------ |
| 50018      | 通用错误码，无具体含义                                       |
| 50019      | 存储引擎设置错误，例如 SET default_storage_engine=InnoDB 报此错误 |
| 50020      | 发生在 SQLEngine 进程启动的阶段，SQLEngine 向 MC 注册失败         |
| 50021      | 执行 DML 时，并发的 DDL 修改了表结构，DML 报此错误                |
| 50022      | 执行 DDL 时，DDL 被后台线程中断，DDL 报此错误                    |
| 50023      | 执行 DML 时，并发的 DDL 修改了表结构，DML 报此错误                |
| 50024      | 执行 DDL 时，校验表结构失败，DDL 报此错误                       |
| 50025      | 同一时间、同一对象上只能有一个 DDL 正在执行，并发的 DDL 报此错误，例如，sqlengine#1 执行 ALTER TABLE db1.tbl1；sqlengine#2 也执行 ALTER TABLE db1.tbl1，sqlengine#2 报错 |
| 50028      | 执行 DDL 时，创建表定义失败                                    |
| 50030      | 执行 DDL（带有自增值属性的 DDL）时，初始化自增值失败           |
| 50031      | 执行 DDL（如建表、建索引）时，分配 index id 失败                |
| 50032      | SQL Engine 通用错误码，无具体含义                             |
| 50033      | 执行 DML，获取自增值 next val 失败                              |
| 50034      | 执行 DDL 或 DML 时，TDStore 无法创建迭代器                        |
| 50035      | 执行 DML，推进自增值 next val 失败                              |
| 50036      | 执行 DDL 时，未定义自增列，但给出了自增值，如 CREATE TABLE t(a INT) AUTO_INCREMENT=10; |
| 50037      | 执行 DDL 或 DML 时，TDStore 上不存在相应事务上下文                |
| 50038      | 执行 DDL 或 DML 时，TDStore 上不存在相应事务上下文，更有可能发生了 "TDStore 切主" |
| 50039      | 发生在进程启动时，初始化路由失败                             |
| 50040      | 用户在 SET VARIABLE 时缺少权限                                 |
| 50041      | 内置函数 STORAGE_FORMAT 只接受1个 uint32 整数作为入参，输入其他参数会报此错误。如 SELECT STORAGE_FORMAT(10001); |
| 50042      | 执行 DML 提交事务时，乐观事务冲突报错                          |
| 50043      | 执行 DML 时，如果有写操作失败，为避免数据不一致，必须手工把事务回滚，然后能执行其他 SQL；在 ROLLBACK 之前执行其他 SQL，报此错误，如 INSERT INTO t - 失败、COMMIT - 报此错误、ROLLBACK - OK、其他 SQL - OK |
| 50044      | 发生在 SQLEngine 进程启动时，集群尚不可用                      |
| 50045      | 发生在 SQLEngine 进程启动时，无法建立事务保活机制，会影响到正常事务的执行流程，拒绝启动 |
| 50046      | 发生在 DDL，ALTER TABLE t CHANGE ENGINE 在 RocksDB 和其他引擎之间切换会报此错误 |
| 50047      | 发生在用 DML 直接修改权限相关的表（user/global_grants 等）报错  |
| 50048      | 执行 DML 时，并发的 DDL 修改了表结构，DML 报此错误                |
| 50049/50050     | 发生在 SQLEngine 进程启动时，SQLEngine 向 MC 申请唯一的标识 ID 并持久化在 SQLEngine 的数据目录下，这期间报错误 |
| 50051      | 执行 DDL 时，Online Copy DDL 流程的错误码                       |
| 50052      | 执行 DDL 时，Online DDL 流程的错误码                            |
| 50053      | 发生在本地变量版本比 MC 维护的版本低的情况                     |
| 50054      | 发生在对只读变量使用 SET PERSIST_ONLY 语法                     |
| 50055      | 发生在 TDStore 进行全量备份时报错                              |
| 50057      | DDL 会持久化一些任务信息，任务信息有长度上限，超过上限会报此错误 |

