# Shell

## shell

* for循环

```text
#!/bin/bash
# this program counts from 1 to 10:
for i in 1 2 3 4 5 6 7 8 9 10; do
    echo $i
done
```

```text
#!/bin/bash
for file in *; do
    echo "Adding .html extension to $file..."
    mv $file $file.html
    sleep 1
done
```

```text
for((i=1;i<=10;i++));do echo $(expr $i \* 4);done
```

* if语句

```text
#!/bin/bash
if test -f /etc/foo
then
    # file exists, so copy and print a message.
    cp /etc/foo .
    echo "Done."
else
    # file does NOT exist, so we print a message and exit.
    echo "This file does not exist."
    exit
fi
```

* while语句

```text
#!/bin/bash
while true; do
   echo "Press CTRL-C to quit."
done
```

```text
#!/bin/bash
x=0;     # initialize x to 0
while [ "$x" -le 10 ]; do
    echo "Current value of x: $x"
    # increment the value of x:
    x=$(expr $x + 1)
    sleep 1
done
```

* switch语句

```text
#!/bin/bash
x=5     # initialize x to 5
# now check the value of x:
case $x in
   0) echo "Value of x is 0."
      ;;
   5) echo "Value of x is 5."
      ;;
   9) echo "Value of x is 9."
      ;;
   *) echo "Unrecognized value."
esac
```

```text
#!/bin/bash
x=5     # initialize x to 5
if [ "$x" -eq 0 ]; then
    echo "Value of x is 0."
elif [ "$x" -eq 5 ]; then
    echo "Value of x is 5."
elif [ "$x" -eq 9 ]; then
    echo "Value of x is 9."
else
    echo "Unrecognized value."
fi
```

## sed

* 普通替换

```text
$ sed 's/Linux/Linux-Unix/' thegeekstuff.txt
```

* 全局替换

```text
$ sed 's/Linux/Linux-Unix/g' thegeekstuff.txt
```

* 替换前两个

```text
$ sed 's/Linux/Linux-Unix/2' thegeekstuff.txt
```

* 替换并将diff输出到output 

```text
$ sed -n 's/Linux/Linux-Unix/gpw output' thegeekstuff.txt
```

* 匹配成功再使用正则

```text
$ sed '/\-/s/\-.*//g' thegeekstuff.txt
```

* 删除最后几个字符

```text
$ sed 's/...$//' thegeekstuff.txt
```

* 删除空行和注释

```text
$ sed -e 's/#.*//;/^$/d'  thegeekstuff.txt
```

* 删除html元素

```text
$ sed -e 's/<[^>]*>//g'
```

* 第一行

```text
$ sed -n '1w output.txt' thegeekstuff.txt
```

* 第一行和最后一行

```text
$ sed -n -e '1w output.txt' -e '$w output.txt' thegeekstuff.txt
```

* 匹配行

```text
$ sed -n -e '/Storage/w output.txt' -e '/Sysadmin/w output.txt' thegeekstuff.txt
```

* 删除第四行和第二行

```text
$ sed -e '4d' -e '2d' thegeekstuff.txt
```

* 全局引用&

```text
$ sed 's@/usr/bin@&/local@g' path.txt
```

* 局部引用

```text
$ sed '$s@\([^:]*\):\([^:]*\):\([^:]*\)@\3:\2:\1@g' path.txt
```

## awk

* 打印所有行

```text
$ awk '{print;}' employee.txt
```

* 表格标题打印

```text
$ awk 'BEGIN {print "Name\tDesignation\tDepartment\tSalary";}
> {print $2,"\t",$3,"\t",$4,"\t",$NF;}
> END{print "Report Generated\n--------------";
> }' employee.txt
```

* 指定列

```text
$ awk '$1 >200' employee.txt
```

* 指定列正则

```text
$ awk '$4 ~/Technology/' employee.txt
```

* begin

```text
$ awk 'BEGIN { count=0;}
$4 ~ /Technology/ { count++; }
END { print "Number of employees in Technology Dept =",count;}' employee.txt
```

