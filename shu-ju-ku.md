# 数据库

## 缓存架构

**进程内缓存**

**到底选Redis还是Memcache，看看源码怎么说**

* 什么时候倾向于选择Redis
  * 复杂数据结构
  * 持久化：mc无法满足持久化的需求，只得选择redis
  * 天然高可用：支持主从
  * 存储的内容比较大：memcache的value存储，最大为1M，如果存储的value很大，只能使用redis。
* 什么时候倾向于选择memcache
  * 纯KV，数据量非常大，并发量非常大的业务，使用memcache或许更适合。

**如何应对潜在的雪崩？**

* 常见方案一：高可用缓存
* 常见方案二：缓存水平切分
* db限流
* 快速失败

**究竟先操作缓存，还是数据库？**

* 读请求，先读缓存，如果没有命中，读数据库，再set回缓存
* 写请求
  * 先缓存，再数据库
  * 缓存，使用delete，而不是set 

**Cache Aside Pattern（旁路缓存模式）**

* 对于读请求
  * 先读cache，再读db
  * 如果，cache hit，则直接返回数据
  * 如果，cache miss，则访问db，并将数据set回缓存
* 对于写请求
  * 淘汰缓存，而不是更新缓存
  * 先操作数据库，再淘汰缓存
* Cache Aside Pattern方案存在什么问题？

  如果先操作数据库，再淘汰缓存，在原子性被破坏时：

  * 修改数据库成功了
  * 淘汰缓存失败了

  导致，数据库与缓存的数据不一致。

**数据库主从不一致，怎么解？**

数据库主库和从库不一致，常见有这么几种优化方案：

* 业务可以接受，系统不优化
* 强制读主，高可用主库，用缓存提高读性能
* 在cache里记录哪些记录发生过写请求，来路由读主还是读从

## 数据库

### 概论

* redis、memcache、mongoDB 对比

  * 操作的便利性

    memcache数据结构单一

    redis丰富一些，数据操作方面，redis更好一些，较少的网络IO次数

    mongodb支持丰富的数据表达，索引，最类似关系型数据库，支持的查询语言非常丰富

  ​

### Mongodb

### MySQL

#### 索引

* 索引基数：唯一值越多，基数越大，越有可能选择这个索引

#### 性能优化

**慢查询**

**查询缓存**

| 变量名 | 说明 |
| :--- | :--- |
| Qcache\_free\_blocks | 缓存中相邻内存块的个数。数目大说明可能有碎片。FLUSH QUERY CACHE 会对缓存中的碎片进行整理，从而得到一个空闲块。 |
| Qcache\_free\_memory | 缓存中的空闲内存。 |
| Qcache\_hits | 每次查询在缓存中命中时就增大。 |
| Qcache\_inserts | 每次插入一个查询时就增大。命中次数除以插入次数就是不中比率；用 1 减去这个值就是命中率。在上面这个例子中，大约有 87% 的查询都在缓存中命 |
| Qcache\_lowmem\_prunes | 缓存出现内存不足并且必须要进行清理以便为更多查询提供空间的次数。这个数字最好长时间来看；如果这个数字在不断增长，就表示可能碎片非常严重，或者内存很少。（上面的 free\_blocks 和free\_memory 可以告诉您属于哪种情况）。 |
| Qcache\_not\_cached | 不适合进行缓存的查询的数量，通常是由于这些查询不是 SELECT 语句。 |
| Qcache\_queries\_in\_cache | 当前缓存的查询（和响应）的数量。 |
| Qcache\_total\_blocks | 缓存中块的数量。 |

**缓冲区和缓存**

