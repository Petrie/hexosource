# 面试题

## 管理岗

* 技术能力
* 沟通能力
* 项目管理
* 人员管理
* 业务熟悉度

## 职业规划

* 职业规划

  一定数据量

  一定并发量

* 公司选择

  工程师文化

  技术态度开放

* 架构师细分

  系统架构师（System Architect）

  应用架构师（Application Architect）

  企业架构师（Enterprise Architect）

  基础设施架构师（Infrastructure Architect\)

* 架构师职责
  * 确认需求
  * 系统分解
  * 技术选型
  * 制定技术规范

## 技术面试题

**来源**：

* [https://www.toptal.com/php/interview-questions](https://www.toptal.com/php/interview-questions)
* 腾讯面试题
* [https://blog.csdn.net/s1070/article/details/51174655](https://blog.csdn.net/s1070/article/details/51174655)

**汇总**

1. php实现冒泡排序

   ```php
   function bubbleSort(&$arr)
   {
       $arrSize = count($arr);
       for ($i = 0; $i < $arrSize -1; $i++) {
           for ($j = 0; $j < $arrSize -1 - $i; $j++) {
               if ($arr[$j] > $arr[$j + 1]) {
                   $tmp = $arr[$j];
                   $arr[$j] = $arr[$j + 1];
                   $arr[$j+1] = $tmp;
               }
           }
       }
   }
   ```

2. php快速排序

   ```php
   function quickSort($array)
   {
       if(!isset($array[1]))
           return $array;
       $mid = $array[0]; //获取一个用于分割的关键字，一般是首个元素
       $leftArray = array();
       $rightArray = array();

       foreach($array as $v)
       {
           if($v > $mid)
               $rightArray[] = $v;  //把比$mid大的数放到一个数组里
           if($v < $mid)
               $leftArray[] = $v;   //把比$mid小的数放到另一个数组里
       }

       $leftArray = quickSort($leftArray); //把比较小的数组再一次进行分割
       $leftArray[] = $mid;        //把分割的元素加到小的数组后面，不能忘了它哦

       $rightArray = quickSort($rightArray);  //把比较大的数组再一次进行分割
       return array_merge($leftArray,$rightArray);  //组合两个结果
   }
   ```

3. 列举php语言结构，并说明它与php内置函数的区别

   php中语言结构有哪些 echo\(\) print\(\) die\(\) isset\(\) unset\(\) include\(\)，注意，include\_once\(\)是函数 require\(\)，注意，require\_once\(\)是函数 array\(\) list\(\) empty\(\) 。

   * 语言结构与函数的区别  
     1. 语言结构比对应功能的函数快  
     2. 语言结构在错误处理上比较鲁棒，由于是语言关键词，所以不具备再处理的环节  
     3. 语言结构不能在配置项\(php.ini\)中禁用，函数则可以。  
     4. 语言结构不能被用做回调函数

4. 写一个函数，能够遍历一个文件夹下的所有文件和子文件夹。

   ```php
   <?php
   function aGetAllFile($folder)
   {
       $aFileArr = array();
       if(is_dir($folder))
       {
           $handle=opendir($folder);
           while(($file=readdir($handle)) !== false)
           {
               //如果是.或者..则跳过
               if($file=="."||$file == "..")
               {
                   continue;
               }
               if(is_file($folder."/".$file))
               {
                   $aFileArr[]=$file;
               }
               else if(is_dir($folder."/".$file))
               {
                   $aFileArr[$file] = aGetAllFile($folder."/".$file);
               }
           }
           closedir($handle);
       }
       return $aFileArr;
   }
   $path = "/Users/liupengfei/it";
   print_r(aGetAllFile($path));
   ```

   ​

5. 算出两个文件的相对路径

   ```php
   <?php
   function getRelativePath($a, $b) {
       $returnPath = array(dirname($b));
       $arrA = explode('/', $a);
       $arrB = explode('/', $returnPath[0]);
       for ($n = 1, $len = count($arrB); $n < $len; $n++) {
           if ($arrA[$n] != $arrB[$n]) {
               break;
           }
       }
       if ($len - $n > 0) {
           $returnPath = array_merge($returnPath, array_fill(1, $len - $n, '..'));
       }

       $returnPath = array_merge($returnPath, array_slice($arrA, $n));
       return implode('/', $returnPath);
   }
   $a = '/a/b/c/d/e.php';
   $b = '/a/b/12/34/c.php';
   echo getRelativePath($a, $b);
   ```

   ```text
   /a/b/12/34/../../c/d/e.php
   ```

   ​

6. 冒泡排序

   ```php
   <?php
   function bubble_sort($array)
   {
       $count = count($array);
       if ($count <= 0) return false;

       for ($i = 0; $i < $count; $i++) {
           for ($j = $count - 1; $j > $i; $j--) {
               if ($array[$j] < $array[$j - 1]) {
                   $tmp = $array[$j];
                   $array[$j] = $array[$j - 1];
                   $array[$j - 1] = $tmp;
               }
           }
       }
       return $array;
   }

   var_dump(bubble_sort([1, 4, 6, 8, 3, 15, 7]));
   ```

   ​

7. 使用五种以上方式获取一个文件的扩展名

   ```php
   <?php
   function get_ext1($file_name){
       return strrchr($file_name, '.');
   }
   function get_ext2($file_name){
       return substr($file_name, strrpos($file_name, '.'));
   }
   function get_ext3($file_name){
       return array_pop(explode('.', $file_name));
   }
   function get_ext4($file_name){
       $p = pathinfo($file_name);
       return $p['extension'];
   }
   function get_ext5($file_name){
       return strrev(substr(strrev($file_name), 0, strpos(strrev($file_name), '.')));
   }
   ```

   ​

8. 写出如下程序的输出结果

   ```php
   <?php

   $GLOBALS['var1'] = 5;
   $var2 = 1;
   function get_value(){
       global $var2;
       $var1 = 0;
       return $var2++;
   }
   get_value();

   echo $var1;
   echo $var2;
   ```

   ```text
   52
   ```

   ​

9. 写出如下程序的输出结果

   ```php
   <?php
   $test = 'aaaaaa';
   $abc = & $test;
   unset($test);
   echo $abc;
   ```

   ```text
   aaaaaa
   ```

   ​

10. 以下的代码会产生什么？为什么？

    ```php
    <?php
    $num = 10;
    function testnum(){
       $num = $num * 10;
    }
    testnum();
    echo $num;
    ```

    ```text
    Notice: Undefined variable: num in /Users/liupengfei/workspace/test/test.php on line 4
    10
    ```

    ​

11. $a = 'abcdef'; 请取出$a的值并打印出第一个字母

    echo $a{0} 或者 echo $a\[0\] 或echo substr\($a,0,1\)

12. echo\(\),print\(\),print\_r\(\)的区别

    echo 和print不是一个函数，是一个语言结构 int print\(string $arg\), 只有一个参数 echo arg1,arg2; 可以输出多个参数，返回void echo和print只能打印出string，不能打印出结构 print\_r能打印出结构

13. 以下表达式的返回值。$a = array\(0\); empty\($a\); $a = array\(null\); empty\($a\);

    ```php
    ➜  shell php t.php
    bool(false)
    bool(false)
    ➜  shell cat t.php
    <?php
    $a = array(0);
    var_dump(empty($a));
    $a = array(null);
    var_dump(empty($a));
    ```

14. asort,arsort,sort区别
    * asort和arsort会对数组进行排序，并保持索引关系。arsort为逆序，asort为正序。
    * sort 排序不会保持索引关系。
15. 如下程序输出值

    ```php
    <?php
     $count = 5;
    function get_count(){
     static $count = 0;
     return $count++;
    }
    echo get_count();
    echo $count++;
    echo get_count();
    echo ++$count;
    ```

    ```text
    php t.php
    0517
    ```

16. 列举PHP中的语言结构，并说明他和PHP内置函数的区别

    [http://wayne173.iteye.com/blog/1766844](http://wayne173.iteye.com/blog/1766844)

    PHP中有一些类似函数的语言结构，但实际上它们不是函数；而是PHP语言本身的一种语言结构,或者说是关键字。

    比如：`echo(),print(),isset(),empty(),unset(),include(),require(),array(),list()`等。

    关键点：

    * 语言结构和PHP函数最根本的区别在于，PHP解析器会先将PHP函数分解成语言结构，而以上的那些本身就是PHP的语言结构。所以语言结构比相同功能的PHP函数运行更快。

17. php session必须调用session\_start才能使用吗？

    不是，php 配置文件里有 可以设置 session.auto\_start =1 这样就不需要调用session\_start\(\)，直接就能使用session

18. 一段程序，有全局变量，静态全局变量，类程序变量，函数内部成员变量，问这些变量存在内存代码段、数据段、堆、棧什么地方？
19. 不用加减乘除，一个数扩大7倍。
20. 自己实现itoa
21. 实现strlen
22. 单链表逆置 Node _rever\( Node_ header\) { 填写实现}
23. 字符串反转
24. 一个字符串按单词逆序,hello word输出world hello char _rever\(char_ s\) { 填写 }
25. 一个字符串中第1个只出现1次的字符，abcdad 输出
26. 实现一个二元组的镜像拷贝，利用循环和递归（大概，我是不懂）
27. 两条交叉链表，如何找到链表的交叉点。
28. 写一个php脚本,抓取页面[http://php.net/manual/en/langref.php右侧的目录列表](http://php.net/manual/en/langref.php右侧的目录列表)
29. 创建一个数据库php\_manual,新建表index,这个表有3个字段: id, title, link.然后创建一个数据库用户php\_manual\_user,密码是php\_manual\_pass.把上述数据库导出成sql,把SQL语句贴到下面,使得我们在mysql命令行终端里执行这些sql语句可以完成上述操作.

    ```sql
      create database php_manual;
      use php_manual;
      create table if not exists index(
          `id` int auto_increment comment '自增id',
            `title` varchar(300) NOT NULL defalt '' ,
          `link`  varchar(1000) NOT NULL default '', 
           primary key ('id')
      ) engine=innodb default charset=utf8;
      create user php_manual_user@'%' idenfitied by 'php_manual_pass';
    ```

14.写一个php脚本,读取第12题的结果langref.txt并解析出title和link,插入第13题创建的数据库表index里.

```php
$dbhost='127.0.0.1';
$port='3306';
$dbname='php_manual';
$user='php_manual_user';
$passwd='php_manual_pass';

$conn = mysqli($dbhos,$user,$passwd,$dbname,$port);
if($conn->connect_error){
  die('connect error');
}
$sql='';

if($conn->query($sql) == true){}else{}
$conn->close();
```

15.写一条shell命令,执行此命令可获取到[http://php.net/manual/en/langref.php的内容并将页面里的所有大写的PHP转成小写,最后将结果保存到/tmp/langref.html里](http://php.net/manual/en/langref.php的内容并将页面里的所有大写的PHP转成小写,最后将结果保存到/tmp/langref.html里).

```text
curl http://php.net/manual/en/langref.php | sed 's/\<php\>/PHP/' > /tmp/langref.html
```

16.改写下边的脚本,使得当接收到SIGINT信号时打印出"caught signal SIGINT, exit"并退出.

```php
<?php
    while (1) {
        echo "\n\n";
        echo "I am doing something important\n";
        echo "if i am interruptted, the data will be corrupted\n";
        echo "be careful\n";
        echo "\n\n";
        sleep(3);
    }
```

```php
<?php
    while (1) {
        echo "\n\n";
        echo "I am doing something important\n";
        echo "if i am interruptted, the data will be corrupted\n";
        echo "be careful\n";
        echo "\n\n";
        sleep(3);
    }
function sig_handler(){
  die("caught signal SIGINT, exit");
}
pcntl_signal(SIGINT, "sig_handler");
```

17.单引号和双引号区别？

* 单引号不会经过服务器翻译
* 双引号需要解析，效率会比较低

18.php session 默认存储位置

PHP session 保存在服务器的文件中，不是内存中，通过配置文件php.ini指定文件路径

```php
session.save_handler = files
session.save_path = "/tmp"
```

19.php主要错误类型和区别

* Notice 没有致命错误的提示，如访问一个不存在的变量
* Warning 比Notice更加重要的错误，比如include一个不存在的文件
* Fatal 致命错误，终止脚本执行，例如语法错误，访问对象中一个不存在的对象，或者require一个不存在的文件

20.php中如何启用错误提示

Check if “`display_errors`” is equal “on” in the **php.ini** or declare “`ini_set('display_errors', 1)`” in your script.Then, include “`error_reporting(E_ALL)`” in your code to display all types of error messages during the script execution.

21.[How does php evaluate the expression $x+++$x++?](https://stackoverflow.com/questions/38672551/how-does-php-evaluate-the-expression-xx)

```markup
1. get the value of x → 20
2. increment the value of x → 21
3. get the value of x → 21
4. increment the value of x → 22
5. sum the values got from 1 and 3 → 41
```

22.What is the problem with the code below? What will it output? How can it be fixed?

```php
$referenceTable = array();
$referenceTable['val1'] = array(1, 2);
$referenceTable['val2'] = 3;
$referenceTable['val3'] = array(4, 5);

$testArray = array();

$testArray = array_merge($testArray, $referenceTable['val1']);
var_dump($testArray);
$testArray = array_merge($testArray, $referenceTable['val2']);
var_dump($testArray);
$testArray = array_merge($testArray, $referenceTable['val3']);
var_dump($testArray);
```

The output will be as follows:

```text
array(2) { [0]=> int(1) [1]=> int(2) }
NULL
NULL
```

The issue here is that, if _either_ the first or second argument to `array_merge()` is not an array, the return value will be `NULL`.

23.What will this code output and why?

```php
$x = true and false;
var_dump($x);
```

答案是true

The issue here is that the `=` operator takes precedence over the `and` operator in order of operations

24.以下输出

```php
var_dump(PHP_INT_MAX + 1);
var_dump((int)(PHP_INT_MAX + 1))
```

结果是：

double\(9.2233720368548E+18\)

int\(-9223372036854775808\)

