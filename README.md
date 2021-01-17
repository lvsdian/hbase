[toc]

### 简介

#### 定义

- HBase是一种分布式、可扩展、支持海量数据存储的NoSQL数据库

#### 数据模型

- 逻辑上，HBase的数据模型同关系型数据库很类似。数据存储在一张表中，有行有列。但从HBase的底层物理存储结构（K-V）来看，HBase是一个**multi-dimensional map**

##### 逻辑结构

​	![](img\1.png)

- 一张表分成多个文件夹，一个列族对应一个文件夹，一个或多个列组成一个列族

  如上，personal_info是一个列族，office_info是一个列族，不同的列族之间，是分开存储的

- Row key按字典序排序

- Regin：将表横向切分，如上，有三个region
- store：真正存数据的地方（不包含元数据），如上，有6个store

##### 物理存储结构

​	![](C:\Users\LSD\Desktop\hbase\img\2.png)

##### 数据模型

- Name Space

  命名空间，类似于关系型数据库的database概念，每个命名空间下有多个表，Hbase有两个自带的命名空间，分别是hbase和dafault.hbase，hbase中存放Hbase内置的表，default表是用户默认使用的命名空间

- Region

  类似于关系型数据库的表概念，不同的是，HBase定义表示只需要声明列族即可，不需要声明具体的列，这意味着，往HBase中写入数据时，字段可以动态、按需指定，因此，和关系型数据相比，HBase能够轻松应对字段变更的场景

- Row

  HBase表中的每行数据都由一个RowKey和多个Column（列）组成，数据是按照RowKey的字典顺序存储，并且查询数据时只能根据RowKey进行检索，所以RowKey的设计十分重要。

- Column

  HBase中的每个列都由Column Family（列族）和Column Qulifier（列限定符）进行限定，例如info:name，info:age。建表时，只需指明列族，而列限定符无需预先定义。

- Time Stamp

  用于标识数据的不同版本(version)，每条数据写入时，如果不指定时间戳，系统会自动为其加上该字段，其值为写入HBase的时间。

- Cell

  由（rowkey、column Family、column Qualifier、timestamp）唯一确定的单元。cell中的数据是没有类型的，全部是字节码形式存储。

#### 基本架构

​	![](C:\Users\LSD\Desktop\hbase\img\3.png)

- Region Server

  Region Server作为Region的管理者，其实现类为HRegionServer，主要作用如下：

  - 对于数据的操作：get、put、delete（DML操作）

  - 对于Region的操作：splitRegion、compactRegion

- Master

  Master是所有Region Server的管理者，其实现类为HMaster，其主要作用如下：

  - 对于表的操作：create、delete、alter（DDL操作）

  - 对于RegionServer的操作：分配regions到每个RegionServer，监控每个RegionServer的状态，负载均衡和故转移

- Zookeeper

  HBase通过Zookeeper来做master的高可用、RegionServer的监控、元数据的入口以及集群配置的维护等工作

- HDFS

  HDFS为HBase提供最终的底层数据存储服务，同时为HBase提供高可用支持

### 安装/操作

#### 安装

#### Shell操作

- 进入HBase客户端命令行：`[atguigu@hadoop102 Hbase]$ bin/hbase shell`

- 查看帮助命令：`Hbase(main):001:0> help`
- 查看当前数据库中有哪些用户表：`Hbase(main):002:0> list`

##### DDL操作

- `create 'student', 'info'`：创建一个`student`表，有一个列族`info`

  ![](C:\Users\LSD\Desktop\hbase\img\4.png)

- `create 'stu', 'info1', 'info2'`：创建stu表，有两个列族

  ![](C:\Users\LSD\Desktop\hbase\img\5.png)

- `describe 'student'`：查看表信息

  ![](C:\Users\LSD\Desktop\hbase\img\6.png)

- `alter 'student', {NAME=> 'info', VERSIONS=>3}`：hbase默认存储1个版本的数据，如此修改后，会保存3个版本的数据

  ![](C:\Users\LSD\Desktop\hbase\img\7.png)

- `drop 'student'`：删除表

  ![](C:\Users\LSD\Desktop\hbase\img\8.png)

- `create_namespace 'bigdata'`：创建命名空间（相当于mysql建库）

  ![](C:\Users\LSD\Desktop\hbase\img\9.png)

- `create "bigdata:stu", "info"`：在`bigdata`这个命名空间下创建表`stu`，如果想要删除`bigdata`这个命名空间，则需要先`disable "bigdata:stu"`，然后`drop "bigdata:stu"`，再`drop "bigdata"`

  ![](C:\Users\LSD\Desktop\hbase\img\10.png)



##### DML操作

- put 

![](C:\Users\LSD\Desktop\hbase\img\18.png)