* 服务端的设置

  * 显示打开表的活动

  ```text
  mysql> SHOW STATUS LIKE 'open%tables';
  +---------------+-------+
  | Variable_name | Value |
  +---------------+-------+
  | Open_tables   | 5000  |
  | Opened_tables | 195   |
  +---------------+-------+
  2 rows in set (0.00 sec)
  ```

  * 显示线程使用统计信息

  ```text
  mysql> SHOW STATUS LIKE 'threads%';
  +-------------------+--------+
  | Variable_name     | Value  |
  +-------------------+--------+
  | Threads_cached    | 27     |
  | Threads_connected | 15     |
  | Threads_created   | 838610 |
  | Threads_running   | 3      |
  +-------------------+--------+
  4 rows in set (0.00 sec)
  ```

  * 确定关键字效率

  ```text
  mysql> show status like '%key_read%';
  +-------------------+-----------+
  | Variable_name     | Value     |
  +-------------------+-----------+
  | Key_read_requests | 163554268 |
  | Key_reads         | 98247     |
  +-------------------+-----------+
  2 rows in set (0.00 sec)
  ```

  * 确定临时表的使用

  ```text
  mysql> SHOW STATUS LIKE 'created_tmp%';
  +-------------------------+-------+
  | Variable_name           | Value |
  +-------------------------+-------+
  | Created_tmp_disk_tables | 30660 |
  | Created_tmp_files       | 2     |
  | Created_tmp_tables      | 32912 |
  +-------------------------+-------+
  3 rows in set (0.00 sec)
  ```

* 每个会话的设置
  * **显示排序统计信息**

    ```text
    mysql> SHOW STATUS LIKE "sort%";
    +-------------------+---------+
    | Variable_name     | Value   |
    +-------------------+---------+
    | Sort_merge_passes | 1       |
    | Sort_range        | 79192   |
    | Sort_rows         | 2066532 |
    | Sort_scan         | 44006   |
    +-------------------+---------+
    4 rows in set (0.00 sec)
    ```

  * 确定表扫描比率

    ```text
    mysql> SHOW STATUS LIKE "com_select";
    +---------------+--------+
    | Variable_name | Value  |
    +---------------+--------+
    | Com_select    | 318243 |
    +---------------+--------+
    1 row in set (0.00 sec)

    mysql> SHOW STATUS LIKE "handler_read_rnd_next";
    +-----------------------+-----------+
    | Variable_name         | Value     |
    +-----------------------+-----------+
    | Handler_read_rnd_next | 165959471 |
    +-----------------------+-----------+
    1 row in set (0.00 sec)
    ```

#### MVCC

> [https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html](https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html)

**mysql三个隐藏的key**

* 一个6字节的DB\_TRX\_ID指示最后一个插入或更新当前行的事务
* 1个字节用来标记当前行时候被删除
* 一个7字节的DB\_ROLL\_PTR回滚指针，指向undo log

#### 排它锁和共享锁

排他锁、共享锁。共享锁针对读，排他锁针对写，完全等同读写锁的概念。如果某个事务在更新某行（排他锁），则其他事物无论是读还是写本行都必须等待；如果某个事物读某行（共享锁），则其他读的事物无需等待，而写事物则需等待。

#### 事务

