---
type: article
title:  "Linux 基本命令"
categories: Linux
tags: Linux
author: DengYuting
---

本文包含 sort、wc、ps、awk、sed命令及其用法
<!--more-->

# Sort命令  

顾名思义：排序！默认递增的顺序

## 参数说明

### 常用命令：  
-   -n 依照数值的大小排序。  
-   -g 依照数值的大小排序，支持浮点数，但是有性能缺陷。
-   -t <分隔字符> 指定排序时所用的栏位分隔字符。  
-   -k 指定排序的列，可以指定多列（最多两列）
> sort -t"," -k 1,2 -g 4301342.csv
-   -r 以相反的顺序来排序。  
-   -u 相同key的条目只存储一条，用于去重
-   -o <输出文件> 将排序后的结果存入指定的文件。  
-   -f 排序时，忽略大小写。  
-   -b 忽略每行前面开始出的空格字符。  
-   -c 检查文件是否已经按照顺序排序。  
-   -d 排序时，处理英文字母、数字及空格字符外，忽略其他的字符。  
-   -i 排序时，除了040至176之间的ASCII字符外，忽略其他的字符。    
-   +<起始栏位>-<结束栏位> 以指定的栏位来排序，范围由起始栏位到结束栏位的前一栏位。  
-   -T 指定排序临时文件的生成位置，默认是/var/tmp目录
-   -S 指定用于进行排序的内存大小，默认最高占用可用内存的90%，单位可以用G,M等

### 特殊方法：  
-  -V 排序版本号，
> 一般的文件名`sort-1.024.003.tgz`都可以识别出1.024.003的版本号来  
-  --batch-size=num，指定最多一次处理的文件数，默认是16  
-  --debug 打印处理过程中的各种调试信息  
-  --files0-from=filename，从文件中读取处理的文件列表，文件名之间需要使用NUL（也就是"\0"）隔开(原文：like the output produced by the command "find ... -print0")，实测只能用这个字符！  
-  -m 将几个排序好的文件进行合并。  
-  -M 将前面3个字母依照月份的缩写进行排序。 
-  --parallel=n 使用n个线程并行排序，默认为处理可用核心数（最多为8）。内存增长规律：Note also that using n threads increases the memory usage by a factor of log n. Also see nproc invocation.

### 指定排序方式：  
- --radixsort,如果排序规范允许，请尝试使用基数排序。基数排序只能用于普通语言环境（C和POSIX），而不能用于数字或月份排序。 基数排序非常快速且稳定。[暂不明白普通语言环境指的是什么？]
- --mergesort，使用归并排序  
- --qsort，使用快速排序，不能用在-u和-s参数上
- --heapsort，使用堆排序，不能用在-u和-s参数上

### 没有理解的几个参数：  
-  --compress-program=RPOGRAM，后接标准输入输出的程序，用于数据压缩
-  --random-source=filename，原话：In random sort, the file content is used as the source of the 'seed'
data for the hash function choice.  Two invocations of random sort with the same seed data will use the same hash function and will produce the same result if the input is also identical.  By default, file /dev/random is used.
-  --mmap，原话：Try to use file memory mapping system call.  It may increase speed in some cases.
-  -h, 原话：Sort by numerical value, but take into account the SI suffix, if present.  Sort first by numeric sign (negative, zero, or positive); then by SI suffix (either empty, or `k' or `K', or one of `MGTPEZY', in that order); and finally by numeric value.  The SI suffix must immediately follow the number. For example, '12345K' sorts before '1M', because M is "larger" than K.  This sort option is useful for sorting the output of a single invocation of 'df' command with -h or -H options (human-readable).

# WC命令
利用wc指令我们可以计算文件的Byte数、字数、或是列数，若不指定文件名称、或是所给予的文件名为"-"，则wc指令会从标准输入设备读取数据。

## 常用命令
- -c或--bytes或--chars 只显示Bytes数。  
- -l或--lines 只显示行数。  
- -w或--words 只显示字数。  

# ps命令
# awk命令
# sed命令

# 扩展的知识  
## linux下nproc  
sort命令--parallel提到的，nproc是操作系统级别对每个用户创建的进程数的限制,在Linux下运行多线程时,每个线程的实现其实是一个轻量级的进程,对应的术语是:light weight process(LWP), 怎么知道一个用户创建了多少个进程呢，默认的ps是不显示全部进程的，需要‘-L' 才能看到所有的进程。

举例1：查看所有用户创建的进程数,使用命令：
```shell
ps h -Led -o user | sort | uniq -c | sort -n
```  
举例2：查看hfds用户创建的进程数，使用命令:
```shell
ps -o nlwp,pid,lwp,args -u hdfs | sort -n
```

## C++ 多进程编程函数posix_spawn  
参考2中一个回答中提到的。下面的代码创建了一个子进程，执行了ls命令并输出到了控制台中。还可以接收参数，传入argv中。  

```cpp
int main(int argc, char *argv[])
{
    pid_t child_pid;
    int ret;
    int wait_val;
 
    cout << "This is main process......" << endl;
    ret = posix_spawn(&child_pid, "ls", NULL, NULL, argv, NULL);
    if (ret != 0){
        cout << "posix_spawn is error" << endl;
        exit(-1);
 
    }
 
    wait(&wait_val);
    cout << "This is main process and the wait value is " << wait_val << endl;
 
    exit(0);
}
```

# 参考
1. <a href="https://unix.stackexchange.com/questions/120096/how-to-sort-big-files">How to sort big files? - StackOverFlow，使用--parallel的样例</a>
2. <a href="https://unix.stackexchange.com/questions/275501/gnu-sort-compress-program-compressing-only-first-temporary">GNU sort --compress-program compressing only first temporary -StackOverFlow, 使用sort --compress-program的样例</a>
3. <a href="https://blog.csdn.net/oDaiLiDong/article/details/50561257">linux下nproc的作用 -CSDN</a>
4. <a href="https://blog.csdn.net/Linux_ever/article/details/50295105">多进程编程函数posix_spawn实例 -CSDN</a>