​	![](C:\Users\LSD\Desktop\hbase\img\19.png)

- get

​	![](C:\Users\LSD\Desktop\hbase\img\11.png)

- scan

![](C:\Users\LSD\Desktop\hbase\img\12.png)

- delete

![](C:\Users\LSD\Desktop\hbase\img\13.png)

![](C:\Users\LSD\Desktop\hbase\img\14.png)

![](C:\Users\LSD\Desktop\hbase\img\15.png)

- deleteall：删除rowkey

![](C:\Users\LSD\Desktop\hbase\img\16.png)

- truncate：清空表![](C:\Users\LSD\Desktop\hbase\img\17.png)

### 进阶

#### 详细架构

![](C:\Users\LSD\Desktop\hbase\img\20.png)

- HBase底层是依赖于HDFS的，所以会有HDFS的DataNode
- HBse依赖于Zookeeper，客户端无论找HMaster还是RegionServer，都先找ZK
- HMaster/Master与ZK交互，对于客户端来说，表数据的操作由ZK分担给HMaster
- 除了HMaster之外，还有HRegionServer，维护region，即一张一张的表
- HMaster中还有HLog，功能是WAL(write ahead log)，记录操作，防止内存中数据丢失
- 一个HRegionServer维护多个region（表），一张表(region)有多个列族，列族是逻辑上的概念，物理结构中叫Store，Store与Store之间存储是分离的，分文件夹的
- Store通过Mem Store，即内存，将数据刷到磁盘上，即HFile
- HFile、HLog都要通过HDFS客户端存到HDFS上
- HBase客户端（Client）先连ZK，再连HRegionServer操作数据

#### 写流程

​		![](C:\Users\LSD\Desktop\hbase\img\21.png)

1. 客户端连接ZK，查询meta表（即记录region的位置信息的表）所在的RegionServer是哪一个
2. 找到meta表所在的机器后（假设meta表在hadoop102机器），查看hadoop102机器中的meta表记录的目标表的位置（假设目标表在hadoop103机器）。将meta信息做一个缓存，下次先查缓存，如果缓存没有meta表信息，再找ZK
3. 访问hadoop103机器，向目标表发送put请求，先写WAL日志，然后将数据写入对应的Mem Store（内存），至此，客户端请求结束，向客户端返回ack。不需要等Mem Store刷到磁盘（因为已经有WAL了，所以认为数据不会丢了）

#### Mem Flush

- 写流程中，给客户端返回ack之后，HBase还会在合适时执行刷盘操作

  ![](C:\Users\LSD\Desktop\hbase\img\22.png)

- 最小的刷写单元是region，而不是单个的memstore，刷写时机：

  - 当某个`memstroe`的大小达到了`hbase.hregion.memstore.flush.size`（默认值 128M）， 其所在 `region`的所有`memstore`都会刷写。

    当`memstore`的大小达到了`hbase.hregion.memstore.flush.size(默认值 128M) * hbase.hregion.memstore.block.multiplier(默认值 4)` 时，会阻止继续往该`memstore`写数据。

  - 当`region server`中`memstore`的总大小达到`java_heapsize * hbase.regionserver.global.memstore.size(默认值 0.4) * hbase.regionserver.global.memstore.size.lower.limit(默认值 0.95)`，`region`会按照其所有 `memstore`的大小顺序（由大到小）依次进行刷写。直到`region server`中所有`memstore`的总大小减小到上述值以下。当`region server`中`memstore`的总大小达到`java_heapsize * hbase.regionserver.global.memstore.size(默认值 0.4)`，时，会阻止继续往所有的`memstore`写数据。

  - 到达自动刷写的时间，也会触发`memstore flush`。自动刷新的时间间隔由该属性进行 配置 `hbase.regionserver.optionalcacheflushinterval(默认 1 小时)`。

  - 当 WAL 文件的数量超过`hbase.regionserver.max.logs`，`region`会按照时间顺序依次进行刷写，直到 WAL 文件数量减小到`hbase.regionserver.max.log`以下（该属性名已经废弃， 现无需手动设置，最大值为 32）。

#### 写流程

![](C:\Users\LSD\Desktop\hbase\img\23.png)

1. 客户端连接ZK，查询meta表（即记录region的位置信息的表）所在的RegionServer是哪一个
2. 找到meta表所在的机器后（假设meta表在hadoop102机器），查看hadoop102机器中的meta表记录的目标表的位置（假设目标表在hadoop103机器）。将meta信息做一个缓存，下次先查缓存，如果缓存没有meta表信息，再找ZK
3. 访问hadoop103机器，向目标表发送get请求，分别在StoreFile（HFile）、MemStore、BlockCache（读缓存）中查询目标数据，并将查询到的数据按时间戳进行合并（同一条数据的不同版本或不同类型(Put/Delete)之间按时间戳合并）
4. 将从StoreFile（HFile）中查询到的数据缓存到Block Cache中
5. 将合并后的结果返回给客户端

