# PHP

### 引用变量

### 字符串

* heredoc
* newdoc

### empty vs isset vs is\_null

```text
https://www.virendrachandak.com/techtalk/php-isset-vs-empty-vs-is_null/
```

### 常量

* PHP 5.3.0 起，有两种方式定义常量，使用 const 关键字或者 define\(\) 函数 

### Switch

* case类型只能是 整形、浮点类型、字符串

### 扩展开发

> [http://www.imsiren.com/archives/196](http://www.imsiren.com/archives/196)
>
> [http://www.cunmou.com/phpbook/preface.md](http://www.cunmou.com/phpbook/preface.md)
>
> [https://devzone.zend.com/1435/wrapping-c-classes-in-a-php-extension/](https://devzone.zend.com/1435/wrapping-c-classes-in-a-php-extension/)
>
> [http://0x1.im/blog/php/how-to-create-a-php-extension.html](http://0x1.im/blog/php/how-to-create-a-php-extension.html)
>
> [https://docstore.mik.ua/orelly/webprog/php/ch14\_08.htm](https://docstore.mik.ua/orelly/webprog/php/ch14_08.htm)

### 协程

> 《GO语言编程》
>
> [http://www.laruence.com/2015/05/28/3038.html](http://www.laruence.com/2015/05/28/3038.html)

协程的支持是在迭代生成器的基础上, 增加了可以回送数据给生成器的功能\(调用者发送数据给被调用的生成器函数\). 这就把生成器到调用者的单向通信转变为两者之间的双向通信.

协作多任务在Windows的早期版本\(windows95\)和Mac OS中有使用, 不过它们后来都切换到使用抢先多任务了. 理由相当明确：如果你依靠程序自动交出控制的话, 那么一些恶意的程序将很容易占用整个CPU, 不与其他任务共享.

#### 协程任务调度

#### 协程堆栈

#### 错误处理

### 同步阻塞 IO、同步非阻塞 IO、多路复用IO 、 异步 IO

> [http://blog.csdn.net/HQ354974212/article/details/76155906](http://blog.csdn.net/HQ354974212/article/details/76155906)

* 同步（synchronous）和异步（asynchronous）的概念描述的是用户线程与内核的交互方式：同步是指用户线程发起 IO 请求后需要等待或者轮询内核 IO 操作完成后才能继续执行；而异步是指用户线程发起 IO 请求后仍继续执行，当内核 IO 操作完成后会通知用户线程，或者调用用户线程注册的回调函数。
* 阻塞（blocking）和非阻塞（non-blocking）的概念描述的是用户线程调用内核 IO 的操作方式：阻塞是指 IO 操作需要彻底完成后才返回到用户空间；而非阻塞是指 IO 操作被调用后立即返回给用户一个状态值，无需等到 IO 操作彻底完成。
* 各种IO特点
  * 同步阻塞IO：调用recv函数时，系统首先查是否有准备好的数据。如果数据没有准备好，那么系统就处于等待状态。应对多客户机的网络应用，通过多进程、多线程解决。fork／pthread\_create
  * 非阻塞IO：把一个socket接口设置为非阻塞，就是告诉内核当I/O操作无法完成时，不要进入睡眠，而是返回一个错误
  * IO复用模型：同时对多个I/O端口进行侦听。
* 各种IO特点和优缺点
  * 阻塞blocking IO 的特点就是在 IO 执行的两个阶段（等待数据和拷贝数据两个阶段）都被 block 了。
  * 在非阻塞式 IO 中，用户进程其实是需要不断的主动询问 kernel 数据准备好了没有。

由于 select 函数是阻塞的，因此多路 IO 复用模型也被称为异步阻塞 IO 模型。

### php常用扩展

* xhprof  PHP性能分析工具
* xdebug  PHP调试工具

### php生命周期

> [http://www.php-internals.com/](http://www.php-internals.com/)

#### 两个阶段

1. 处理请求之前的开始阶段
   1. 模块初始化阶段 MINIT
   2. 模块激活阶段 RINIT
2. 请求之后的结束阶段
   1. 停用模块RSHUTDOWN
   2. 关闭模块MSHUTDOWN

CLI/CGI模式的PHP属于单进程的SAPI模式。这类的请求在处理一次请求后就关闭。也就是只会经过如下几个环节： 开始 - 请求开始 - 请求关闭 - 结束 SAPI接口实现就完成了其生命周期。

### php配置文件

#### php.ini

* short\_open\_tag
* asp\_tags
* precision
* disable\_functions
* disable\_classes
* memory\_limit
* realpath\_cache\_size
* realpath\_cache\_ttl
* post\_max\_size
* default\_mimetype
* default\_charset
* extension\_dir
* extension
* zend\_extension

#### php-fpm.conf

> [http://php.net/manual/zh/install.fpm.configuration.php](http://php.net/manual/zh/install.fpm.configuration.php)
>
> [https://myjeeva.com/php-fpm-configuration-101.html](https://myjeeva.com/php-fpm-configuration-101.html)

* process\_control\_timeout
* events.mechanism = epoll
* pm.max\_requests = 10000
* request\_slowlog\_timeout = 5s
* slowlog = /var/log/$pool.log.slow
* request\_terminate\_timeout

> [https://laravel-china.github.io/php-the-right-way/](https://laravel-china.github.io/php-the-right-way/)

### 字符串处理函数

> [http://php.net/manual/zh/ref.strings.php](http://php.net/manual/zh/ref.strings.php)

### 数组处理函数

> [http://php.net/manual/zh/function.array.php](http://php.net/manual/zh/function.array.php)

* array\_shift 将数组开头数组移出
* array\_unshift 在数组开头插入一个或个多个单元
* array\_change\_key\_case 将所有key大小写转换
* array\_chunk 将数组分片
* array\_column 取出二维数组指定列
* array\_combine 创建一个数组，用一个数字的键作为其键名，另一个数组的值作为其值
* array\_count\_values 统计数组中所有的值
* array\_diff\_assoc 所有在array1但不在其他所有数组的元素，带索引取差集
* array\_diff\_key 所有在array2但不在其他所有数组的元素，带key取差集
* array\_diff\_uassoc 回调函数比较异同
* array\_diff\_ukey 回调函数比较异同
* end 将 `array` 的内部指针移动到最后一个单元并返回其值。

### error\_reporting\(\)

* E\_NOTICE 通知信息会对代码中可能出现的bug给出警告，例如，_$arr\[item\]_ 最好写成 _$arr\['item'\]_ ，因为 PHP 会试图将 _"item"_ 当成一个常量。如果它不是一个常量，PHP才会把它当做数组的字符串索引。
* E\_STRICT 在开发阶段启用 **E\_STRICT** 会有一些好处。严格的信息将帮助你使用最新和最好的建议的方法来编写代码，例如它会警告你使用了将被废弃的函数。
* todo 更加详细的功能描述

> [http://php.net/manual/zh/errorfunc.configuration.php\#ini.error-reporting](http://php.net/manual/zh/errorfunc.configuration.php#ini.error-reporting)

### 编码规范psr

* PSR-1：基础代码风格
* PSR-2：严格代码风格
* PSR-3：日志记录器接口
* PSR-4：自动加载

### 文件读写

### preg\_match

### pdo

> [http://wiki.hashphp.org/PDO\_Tutorial\_for\_MySQL\_Developers](http://wiki.hashphp.org/PDO_Tutorial_for_MySQL_Developers)

* 创建连接

  ```php
  <?php 
  $db = new PDO('mysql:host=localhost;dbname=testdb;charset=utf8','username','password');
  ```

* setAttribute

  ```php
  <?php
  $db = new PDO('mysql:host=localhost;dbname=testdb;charset=utf8mb4', 'username', 'password');
  $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
  $db->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
  ```

* Error Handing

  PDO has three handling modes.

  1. PDO::ERRMODE_SILENT acts like mysql_\* where you must check each result and then look at $db-&gt;errorInfo\(\); to get the error details
  2. PDO::_ERRMODE\_WARNING_ throws php Warning
  3. PDO::ERRMODE\_EXCEPTION throws PDOEXception. In my opinion this is the mode you should use.it acts very much like `or die(mysql_error())` when it is;t caught.

  ```php
  <?php
  function getData($db){
    $stmt = $db->query('SELECT * FROM table');
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
  }

  try{
    getData($db);
  }catch(PDOException $ex){
    //handle me
  }
  ```

* Running Simple Select Statements

  ```php
  <?php
  foreach($db->query('SELECT * FROM table') as $row){
    echo $row['field'].' '.$row['field'];
  }
  ```

  ```php
  <?php
  $stmt  = $db->query('SELECT * FROM table');
  while($row = $stmt->fecth(PDO::FETCH_ASSOC)){
    echo $row['field'].' '.$row['field2'];
  }
  ```

  or

  ```php
  <?php
  $stmt = $db->query('SELECT * FROM table');
  $results = $stmt->fecthALL(PDO::FETCH_ASSOC);
  ```

* Getting Row Count

  ```php
  <?php
  $stmt = $db->query('SELECT * FROM table');
  $row_count = $stmt->rowCount();
  ```

* Getting the last Insert Id

  ```php
  <?php
   $result = $db->exec("INSERT INTO table(firstname, lastname) values('John','Doe')");
  $insertId = $db->lastInsertId();
  ```

* Running Simple INSERT,UPDATA, or DELETA statements

  ```php
  <?php 
   $affected_rows = $db->exec("UPDATE table set field = 'value'");
  ```

* Running Statements With Parameters

  ```php
  $stmt = $db->prepare("SELECT * FROM table WHERE id=? AND name=?");
  $stmt->execute(array($id,$name));
  $rows = $stmt->fetchAll(PDO::FECTH_ASSOC);
  ```

* bindValue

  ```php
  <?php
  $stmt = $db->prepare("SELECT * FROM table where id=? and name=?");
  $stmt->bindValue(1,$id,PDO::PARAM_INT)
  $stmt->bindValue(2,$name,PDO::PARAM_STR)
  $stmt->execute();
  $rows = $stmt->fecthAll(PDO::FETCH_ASSOC)
  ```

* named Placeholders

  ```php
  <?php
    $stmt = $db->prepare("SELECT * FROM table WHERE id=:id AND name=:name");
    $stmt->bindValue(':id',$id,PDO::PARAM_INT);
    $stmt->bindValue(':name',$name,PDO::PARAM_STR);
    $stmt->execute();
    $rows = $stmt->fecthAll(PDO::FETCH_ASSOC);
  ```

* INSERT,DELETE UPDATE Prepared Queries

  ```php
  <?php
    #insert
  $stmt = $db->prepare("INSERT INTO table(field1,field2,field3,field4,field5) VALUES(:field1,:field2,:field3,:field4,:field5)");
  $stmt->execute(array(':field1' => $field1, ':field2' => $field2, ':field3' => $field3, ':field4' => $field4, ':field5' => $field5));
  $affected_rows = $stmt->rowCount();
  #delete
  $stmt = $db->prepare("DELETE FROM table WHERE id=:id");
  $stmt->bindValue(':id', $id, PDO::PARAM_STR);
  $stmt->execute();
  $affected_rows = $stmt->rowCount();
  #update
  $stmt = $db->prepare("UPDATE table SET name=? WHERE id=?");
  $stmt->execute(array($name, $id));
  $affected_rows = $stmt->rowCount();
  ```

### mysqli

* 面向对象链接数据库

  ```php
  <?php
  $mysqli = new mysqli('localhost', 'my_user', 'my_password', 'my_db');

  /*
  * This is the "official" OO way to do it,
  * BUT $connect_error was broken until PHP 5.2.9 and 5.3.0.
  */
  if ($mysqli->connect_error) {
     die('Connect Error (' . $mysqli->connect_errno . ') '
             . $mysqli->connect_error);
  }
  ```

  /\*

* Use this instead of $connect\_error if you need to ensure
* compatibility with PHP versions prior to 5.2.9 and 5.3.0.

  ```text
              */
             if (mysqli_connect_error()) {
                 die('Connect Error (' . mysqli_connect_errno() . ') '
                         . mysqli_connect_error());
             }

             echo 'Success... ' . $mysqli->host_info . "\n";

             $mysqli->close();
  ```

  ```text

  ```

* 面向过程链接数据库

  ```php
  <?php
  $link = mysqli_connect('localhost', 'my_user', 'my_password', 'my_db');

  if (!$link) {
      die('Connect Error (' . mysqli_connect_errno() . ') '
              . mysqli_connect_error());
  }

  echo 'Success... ' . mysqli_get_host_info($link) . "\n";

  mysqli_close($link);
  ?>
  ```

  ​

### curl

#### curl\_setopt\(\)

CURLOPT\_RETURNTRANSFER Return the response as a string instead of outputting to the screen

CURLOPT\_CONNECTTIMEOUT Number of second to attempting to connect

CURLOPT\_TIMEOUT Number of seconds all cUrl to execute

CURLOPT\_USERAGENT Useragent to use for request

CURLOPT\_URL URL to send request to

CURLOPT\_POST send request as post

CURLOPT\_POSTFIELDS Array of data to POST in request

#### GET request

```php
$curl = curl_init();
curl_setopt_array($curl, array(
    CURLOPT_RETURNTRANSFER => 1,
    CURLOPT_URL => 'http://google.com',
     CURLOPT_USERAGENT => 'xxxx',
));
$resp = curl_exec($curl);
curl_close($curl);
```

#### POST request

```php
$curl = curl_init();
curl_setopt_array($curl, array(
    CURLOPT_RETURNTRANSFER => 1,
      CURLOPT_URL => 'http://google.com',
    CURLOPT_USERAGENT => 'xxx',
    CURLOPT_POST=> 1,
     CURLOPT_POSTFIELD => array(
        item1 => 'value',
          item2 => 'value2',
    ),
));
$resp = curl_exec($curl);
curl_close($curl);
```

* curl\_error\(\)
* curl\_errno\(\)

### curl\_multi

## PHP框架

如何选择框架

* 学习曲线
* 可扩展性
* 维护团队是否活跃
* LTS 支持
* 社区是否活跃

### Laravel vs Yii vs Symfony

* 模版引擎
  * Symfony 用Twig
  * laravel 用 blade
  * Yii没有默认的模版引擎
* 模块化
  * laravel 没有模块化的功能
  * symfony和yii都有
* 性能
* db支持
* 可扩展性
  * laravel是赢家

#### laravel

laravel最佳实践

[https://github.com/alexeymezenin/laravel-best-practices\#business-logic-should-be-in-service-class](https://github.com/alexeymezenin/laravel-best-practices#business-logic-should-be-in-service-class)

**laravel特点**

* 引入大量成熟的开源库
* 拥抱php:命名空间、接口、匿名方法
* Eloquent 整合
* artisan

#### Symfony

[https://www.chrisyue.com/translation-create-your-own-framework-on-top-of-the-symfony-components-index.html](https://www.chrisyue.com/translation-create-your-own-framework-on-top-of-the-symfony-components-index.html)

### PHP Composer

* 安装

  ```php
  curl -sS https://getcomposer.org/installer | php
  mv composer.phar /usr/local/bin/composer        
  composer -V
  ```

* 创建一个`composer.json`文件，写入相应的包名和版本号

  ```text
  {
  "require":     {
  "monolog/monolog": "1.0.*"
              }
  }
  ```

* 执行 `composer install`，就进入自动安装，安装完成后会生成一个`composer.lock`文件，里面是特定的版本号名，需要这个文件和`composer.json`一起提交到版本管理里去。
* 自动加载

  ```php
  require 'vendor/autoload.php';
  ```

### 单元测试

### 设计模式

#### 单例模式

php 单例模式禁用clone方法，通过私有化\_\_clone函数

php 单例模式禁用construct方法

> [http://design-patterns.readthedocs.io/zh\_CN/latest/index.html](http://design-patterns.readthedocs.io/zh_CN/latest/index.html)
>
> [https://laravel-china.github.io/php-the-right-way/pages/Design-Patterns.html](https://laravel-china.github.io/php-the-right-way/pages/Design-Patterns.html)

### 内存管理

> [https://secure.php.net/manual/en/internals2.memory.php](https://secure.php.net/manual/en/internals2.memory.php)
>
> [http://g.oswego.edu/dl/html/malloc.html](http://g.oswego.edu/dl/html/malloc.html)
>
> [http://www.cnblogs.com/taek/p/4229869.html](http://www.cnblogs.com/taek/p/4229869.html)
>
> [https://www.liuyingji.me/archives/142/](https://www.liuyingji.me/archives/142/)
>
> [https://phpinternals.net/docs/php\_printf](https://phpinternals.net/docs/php_printf)​

## php7

### VM

> [http://nikic.github.io/2017/04/14/PHP-7-Virtual-machine.html](http://nikic.github.io/2017/04/14/PHP-7-Virtual-machine.html)
>
> [https://gywbd.github.io/posts/2016/2/zend-execution-engine.html](https://gywbd.github.io/posts/2016/2/zend-execution-engine.html)
>
> [http://jpauli.github.io/2015/02/05/zend-vm-executor.html](http://jpauli.github.io/2015/02/05/zend-vm-executor.html)
>
> [http://www.laruence.com/2008/08/11/147.html](http://www.laruence.com/2008/08/11/147.html)

### AST\(抽象语法树\)

> [http://0x1.im/blog/php/changes-of-php7-because-of-ast.html](http://0x1.im/blog/php/changes-of-php7-because-of-ast.html)

PHP7 的内核中有一个重要的变化是加入了 AST。在 PHP5中，从 php 脚本到 opcodes 的执行的过程是：

1. Lexing：词法扫描分析，将源文件转换成 token 流；
2. Parsing：语法分析，在此阶段生成 op arrays。

PHP7 中在语法分析阶段不再直接生成 op arrays，而是先生成 AST，所以过程多了一步：

1. Lexing：词法扫描分析，将源文件转换成 token 流；
2. Parsing：语法分析，从 token 流生成抽象语法树；
3. Compilation：从抽象语法树生成 op arrays。

### 源码解析

#### zval

> [https://nikic.github.io/2015/05/05/Internal-value-representation-in-PHP-7-part-1.html](https://nikic.github.io/2015/05/05/Internal-value-representation-in-PHP-7-part-1.html)
>
> [http://0x1.im/blog/php/Internal-value-representation-in-PHP-7-part-1.html](http://0x1.im/blog/php/Internal-value-representation-in-PHP-7-part-1.html)
>
> [http://0x1.im/blog/php/Internal-value-representation-in-PHP-7-part-2.html](http://0x1.im/blog/php/Internal-value-representation-in-PHP-7-part-2.html)

* zval in php5

  ```c
  typedef union _zvalue_value {
      long lval;                 // For booleans, integers and resources
      double dval;               // For floating point numbers
      struct {                   // For strings
          char *val;
          int len;
      } str;
      HashTable *ht;             // For arrays
      zend_object_value obj;     // For objects
      zend_ast *ast;             // For constant expressions
  } zvalue_value;

  typedef struct _zval_struct {
      zvalue_value value;
      zend_uint refcount__gc;
      zend_uchar type;
      zend_uchar is_ref__gc;
  } zval;
  /*In order to support this additional cycle collector, the actually used zval structure is the following:*/
  typedef struct _zval_gc_info {
      zval z;
      union {
          gc_root_buffer       *buffered;
          struct _zval_gc_info *next;
      } u;
  } zval_gc_info;
  ```

* PHP5 中 zval 实现方式存在的主要问题：

  * zval 总是需要从堆中分配内存；
  * zval 总是存储引用计数和循环回收的信息，即使是整型这种可能并不需要此类信息的数据；
  * 在使用对象或者资源时，直接引用会导致两次计数（原因会在下一部分讲）；
  * 某些间接访问需要一个更好的处理方式。比如现在访问存储在变量中的对象间接使用了四个指针（指针链的长度为四）。这个问题也放到下一部分讨论；
  * 直接计数也就意味着数值只能在 zval 之间共享。如果想在 zval 和 hashtable key 之间共享一个字符串就不行（除非 hashtable key 也是 zval）。

  **修改动机**

  这里说的都是指在 64 位的系统上。首先，由于 `str` 和 `obj` 占用的大小一样， `zvalue_value` 这个联合体占用 16 个字节（bytes）的内存。整个 `zval` 结构体占用的内存是 24 个字节（考虑到内存对齐），`zval_gc_info` 的大小是 32 个字节。综上，在堆（相对于栈）分配给 zval 的内存需要额外的 16 个字节，所以每个 zval 在不同的地方一共需要用到 48 个字节（要理解上面的计算方式需要注意每个指针在 64 位的系统上也需要占用 8 个字节）。

* zval in php7

  ```c
  typedef union _zend_value {
      zend_long         lval;
      double            dval;
      zend_refcounted  *counted;
      zend_string      *str;
      zend_array       *arr;
      zend_object      *obj;
      zend_resource    *res;
      zend_reference   *ref;
      zend_ast_ref     *ast;

      // Ignore these for now, they are special
      zval             *zv;
      void             *ptr;
      zend_class_entry *ce;
      zend_function    *func;
      struct {
          ZEND_ENDIAN_LOHI(
              uint32_t w1,
              uint32_t w2)
      } ww;
  } zend_value;
  struct _zval_struct {
      zend_value value;
      union {
          struct {
              ZEND_ENDIAN_LOHI_4(
                  zend_uchar type,
                  zend_uchar type_flags,
                  zend_uchar const_flags,
                  zend_uchar reserved)
          } v;
          uint32_t type_info;
      } u1;
      union {
          uint32_t var_flags;
          uint32_t next;                 // hash collision chain
          uint32_t cache_slot;           // literal cache slot
          uint32_t lineno;               // line number (for ast nodes)
          uint32_t num_args;             // arguments number for EX(This)
          uint32_t fe_pos;               // foreach position
          uint32_t fe_iter_idx;          // foreach iterator index
      } u2;
  };
  ```

  * zval不再总是堆分配，不再存储引用计数
  * 标量类型不在堆分配，不再引用计数
  * 移除双重引用计数
  * 引用计数存在值本身，值可以共享。如：字符串可以被zval和hashtable共享

#### 引用

这个设计的一个很大的问题在于它无法在一个 PHP 引用变量和 PHP 非引用变量之间共享同一个值。比如下面这种情况：

```php
<?php
$a = [];  // $a         -> zval_1(type=IS_ARRAY, refcount=1, is_ref=0) -> HashTable_1(value=[])
$b = $a;  // $a, $b     -> zval_1(type=IS_ARRAY, refcount=2, is_ref=0) -> HashTable_1(value=[])
$c = $b   // $a, $b, $c -> zval_1(type=IS_ARRAY, refcount=3, is_ref=0) -> HashTable_1(value=[])

$d =& $c; // $a, $b -> zval_1(type=IS_ARRAY, refcount=2, is_ref=0) -> HashTable_1(value=[])
          // $c, $d -> zval_1(type=IS_ARRAY, refcount=2, is_ref=1) -> HashTable_2(value=[])
          // $d 是 $c 的引用, 但却不是 $a 的 $b, 所以这里 zval 还是需要进行复制
          // 这样我们就有了两个 zval, 一个 is_ref 的值是 0, 一个 is_ref 的值是 1.

$d[] = 1; // $a, $b -> zval_1(type=IS_ARRAY, refcount=2, is_ref=0) -> HashTable_1(value=[])
          // $c, $d -> zval_1(type=IS_ARRAY, refcount=2, is_ref=1) -> HashTable_2(value=[1])
          // 因为有两个分离了的 zval, $d[] = 1 的语句就不会修改 $a 和 $b 的值.
```

这种行为方式也导致在 PHP 中使用引用比普通的值要慢。比如下面这个例子：

```php
<?php
$array = range(0, 1000000);
$ref =& $array;
var_dump(count($array)); // <-- 这里会进行分离
```

因为 `count()` 只接受传值调用，但是 `$array` 是一个 PHP 引用，所以 `count()` 在执行之前实际上会有一个对数组进行完整的复制的过程。如果 `$array` 不是引用，这种情况就不会发生了。

#### 复杂类型公共头

```c
struct _zend_refcounted {
    uint32_t refcount;
    union {
        struct {
            ZEND_ENDIAN_LOHI_3(
                zend_uchar    type,
                zend_uchar    flags,
                uint16_t      gc_info)
        } v;
        uint32_t type_info;
    } u;
};
```

#### 字符串

```c
struct _zend_string {
    zend_refcounted   gc;
    zend_ulong        h;        /* hash value */
    size_t            len;
    char              val[1];
};
```

#### 对象

#### hashtable

> [http://nikic.github.io/2014/12/22/PHPs-new-hashtable-implementation.html](http://nikic.github.io/2014/12/22/PHPs-new-hashtable-implementation.html)

* php5中hash table  效率低的原因
  * hash表元素需要单独堆分配，额外需要8/16字节的内存，需要更多额外内存且降低缓存命中
  * zval需要单独分配内存，
  * 双向链表需要额外的四个指针，因此索引链表十分缓存不友好
* php7中的优势
  * 简单类型的zval不再需要单独的内存分配，节约内存并且缓存友好
  * zval存储简单类型不需要存储refcount和gc root buffer
  * 复杂类型嵌入refcount，他们可以被zval共享。比如字符串可以共享，这对hashtable
* php5和php7内存管理区别
  * zval不再一致单独分配，节约16字节
  * hash元素不再单独分配，节约16个字节
  * 标量类型只占用16个字节 
  * hash元素默认就是有序的，不再需要双向链表
  * 冲突处理简单的连接，通过zval内嵌的字段
  * zval嵌入hash元素，不再需要指针
  * 压缩数组

### php7新特性

* 标量类型声明

  强制模式、严格模式

* 返回类型声明

  同样也分为强制模式和严格模式

* null合并运算符
* 太空船操作（组合比较符）
* 通过define 定义常量数组

  ```php
  define('ANIMALS', [
      'dog',
      'cat',
      'bird'
  ]);

  echo ANIMALS[1]; // outputs "cat"
  ```

* 匿名类
* Group use decalrations

  从同一 namespace 导入的类、函数和常量现在可以通过单个 use 语句 一次性导入了。

* 迭代器新增return方法
* 迭代器组合
* intdiv对除法结果取整

### php7性能优化

* 记得启用Zend Opcache, 因为PHP7即使不启用Opcache速度也比PHP-5.6启用了Opcache快, 所以之前测试时期就发生了有人一直没有启用Opcache的事情.
* 使用新的编译器
* Hugepage
* Opcache file cache
* PGO

### php7升级注意事项

* mysql\_escape\_string 随之mysql扩展被移除
* HTTP\_RAW\_POST\_DATA -&gt; php://input

