# Redis

### Codis

[http://www.infoq.com/cn/presentations/design-and-implementation-of-wandoujia-distributed-redis](http://www.infoq.com/cn/presentations/design-and-implementation-of-wandoujia-distributed-redis)

## 数据结构

### 字符串

### 链表

链表节点

```c
typedef struct listNode{
  //前置节点
  struct listNode * prev；
    //后置节点
    struct listNode * next；
      //节点的值
    void * value;
} listNode;
```

链表结构体

```c
typedef struct  list{
  //表头节点
  //表尾节点
  //链表所包含的节点数量
  //节点值复制函数
  //节点值释放函数
  //节点值对比函数
}
```

### 字典

字典节点

```c
typedef struct dictEntry{
  //key
  void *key,
  //value
  union{
    void *val;
    uint64_tu64;
    int64_ts64;
  } v;
  struct dictEntry *next;
} dictEntry;
```

字典结构体

```c
typedef struct dict{
  //类型特定函数
  dictType *type;
  //私有函数
  void *privdata;
  //hash table
  dictht ht[2];
  int trehashidx;
}dict;
```

#### 解决冲突

链表

#### 字典rehash

**hash表的扩展与收缩**

**扩展操作**

* 扩展启动条件
  * 服务器目前没有执行BGSAVE或BGREWRITEAOF命令，并且hash表的负载因子大于等于1
  * 服务器目前正在执行BGSAVE或者BGREWRITEOF命令，并且hash标的负载因子大与等于5

如果执行的是扩展操作，那么ht\[1\]的大小为第一个大于等于 ht\[0\].used\*2 的 2的n次方幂

**收缩操作**

* 收缩启动条件
  * 哈希表的负载因子小于0.1时，自动收缩

如果是收缩操作，那么ht\[1\]的大小为第一个大于等于ht\[0\].used的 2的n次方幂

**渐进式rehash**

* 为h\[1\]分配空间
* 将rehashidx置为0，表示rehash正式开始
* 在rehash期间，每次对字典的更新，删除，查询，插入，程序除了执行指定操作，还会将ht\[0\].rehashidx索引上的所有键值对rehash到ht\[1\],当rehash完成后，rehashidx 加1
* 随着字典不断操作，rehash完成，rehashidx置为-1

渐进式rehash的好处是将工作分摊到每一个操作，从而避免集中处理带来的庞大计算量。

rehash期间不会对h\[0\]做插入，都插入到h\[1\]

### 跳跃表

有序集合的底层实现之一

集群节点内部数据结构

跳跃表节点

```c
typedef struct zskiplistNode{
  //层
  struct zskiplistLevel{
    struct zskiplistNode * forward;
    //跨度
    unsigned int span;
  }level[];
  struct zskiplistNode * backward;
  double score;
  robj obj;

}zskiplistNode;
```

跳跃表结构体

```c
typedef struct zskiplist{
  struct skiplistNode *header, *tail;
  unsigned long length;
  int level;
}zskiplist
```

### 整数集合intset

整数集合节点

```c
typedef struct intset{
  uint32_t encoding;
  uint32_t length;
  int8_t contents;
}
```

encoding三种类型

* INTSET\_ENC\_INT16
* INTSET\_ENC\_INT32
* INTSET\_ENC\_INT64

### 对象

```c
typedef struct redisObj{
  //类型
  unsigned type:4;
  //编码
  unsigned encoding:4;
  //指向底层实现数据结构的指针
  void *ptr;
}
```

#### 字符串对象

字符串对象的编码可以是int、raw或者embstr

1.如果保存整数值，并且可以用long来表示，那么字符串对象的编码设置为int。

2.如果保存字符串值，并且字符串长度32字节，那么字符串对象的编码设置为简单动态字符串embstr。

3.如果保存字符串值，且长度大于32字节，那么字符串对象的编码设置为简单动态字符串redisObject。

embstr编码的特别之处，embstr是一种优化的raw编码，优化点：将redisObject和sdshdr两个结构存到一起。

#### 列表对象

列表对象的编码可以是ziplist或者linkedlist

ziplist和linkedlist转换条件

1. 列表对象保存的所有字符串元素的长度都小于64个字节。
2. 列表对象保存的元素数量小于512。

以上两个条件可以修改

#### 哈希对象

哈希对象的编码可以是ziplist或者hashtable

ziplist和hashtable转换条件

1. 哈希对象保存的所有键值对的键和值得字符串长度都小于64字节。
2. 哈希对象保存的键值对数量小于512个；不能满足这两个条件的哈希对象需要使用hashtable编码

以上两个条件可以修改。

#### 集合对象

集合对象的编码可以是intset或者hashtable。

转换条件

1. 集合对象保存的所有元素都是整数值。
2. 集合对象保存的元素数量不超过512个。

第二个条件可以修改。

#### 有序集合对象

有序集合的编码可以是ziplist和skiplist

skiplist编码的有序集合对象使用zset结构作为底层实现，一个zset结构同时包含一个字典和一个跳跃表。

转换条件

1. 有序集合保存的元素数量小于128个；
2. 有序集合保存的所有元素成员的长度都小于64字节。

以上两个条件可以修改

## 类型检查和命令多态

Redis中用户操作键德尔命令基本上可以分为两种

1. 对任何键执行的命令 如del、expire、rename等
2. 另一种只能针对特定类型

### 类型检查

类型特定命令的类型检查是通过redisObject结构的type属性来实现的。

多态命令的实现。

### 内存回收

因为C语言并不具备自动内存回收功能，所以Redis在自己的对象系统中构建了一个引用计数技术实现内存回收。

```c
typedef struct redisObject{
  //引用计数
  int refcount;
}robj;
```

### 对象共享

多个建共享同一对象两个步骤

1. 将数据库键的值指针指向一个现有的对象
2. 将被共享的值对想的引用计数加一

Redis会在初始化服务器是，创建一万个字符串对象，这些对象包含了0-9999所有整形值。这些对象用来共享。

这个数量可以通过redis.h/REDIS\_SHARD\_INTEGERS常量来修改。

### 对象的空转时长

除了前面介绍的type、encoding、ptr和refcount四个属性外，redisObject结构包含的最后一个属性为lru属性，改属性记录了对象最后一个被命令访问的时间。

```c
typedef struct redisObject{
  unsigned lru;
}robj;
```

查看空转时长

```c
OBJECT IDLETIME
```

如果服务器的maxmemery选项被打开，切配置为volatile-lru或者alleys-lru时。那么当内存占用超过maxmemery，会回收空转时长较长的key

## 数据库

### 服务器中的数据库

Redis服务器将所有数据库都保存在服务器状态redis.h/redisServer结构的db数组中，db数组的每个项都是一个redis.h/redisDb结构。

```c
struct redisServer{
  redisDb *db;
}
```

初始化时候创建数据库的数量

```c
struct redisServer{
  int dbnum;
}
```

### 切换数据库

默认情况下，Rediso客户端的目标数据库为0号数据库，单客户端可以通过SELEC来切换数据库

```c
typedef struct redisClient{
  redisDb *db;
}redisClient;
```

### 数据库空间

```c
typedef struct redisDb{
  //数据库键空间，保存着数据库中的所有键值对。
  dict *dict;
}redisDb;
```

数据库的键空间是一个字典。

#### 读写键空间时的维护操作

1. 读取一个键之后，更新对应命中次数和不命中次数 ，这两个值可以通过status查看
2. 读取一个建之后，服务器会更新键的LRU时间
3. 如果客户端使用watch监视了某个键，那么服务器会将这个键标记为脏。
4. 键被修改，发送数据库通知

### 设置键的生存时间

通过EXPIRE或者PEXPIRE，客户端可以以秒或毫秒精度设置生存时间。TTL查看剩余时间。

#### 保存过期时间

redisDb结构的expires字典保存了数据库中所有键的过期时间，我们称这个字典为过期字典：

```c
typedef struct redisDb{
  dict *expires;

}redisDb;
```

* 过期字典的键是一个指针，这个指针指向键空间中的某个键对象。
* 过期字典的值是一个long long类型的整数，保存时间戳。

#### 过期删除策略

redis使用惰性删除和定期删除两种策略。

* 定期删除
  * 函数每次运行时，都从一定数量的数据库中取出一定数量的随机键进行检查，并删除
  * 全局变量current\_db记录进度。
  * 循环执行

### AOF、RDB和复制功能对过期键的处理

#### RDB文件

* 生成RDB文件

会检查过期情况，过期的不会被保存

* 载入RDB文件

主服务器载入会检查。从服务器不会检查。

### AOF文件

当过期键被惰性删除或者定期删除后，程序回想AOF文件追加DEL命令，来显示记录。

#### AOF重写

过期键不会被重写。

### 复制

在服务器运行在复制模式下，从服务器的过期键删除动作有主服务器控制：

1. 主服务器删除一个过期键，会显示想所有从服务发送DEL命令。
2. 从服务器读操作，碰到过期键不会删除，而会继续像处理未过期的键一样来出来过期键
3. 从服务器只有主服务器发来DEL命令之后，才会删除。

### 数据库通知

数据库通知是Redis 2.8版本新增的功能，这个功能可以通过订阅的方式来获取数据库的变化

## RDB持久化

todo