#### StoreFile Compaction

- 由于memstore每次刷写都会生成一个新的HFile，且同一个字段的不同版本（timestamp） 和不同类型（Put/Delete）有可能会分布在不同的 HFile 中，因此查询时需要遍历所有的 HFile。为了减少 HFile 的个数，以及清理掉过期和删除的数据，会进行 StoreFile Compaction。 

- Compaction 分为两种，分别是 **Minor Compaction** 和 **Major Compaction**。

  - Minor Compaction 会将临近的若干个较小的 HFile 合并成一个较大的 HFile，但**不会***清理过期和删除的数据。 
  - Major Compaction 会将一个 Store 下的所有的 HFile 合并成一个大 HFile，并且**会**清理掉过期 和删除的数据

  ![](C:\Users\LSD\Desktop\hbase\img\24.png)

#### Region Split

- 默认情况下，每个 Table 起初只有一个 Region，随着数据的不断写入，Region 会自动进行拆分。刚拆分时，两个子 Region 都位于当前的 Region Server，但处于负载均衡的考虑， HMaster 有可能会将某个 Region 转移给其他的 Region Server。

- Region Split 时机：

  1. 当1个region中的某个Store下所有StoreFile的总大小超过`hbase.hregion.max.filesize`， 该 Region 就会进行拆分（0.94 版本之前）。 
  2.  当 1 个 region 中 的 某 个 Store 下所有 StoreFile 的 总 大 小 超 过 `Min(R^2 * "hbase.hregion.memstore.flush.size", hbase.hregion.max.filesize")`，该 Region 就会进行拆分，其 中 R 为当前 Region Server 中属于该 Table 的个数（0.94 版本之后）。

  ![](C:\Users\LSD\Desktop\hbase\img\25.png)

### API

#### api操作

#### HBase与Hive集成

##### HBase与Hive对比

- Hive

  - Hive 的本质其实就相当于将 HDFS 中已经存储的文件在 Mysql 中做了一个双射关系，以 方便使用 HQL 去管理查询。
  - Hive 适用于离线的数据分析和清洗，延迟较高。
  - Hive 存储的数据依旧在 DataNode 上，编写的 HQL 语句终将是转换为 MapReduce 代码执 行。

- HBase

  - HBase是一种**面向列族存储**的非关系型数据库。
  - HBase适用于单表非关系型数据的存储，不适合做关联查询，类似 JOIN 等操作。

  - HBase基于HDFS，数据持久化存储的体现形式是 HFile，存放于 DataNode 中，被 ResionServer 以 region 的形 式进行管理。
  - HBase延迟较低，接入在线业务使用，面对大量的企业数据，HBase 可以直线单表大量数据的存储，同时提供了高效的数据访问 速度。

### HBase优化

#### 高可用

#### 预分区

- 每一个 region 维护着 StartRow 与 EndRow，如果加入的数据符合某个 Region 维护的 RowKey 范围，则该数据交给这个 Region 维护。那么依照这个原则，我们可以将数据所要 投放的分区提前大致的规划好，以提高 HBase 性能。

  - 手动设置预分区

    ![](C:\Users\LSD\Desktop\hbase\img\26.png)

  - 生成分区

    ![](C:\Users\LSD\Desktop\hbase\img\27.png)

  - 按16进制序列预分区，指定分区数

    ![](C:\Users\LSD\Desktop\hbase\img\28.png)

  - 生成分区

    ![](C:\Users\LSD\Desktop\hbase\img\29.png)

    ![](C:\Users\LSD\Desktop\hbase\img\30.png)

  - 文件中预设分区

    ![](C:\Users\LSD\Desktop\hbase\img\31.png)

    ![](C:\Users\LSD\Desktop\hbase\img\32.png)

    ![](C:\Users\LSD\Desktop\hbase\img\33.png)

  - 生成分区

    ![](C:\Users\LSD\Desktop\hbase\img\34.png)

  - API中创建预分区

    ```java
    //自定义算法，产生一系列 hash 散列值存储在二维数组中
    byte[][] splitKeys = 某个散列值函数
    //创建 HbaseAdmin 实例
    HBaseAdmin hAdmin = new HBaseAdmin(HbaseConfiguration.create());
    //创建 HTableDescriptor 实例
    HTableDescriptor tableDesc = new HTableDescriptor(tableName);
    //通过 HTableDescriptor 实例和散列值二维数组创建带有预分区的 Hbase 表
    hAdmin.createTable(tableDesc, splitKeys);
    ```

