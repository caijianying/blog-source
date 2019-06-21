---
title: Shell学习
date: 2016-07-10 10:36:52
tags: 开发笔记
categories:
- shell
---
# Shell学习
## 1.流程控制
- for
使用格式如下：
```
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```
其中需要循环的数据用空格分离开
- while
```
int=1
while(( $int<=5 ))
do
        echo $int
        let "int++"
done
```
- if/else
以下实例判断两个变量是否相等：
```
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```
## 2.数组
- 数组定义
```
array_name=(value1 ... valuen)
```
数组用括号来表示，元素用"空格"符号分割开
- 读取数组
```
${array_name[index]}
```
- 数组长度
```
${#my_array[*]}
```
- 遍历数组
```
arr=(1 2 3 4 5)
for var in ${arr[@]};
do
    echo $var
done
```
循环输出
```
for j in $(seq 7000 7005)
do
  echo $j
done

```
## 3.函数
- 函数传参
```
#/bin/sh
function addslots()
{
for i in `seq  $2 $3`;do
 src/redis-cli  -p $1 cluster addslots $i  
done
}
case $1 in
    addslots)
    echo "add slots ..."
    addslots $2 $3 $4;
  ;;
  *)
    echo "Usage: inetpanel [addslots]"
  ;;
esac
```
## 4.$0,$?,$!等的特殊用法
变量|说明
---|---
$$|Shell本身的PID（ProcessID）
$!|Shell最后运行的后台Process的PID
$?|最后运行的命令的结束代码（返回值）
$-|使用Set命令设定的Flag一览
$*|所有参数列表。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
$@|所有参数列表。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。
$#|添加到Shell的参数个数
$0|Shell本身的文件名
$1～$n|添加到Shell的各参数值。$1是第1参数、$2是第2参数…

## 5.变量判断
- -n
可以使用-n来判断一个string不是NULL值
```
if [ ! -f "/bin/redis-cli" ]; then 
   ln -s $LOGSTASH_HOME/bin/redis-cli /bin/redis-cli
fi 
```
- -f
可以使用-f来判断一个文件是否存在
```
PID=`cat logs/$confName.pid`
    if [ -n "$PID" ];then
       kill -9 $PID
       rm -f logs/$confName.pid
    fi
```
- 更多
```
[ -f "$file" ] 判断$file是否是一个文件

[ $a -lt 3 ] 判断$a的值是否小于3，同样-gt和-le分别表示大于或小于等于

[ -x "$file" ] 判断$file是否存在且有可执行权限，同样-r测试文件可读性

[ -n "$a" ] 判断变量$a是否有值(空)，测试空串用-z

[ -n $a ] 判断变量$a是否为null

[ "$a" = "$b" ] 判断$a和$b的取值是否相等

[ cond1 -a cond2 ] 判断cond1和cond2是否同时成立，-o表示cond1和cond2有一成立
```
## 6.写文件
- 两种方式

两种方式：>重写>>追加
- 写log
```
shell上:
0表示标准输入
1表示标准输出
2表示标准错误输出
> 默认为标准输出重定向，与 1> 相同
2>&1 意思是把 标准错误输出 重定向到 标准输出.
&>file 意思是把 标准输出 和 标准错误输出 都重定向到文件file中
```