> [https://amao12580.github.io/post/2016/06/What-is-a-transaction/](https://amao12580.github.io/post/2016/06/What-is-a-transaction/)
>
> [http://tech.meituan.com/innodb-lock.html](http://tech.meituan.com/innodb-lock.html)
>
> [http://hedengcheng.com/?p=148](http://hedengcheng.com/?p=148)

#### ACID

#### 隔离级别

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| :--- | :--- | :--- | :--- |
| 未提交读\(Read Uncommitted\) | 可能 | 可能 | 可能 |
| 已提交读\(Read Commited\) | 不可能 | 可能 | 可能 |
| 重复读 \(Repeatable read\) | 不可能 | 不可能 | 可能 |
| 序列化\(\(Serializable\) | 不可能 | 不可能 | 不可能 |

* 脏读：当一个事务对数据进行修改但未提交事务，另外一个事务访问并使用了这个数据
* 不可重复读：事务1，多次读取同一数据。在两次读取之间，事务2更新了此数据，事务1在两次会读到不同的数据，因此称为不可重复读。
* 幻读：事务1对表所有数据做了更新，事务2向表中插入一条新数据。事务1后续会发现还有没更新的数据，就好像产生了幻觉一样。

#### 存储引擎

#### MyISAM与InnoDB索引的区别\#\#\#\#

* innodb 支持行级锁，myisam只支持表索引
* MyISAM 支持全文索引，Innodb 直到5.6才支持
* Innodb实现了事务、外键。MyISAM不支持

#### InnoDB的Primary Index 和 secondary Index有何区别？\#\#\#\#

#### int\(4\) vs int\(11\)

The x in INT\(x\) has nothing to do with space requirements or any other performance issues, it's really just the display width. Generally setting the display widths to a reasonable value is mostly useful with the UNSIGNED ZEROFILL option.

* the `UNSIGNED ZEROFILL` option.

```text
  //INT(4) UNSIGNED ZEROFILL
  0001
  0002 
  ...
  0099
  ...
  0999
  ...
  9999
  ...
  10000

  //INT(2) UNSIGNED ZEROFILL
  01
  02 
  ...
  09
  ...
  99
  ...
  100
```

* Without the `UNSIGNED ZEROFILL` option

  ```text
  //INT(4)
     1
     2 
  ...
    99
  ...
   999
  ...
  9999
  ...
  10000

  //INT(2)
   1
   2 
  ...
   9
  ...
  99
  ...
  100
  ```

#### varchar vs char

* variable-length vs fixed

The length of a `CHAR` column is fixed to the length that you declare when you create the table.

Values in `VARCHAR` columns are variable-length strings.

* varchar 存储字符串长度

In contrast to `CHAR`, `VARCHAR` values are stored as a 1-byte or 2-byte length prefix plus data.

* varchar trailing space

If a given value is stored into the `CHAR(4)` and `VARCHAR(4)` columns, the values retrieved from the columns are not always the same because trailing spaces are removed from `CHAR` columns upon retrieval. The following example illustrates this difference:

```sql
mysql> CREATE TABLE vc (v VARCHAR(4), c CHAR(4));
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO vc VALUES ('ab  ', 'ab  ');
Query OK, 1 row affected (0.00 sec)

mysql> SELECT CONCAT('(', v, ')'), CONCAT('(', c, ')') FROM vc;
+---------------------+---------------------+
| CONCAT('(', v, ')') | CONCAT('(', c, ')') |
+---------------------+---------------------+
| (ab  )              | (ab)                |
+---------------------+---------------------+
1 row in set (0.06 sec)
```

#### DELETE TRUNCATE

Basically DELETE TABLE is logged operation and every row deleted is logged. Therefore the process is usually slow. TRUNCATE TABLE also deletes rows in a table but it will not log any of the rows deleted. The process is faster in comparison. TRUNCATE TABLE can be rolled back and is functionally similar to the DELETE statement using no WHERE clause.

#### float double

* Floating point numbers are stored in FLOAT with eight place accuracy and it has four bytes.
* Floating point numbers are stored in DOUBLE with accuracy of 18 places and it has eight bytes.

  float tips

  > MySQL permits a nonstandard syntax: FLOAT\(M,D\) or REAL\(M,D\) or DOUBLE PRECISION\(M,D\). Here, “\(M,D\)” means than values can be stored with up to M digits in total, of which D digits may be after the decimal point. For example, a column defined as FLOAT\(7,4\) will look like -999.9999 when displayed. MySQL performs rounding when storing values, so if you insert 999.00009 into a FLOAT\(7,4\) column, the approximate result is 999.0001.

#### varchar\(n\) 可以存储几个中文

varchar\(n\) 表示n个字符，无论汉字和英文，`MySql`都能存入 `n` 个字符，仅实际字节长度有所区别。

### Redis

#### 复制

> [https://redis.io/topics/replication](https://redis.io/topics/replication)

#### 数据一致性

#### 数据持久化

**RDB快照**

1. fork出一个子进程
2. 通过save 指令配置10分钟以内有1000次写入就生成快照
3. rdb文件不会损坏：临时文件，通过原子操作rename

**AOF日志**

1. 不记录对数据无影响的命令
2. fork进程，遍历数据
3. 清空缓存，rename临时文件
4. AOF可靠性设置

   ```text
   appendfsync no
   appendfsync everysec 
   appendfsync always
   ```

   ​

#### 数据结构

字符串、散列、列表、集合、有序集合、HyperLogLog

#### 配置

* daemonize no 默认情况下，redis不是在后台运行的，如果需要在后台运行，把该项的值更改为yes

```bash
daemonize yes
```

* timeout设置客户端连接时的超时时间，单位为秒。当客户端在这段时间内没有发出任何指令，那么关闭该连接，0是关闭此设置
* loglevel 日志级别
* logfile 日志文件地址
* 可用数据库 database 

**rdb**

* 快照save   指出在多长时间有多少次更新，就将数据同步到文件rdb。
* rdbcompression yes 存储至本地数据时（持久化到rdb文件）是否压缩数据，默认为yes。
* dbfile dumo.rdb 本地持久化数据库文件名
* 数据库镜像备份的文件放置的路径。  `dir ./` 

**复制**

* 主从复制. 设置该数据库为其他数据库的从数据库.  `slaveof <masterip> <masterport>`
* 当master服务设置了密码保护时\(用requirepass制定的密码\) `masterauth <master-password>`
* `slave-serve-stale-data yes`如果slave-serve-stale-data设置为yes\(默认设置\)，从库会继续相应客户端的请求。如果slave-serve-stale-data是指为no，出去INFO和SLAVOF命令之外的任何请求都会返回一个错误"SYNC with master in progress"

**约束**

* `maxclients 128`设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数。
* `maxmemory <bytes>` 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key,Redis同时也会移除空的list对象
* `maxmemory-policy volatile-lru` 当内存达到最大值的时候Redis会选择删除哪些数据？

  有五种方式可供选择

  * volatile-lru -&gt; 利用LRU算法移除设置过过期时间的key \(LRU:最近使用 Least Recently Used \)
  * allkeys-lru -&gt; 利用LRU算法移除任何key
  * volatile-random -&gt; 移除设置过过期时间的随机key
  * allkeys-&gt;random -&gt; remove a random key, any key
  * volatile-ttl -&gt; 移除即将过期的key\(minor TTL\)
  * noeviction -&gt; 不移除任何可以，只是返回一个写错误

**AOF**

* 开启append only模式之后，redis会把所接收到的每一次写操作请求都追加到appendonly.aof文件中，当redis重新启动时，会从该文件恢复出之前的状态。

```text
appendonly no
appendfilename appendonly.aof
```

Redis支持三种同步AOF文件的策略:

* no: 不进行同步，系统去操作 . Faster.
* always: always表示每次有写操作都进行同步. Slow, Safest.
* everysec: 表示对写操作进行累积，每秒同步一次. Compromise.
* 当AOF文件增长到一定大小的时候Redis能够调用 BGREWRITEAOF 对日志文件进行重写 

**slow log**

可以通过两个参数设置slow log

* 一个是告诉Redis执行超过多少时间被记录的参数slowlog-log-slower-than\(微妙\)，
* 另一个是slow log 的长度。当一个新命令被记录的时候最早的命令将被从队列中移除

#### 高级配置

**hash**

* 当hash中包含超过指定元素个数并且最大的元素没有超过临界时，hash将以一种特殊的编码方式（大大减少内存使用）来存储，Redis Hash对应Value内部实际就是一个HashMap，实际这里会有2种不同实现.这个Hash的成员比较少时Redis为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的HashMap结构，对应的value redisObject的encoding为zipmap。当成员数量增大时会自动转成真正的HashMap,此时encoding为ht。

```text
hash-max-zipmap-entries 512
hash-max-zipmap-value 64
```

**list**

* list数据类型多少节点以下会采用去指针的紧凑存储格式。
* list数据类型节点值大小小于多少字节会采用紧凑存储格式。

```text
list-max-ziplist-entries 512
list-max-ziplist-value 64
```

**set**

* set数据类型内部数据如果全部是数值型，且包含多少节点以下会采用紧凑格式存储。

```text
set-max-intset-entries 512
```

zsort

* zsort数据类型多少节点以下会采用去指针的紧凑存储格式。
* zsort数据类型节点值大小小于多少字节会采用紧凑存储格式。

```text
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
```

**include**

* 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件

### codis

### Predis

### memcache

#### cas

**check and set \(乐观锁的概念\)**

Memcached 1.2.5以及更高版本，提供了gets和cas命令，它们可以解决上面的问题。如果您使用gets命令查询某个key的cache会给您返回该item当前值的唯一标识。如果您覆写了这个item并想把它写回到Memcached中，您可以通过cas命令把那唯一标识送给 Memcached。如果该Item存放在Memcached中的唯一标识与您提供的一致，您的写操作将会成功。如果另一个进程在这期间也修改了这个Item，那么该Item存放在Memcached中的唯一标识将会改变，您的写操作就会失败。

## swoole

### 异步mysql

> [http://www.programgo.com/article/65853285546/](http://www.programgo.com/article/65853285546/)
>
> [http://rango.swoole.com/archives/288](http://rango.swoole.com/archives/288)

## zookeeper

> [https://www.ibm.com/developerworks/cn/data/library/bd-zookeeper/](https://www.ibm.com/developerworks/cn/data/library/bd-zookeeper/)
>
> [http://zookeeper.apache.org/doc/trunk/zookeeperTutorial.html?cm\_mc\_uid=76943637784814694455375&cm\_mc\_sid\_50200000=1489242071](http://zookeeper.apache.org/doc/trunk/zookeeperTutorial.html?cm_mc_uid=76943637784814694455375&cm_mc_sid_50200000=1489242071)

### 部署架构图

![kafka&#x67B6;&#x6784;&#x56FE;](https://cwiki.apache.org/confluence/download/attachments/24193436/service.png?version=1&modificationDate=1295027310000&api=v2)

## MapReduce

> [http://blog.csdn.net/v\_july\_v/article/details/6637014](http://blog.csdn.net/v_july_v/article/details/6637014)

## 消息队列

### Kafka

> [https://www.ibm.com/developerworks/cn/opensource/os-cn-kafka/](https://www.ibm.com/developerworks/cn/opensource/os-cn-kafka/)
>
> [http://www.infoq.com/cn/articles/kafka-analysis-part-1](http://www.infoq.com/cn/articles/kafka-analysis-part-1)

#### 设计目标

* 以时间复杂度为O\(1\)的方式提供消息持久化能力，即使对TB级以上数据也能保证常数时间复杂度的访问性能。
* 支持Kafka Server间的消息分区，及分布式消费，同时保证每个Partition内的消息顺序传输。
* 同时支持离线数据处理和实时数据处理。
* Scale out：支持在线水平扩展。

## LVS

## 安全

### sql注入

### xss

## RPC调用

### thrift

## 加密算法

### 基本概念

#### DES加密算法

DES加密算法是一种分组密码，以64位为分组对数据加密，它的密钥长度是56位，加密解密用同一算法。DES加密算法是对密钥进行保密，而公开算法，包括加密和解密算法。这样，只有掌握了和发送方相同密钥的人才能解读由DES[加密算法](http://www.cksis.com/blog/category/jiamisuanfa)加密的密文数据。因此，破译DES加密算法实际上就是搜索密钥的编码。对于56位长度的密钥来说，如果用穷举法来进行搜索的话，其运算次数为256。

随着计算机系统能力的不断发展，DES的安全性比它刚出现时会弱得多，然而从非关键性质的实际出发，仍可以认为它是足够的。不过，DES现在仅用于旧系统的鉴定，而更多地选择新的加密标准。

#### AES加密算法

AES加密算法是密码学中的高级加密标准，该加密算法采用对称分组密码体制，密钥长度的最少支持为128、192、256，分组长度128位，算法应易于各种硬件和软件实现。这种加密算法是美国联邦政府采用的区块加密标准，这个标准用来替代原先的DES，已经被多方分析且广为全世界所使用。

#### RSA加密算法

RSA加密算法是目前最有影响力的[公钥加密](http://www.cksis.com/blog/892-gongyuejiamisuanfa.html)算法，并且被普遍认为是目前最优秀的公钥方案之一。RSA是第一个能同时用于加密和数宇签名的算法，它能够抵抗到目前为止已知的所有密码攻击，已被ISO推荐为公钥数据加密标准。RSA加密算法基于一个十分简单的数论事实：将两个大素数相乘十分容易，但那时想要，但那时想要对其乘积进行因式分解却极其困难，因此可以将乘积公开作为加密密钥。

#### base64加密算法

Base64加密算法是网络上最常见的用于传输8bit字节代码的编码方式之一，Base64编码可用于在HTTP环境下传递较长的标识信息。例如，在JAVAPERSISTENCE系统HIBEMATE中，采用了Base64来将一个较长的唯一标识符编码为一个字符串，用作HTTP表单和HTTPGETURL中的参数。在其他应用程序中，也常常需要把二进制数据编码为适合放在URL（包括隐藏表单域）中的形式。此时，采用Base64编码不仅比较简短，同时也具有不可读性，即所编码的数据不会被人用肉眼所直接看到。

#### MD5加密算法

MD5为计算机安全领域广泛使用的一种散列函数，用以提供消息的完整性保护。对[MD5加密算法](http://www.cksis.com/blog/316-md5jiamiyingyong.html)简要的叙述可以为：MD5以512位分组来处理输入的信息，且每一分组又被划分为16个32位子分组，经过了一系列的处理后，算法的输出由四个32位分组组成，将这四个32位分组级联后将生成—个128位散列值。

#### SHA1加密算法

SHA1是和MD5一样流行的消息摘要算法。SHA加密算法模仿MD4加密算法。SHA1设计为和数字签名算法（ＤＳＡ）一起使用。

## 字符编码&字符集

> [http://cenalulu.github.io/linux/character-encoding/](http://cenalulu.github.io/linux/character-encoding/)

### 字符集

字符集就规定了某个文字对应的二进制数字存放方式（编码）和某串二进制数值代表了哪个文字（解码）的转换关系。

### 字符编码

* `字库表`决定了整个字符集能够展现表示的所有字符的范围。
* `编码字符集`，即用一个编码值`code point`来表示一个字符在字库中的位置。
* `字符编码`，将编码字符集和实际存储数值之间的转换关系。

### UTF-8和Unicode的关系

* Unicode就是上文中提到的编码字符集，而UTF-8就是字符编码，即Unicode规则字库的一种实现形式。
* Unicode的编号从`0000`开始一直到`10FFFF`共分为16个Plane，每个Plane中有65536个字符。而UTF-8则只实现了第一个Plane，可见UTF-8虽然是一个当今接受度最广的字符集编码，但是它并没有涵盖整个Unicode的字库，这也造成了它在某些场景下对于特殊字符的处理困难（下文会有提到）。

### UTF-8编码简介

UTF-8编码为变长编码。最小编码单位（`code unit`）为一个字节。一个字节的前1-3个bit为描述性部分，后面为实际序号部分。

* 如果一个字节的第一位为0，那么代表当前字符为单字节字符，占用一个字节的空间。0之后的所有部分（7个bit）代表在Unicode中的序号。
* 如果一个字节以110开头，那么代表当前字符为双字节字符，占用2个字节的空间。110之后的所有部分（5个bit）加上后一个字节的除10外的部分（6个bit）代表在Unicode中的序号。且第二个字节以10开头
* 如果一个字节以1110开头，那么代表当前字符为三字节字符，占用2个字节的空间。110之后的所有部分（5个bit）加上后两个字节的除10外的部分（12个bit）代表在Unicode中的序号。且第二、第三个字节以10开头
* 如果一个字节以10开头，那么代表当前字节为多字节字符的第二个字节。10之后的所有部分（6个bit）和之前的部分一同组成在Unicode中的序号。

细心的读者不难从以上的简单介绍中得出以下规律：

* 3个字节的UTF-8十六进制编码一定是以`E`开头的
* 2个字节的UTF-8十六进制编码一定是以`C`或`D`开头的
* 1个字节的UTF-8十六进制编码一定是以比`8`小的数字开头的

## 工作内容

### 任务发布

## 微信红包

> [http://www.infoq.com/cn/articles/2017hongbao-weixin](http://www.infoq.com/cn/articles/2017hongbao-weixin)
>
> [http://colobu.com/2015/05/04/weixin-red-packets-design-discussion/](http://colobu.com/2015/05/04/weixin-red-packets-design-discussion/)

### 微信红包两大业务特点

1. 微信红包业务比普通商品“秒杀”有更海量的并发要求。
2. 微信红包业务要求更严格的安全级别。

### 微信红包技术难点

1. 事务级操作量大
2. 事务性要求严格

### 解决高并发问题通常使用的方案

1. 使用内存操作替代实时的DB事务操作。

   > 这种方案的优点是通过用内存操作代替磁盘，提高了并发性能。两个存储可能失败

2. 使用乐观锁替代悲观锁。

   > 所谓悲观锁，是关系数据库管理系统里的一种并发控制的方法。它可以阻止一个事务以影响其他用户的方式来修改数据。如果一个事务执行的操作对某行数据应用了锁，那只有当这个事务把锁释放，其他事务才能够执行与该锁冲突的操作。对应于上文分析中的“并发请求抢锁”行为。
   >
   > 所谓乐观锁，它假设多用户并发的事务在处理时不会彼此互相影响，各事务能够在不产生锁的情况下处理各自影响的那部分数据。在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其他事务又修改了该数据。如果其他事务有更新的话，正在提交的事务会进行回滚。
   >
   > 商品“秒杀”系统中，乐观锁的具体应用方法，是在DB的“库存”记录中维护一个版本号。在更新“库存”的操作进行前，先去DB获取当前版本号。在更新库存的事务提交时，检查该版本号是否已被其他事务修改。如果版本没被修改，则提交事务，且版本号加1；如果版本号已经被其他事务修改，则回滚事务，并给上层报错。

   应用于微信红包系统，则会存在下面三个问题

   > 1. 拆红包采用乐观锁，并发抢到相同版本号的拆红包请求中，只有一个能拆红包成功，其他的请求将事        务回滚并返回失败，给用户报错，用户体验完全不可接受。
   > 2. 如果采用乐观锁，将会导致第一时间同时拆红包的用户有一部分直接返回失败，反而那些“手慢”的用户，有可能因为并发减小后拆红包成功，这会带来用户体验上的负面影响。
   > 3. 如果采用乐观锁的方式，会带来大数量的无效更新请求、事务回滚，给DB造成不必要的额外压力。

### 微信红包系统的高并发解决方案

* 系统垂直SET化，分而治之。
* 逻辑Server层将请求排队，解决DB并发问题。
  * 将同一个红包ID的所有请求stick到同一台Server。
  * 设计单机请求排队方案。
  * 增加memcached控制并发。
* 双维度库表设计，保障系统性能稳定
  * 初期是根据红包ID的hash值分为多库多表。
  * 采用冷热分离，将历史冷数据与当前热数据分开存储，可以解决这个问题。

## 分布式事务

coolshell的解析[http://coolshell.cn/articles/10910.html](http://coolshell.cn/articles/10910.html)

## 前言\#\#

**一台\(单点\)服务器提供数据服务时，会遇到的问题**

* 一台服务器的性能不足以提供足够的能力服务所有请求
* 如果这台服务器宕机，将会造成服务不可用甚至丢失数据

**解决单点问题，引入更多服务器，两种手段来扩展数据服务**

* 数据分区
* 数据镜像

**以上方案中需要考虑的问题总结为**

* 容灾：数据不丢
* 数据一致性：事务处理
* 性能

#### 2pc，又称两阶段提交

引入协调者、参与者来保证ACID，分为两个阶段如下：

**第一阶段**

1. 协调者询问所有参与者，是否可以执行提交操作
2. 参与者开始事务执行的准备工作，预留资源、锁定资源、写redo\undolog
3. 响应协调者是否可以提交

**第二阶段**

1. 如果所有参与者都回应可以提交。那么，协调者向所有参与者发送“正式提交”的命令。参与者完成提交工作，响应完成
2. 如果有一个参与者拒绝提交，则协调者向所有参与者发送回滚消息

**2pc存在的最大问题**

如果一阶段完成，参与者第二阶段没有收到决策，那么数据节点会进入“不知所措”的状态，这个状态会block整个事务。

#### 3pc，又称三阶段提交，2pc的升级版

3pc与2pc的主要区别在于：在第一阶段询问是否可以提交的时候，不锁定资源。除非所有参与者都回应可以提交，才开始在第二阶段锁定资源

### PAXOS

> [https://www.zhihu.com/question/19787937](https://www.zhihu.com/question/19787937)
>
> 作者博客[http://lamport.azurewebsites.net/pubs/pubs.html\#lamport-paxos](http://lamport.azurewebsites.net/pubs/pubs.html#lamport-paxos)
>
> coolshell的解析[http://coolshell.cn/articles/10910.html](http://coolshell.cn/articles/10910.html)

**基本概念**

PAXOS 算法，集群中任何一个节点都可以发起数据修改的提案，最终提案是否通过取决于集群中是否有过半数的节点同意。

他是一个民主选举算法，过半数节点的决定会成为整个集群的统一决定。

## 其他

### 微信序列号生成器架构设计及演变

> [http://www.infoq.com/cn/articles/wechat-serial-number-generator-architecture](http://www.infoq.com/cn/articles/wechat-serial-number-generator-architecture)

#### 配置开发机以让外网访问

#### 你有哪些得意之作

#### 如何做测试

#### 接触到的Mysql数据库表最大是多少行记录?

#### 出现性能瓶颈怎么解决？

#### 学习一个新框架如何入手?

## 人力问题

#### 你最大的缺点是什么

面试官通常考察影应聘者的因素，从深往浅主要有：三观，性格特点，思维方式，技能，知识。

想象一个冰山模型，最下面的是三观，往上是性格特点，思维方式，这三个都在水面下，是先天遗传和后天环境交互影响的结果。水面上的是技能和知识，主要依赖后天的学习和训练来养成。发现没有，水面下的三个因素，因为涉及遗传这个不可抗力，会更稳定，也就是缺少了可塑性。这些地方出了问题，可以改，但是难度系数大，收益效率低。水面上的两点，可塑性大。如果这个地方出了问题，改起来容易。请自动类比一台CPU坏了的电脑和一台F键坏了的电脑，哪台让你更揪心？

_**手动加粗**_

> 在谈到自己缺点的时候，尽量避开三观，性格方面的缺点。思维方式作为可选项，但不是优选项。最好还是着眼于知识和技能。因为这两点改进空间大，速度快。

**手动加粗2**

往高处说，往远处说

1. **往高处说：能力层次有高有低，请你挑一个与你目前所在层次相隔较远的能力缺陷来说。**
2. **术业有专攻，找一个与你本职工作间隔较远的专业能力缺陷来说。**

   ​

## 目标公司

* 美团    望京
* 链家网