#### RowKey设计

- 一条数据的唯一标识就是RowKey，这条数据存储于哪个分区，取决于 RowKey 处于哪一个预分区的区间内，设计 RowKey 的主要目的，就是让数据均匀的分布于所有的 region 中，在一定程度上防止数据倾斜。
  - **散列性**：RowKey尽量均匀散落在多个分区中
  - **唯一性**：RowKey唯一标识一条数据
  - **长度原则**：70-100位

#### 内存优化

- HBase 操作过程中需要大量的内存开销，毕竟 Table 是可以缓存在内存中的，一般会分 配整个可用内存的 70%给 HBase 的 Java 堆。但是**不建议分配非常大的堆内存**，因为 GC 过 程持续太久会导致 RegionServer 处于长期不可用状态，一般 16~48G 内存就可以了，如果因 为框架占用内存过高导致系统内存不足，框架一样会被系统服务拖死。

#### 基础优化

1. 允许在 HDFS 的文件中追加内容

   `hdfs-site.xml、hbase-site.xml`

   ```
   属性：dfs.support.append
   解释：开启 HDFS 追加同步，可以优秀的配合 HBase 的数据同步和持久化。默认值为 true。
   ```

2. 优化 DataNode 允许的最大文件打开数

   `hdfs-site.xml`

   ```
   属性：dfs.datanode.max.transfer.threads
   解释：HBase 一般都会同一时间操作大量的文件，根据集群的数量和规模以及数据动作，
   设置为 4096 或者更高。默认值：4096
   ```

3. 优化延迟高的数据操作的等待时间

   `hdfs-site.xml`

   ```
   属性：dfs.image.transfer.timeout
   解释：如果对于某一次数据操作来讲，延迟非常高，socket 需要等待更长的时间，建议把该值设置为更大的值（默认 60000 毫秒），以确保 socket 不会被 timeout 掉。
   ```

4. 优化数据的写入效率

   `mapred-site.xml`

   ```
   属性：
   mapreduce.map.output.compress
   mapreduce.map.output.compress.codec
   解释：开启这两个数据可以大大提高文件的写入效率，减少写入时间。第一个属性值修改为true，第二个属性值修改为：org.apache.hadoop.io.compress.GzipCodec 或者其他压缩方式。
   ```

5. 设置 RPC 监听数量

   `hbase-site.xml`

   ```
   属性：Hbase.regionserver.handler.count
   解释：默认值为30，用于指定 RPC 监听的数量，可以根据客户端的请求数进行调整，读写请求较多时，增加此值。
   ```

6. 优化 HStore 文件大小

   `hbase-site.xml`

   ```
   属性：hbase.hregion.max.filesize
   解释：默认值 10737418240（10GB），如果需要运行 HBase 的 MR 任务，可以减小此值，因为一个 region 对应一个 map 任务，如果单个 region 过大，会导致 map 任务执行时间过长。该值的意思就是，如果 HFile 的大小达到这个数值，则这个 region 会被切分为两个 Hfile。
   ```

7. 优化 HBase 客户端缓存

   `hbase-site.xml`

   ```
   属性：hbase.client.write.buffer
   解释：用于指定 Hbase 客户端缓存，增大该值可以减少 RPC 调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到减少 RPC 次数的目的。
   ```

8. 指定 scan.next 扫描 HBase 所获取的行数

   `hbase-site.xml`

   ```
   属性：hbase.client.scanner.caching
   解释：用于指定 scan.next 方法获取的默认行数，值越大，消耗内存越大。
   ```

9. flush、compact、split 机制

   flush：当 MemStore 达到阈值（默认128M），将 Memstore 中的数据 Flush 进 Storefile；

   compact机制：把 flush 出来的小文件合并成大的 Storefile 文件；

   split机制：当 Region 达到阈值，会把过大的 Region 一分为二。

   ```
   属性：hbase.hregion.memstore.flush.size
   解释：这个参数的作用是当单个HRegion 内所有的Memstore大小总和超过指定值时，flush该HRegion的所有 memstore。RegionServer的flush是通过将请求添加一个队列，模拟生产消费模型来异步处理的。那这里就有一个问题，当队列来不及消费，产生大量积压请求时，可能会导致内存陡增，最坏的情况是触发 OOM。
   属性：
   hbase.regionserver.global.memstore.upperLimit：0.4
   hbase.regionserver.global.memstore.lowerLimit：0.38
   解释：当 MemStore 使用内存总量达到hbase.regionserver.global.memstore.upperLimit 指定值时，将会有多个 MemStores flush 到文件中，MemStore flush 顺序是按照大小降序执行的，直到刷新到 MemStore使用内存略小于lowerLimit
   ```

   

