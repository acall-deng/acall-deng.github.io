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

## 01. 参数说明

### (1) 常用命令：  
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

### (2) 特殊方法：  
-  -V 排序版本号，
> 一般的文件名`sort-1.024.003.tgz`都可以识别出1.024.003的版本号来  
-  --batch-size=num，指定最多一次处理的文件数，默认是16  
-  --debug 打印处理过程中的各种调试信息  
-  --files0-from=filename，从文件中读取处理的文件列表，文件名之间需要使用NUL（也就是"\0"）隔开(原文：like the output produced by the command "find ... -print0")，实测只能用这个字符！  
-  -m 将几个排序好的文件进行合并。  
-  -M 将前面3个字母依照月份的缩写进行排序。 
-  --parallel=n 使用n个线程并行排序，默认为处理可用核心数（最多为8）。内存增长规律：Note also that using n threads increases the memory usage by a factor of log n. Also see nproc invocation.

### (3) 指定排序方式：  
- --radixsort,如果排序规范允许，请尝试使用基数排序。基数排序只能用于普通语言环境（C和POSIX），而不能用于数字或月份排序。 基数排序非常快速且稳定。[暂不明白普通语言环境指的是什么？]
- --mergesort，使用归并排序  
- --qsort，使用快速排序，不能用在-u和-s参数上
- --heapsort，使用堆排序，不能用在-u和-s参数上

### (4) 没有理解的几个参数：  
-  --compress-program=RPOGRAM，后接标准输入输出的程序，用于数据压缩
-  --random-source=filename，原话：In random sort, the file content is used as the source of the 'seed'
data for the hash function choice.  Two invocations of random sort with the same seed data will use the same hash function and will produce the same result if the input is also identical.  By default, file /dev/random is used.
-  --mmap，原话：Try to use file memory mapping system call.  It may increase speed in some cases.
-  -h, 原话：Sort by numerical value, but take into account the SI suffix, if present.  Sort first by numeric sign (negative, zero, or positive); then by SI suffix (either empty, or `k' or `K', or one of `MGTPEZY', in that order); and finally by numeric value.  The SI suffix must immediately follow the number. For example, '12345K' sorts before '1M', because M is "larger" than K.  This sort option is useful for sorting the output of a single invocation of 'df' command with -h or -H options (human-readable).

## 02. 高级可装逼用法(-k选项)

```shell
$ sort -t ' ' -k 1.2,1.2 -nrk 3,3 facebook.txt
baidu 100 5000
google 110 5000
sohu 100 4500
guge 50 3000
```  
1.2,1.2表明只对第一个域的第二个字母进行排序。（如果使用-k 1.2，则因为你省略了End部分，意味着你将对从第二个字母起到本域最后一个字符为止的字符串进行排序）。对于员工工资进行排序，我们也使用了-k 3,3，这是最准确的表述，表示我们“只”对本域进行排序，因为如果你省略了后面的3，就变成了我们“对第3个域开始到最后一个域位置的内容进行排序” 了。

```shell
FStart.CStart Modifie,FEnd.CEnd Modifier
-------Start--------,-------End--------
 FStart.CStart 选项  ,  FEnd.CEnd 选项
```
总的来说，这个语法格式可以被其中的逗号,分为两大部分，Start部分和End部分。Start部分也由三部分组成，其中的Modifier部分就是我们之前说过的类似n和r的选项部分。我们重点说说Start部分的FStart和C.Start。C.Start也是可以省略的，省略的话就表示从本域的开头部分开始。FStart.CStart，其中FStart就是表示使用的域，而CStart则表示在FStart域中从第几个字符开始算“排序首字符”。同理，在End部分中，你可以设定FEnd.CEnd，如果你省略.CEnd，则表示结尾到“域尾”，即本域的最后一个字符。或者，如果你将CEnd设定为0(零)，也是表示结尾到“域尾”。

# WC命令
利用wc指令我们可以计算文件的Byte数、字数、或是列数，若不指定文件名称、或是所给予的文件名为"-"，则wc指令会从标准输入设备读取数据。

## 01. 常用命令
- -c或--bytes或--chars 只显示Bytes数。  
- -l或--lines 只显示行数。  v
- -w或--words 只显示字数。  

# sed命令  

## 01. 选项  
- -e<script>或--expression=<script>：以选项中的指定的script来处理输入的文本文件；
- -f<script文件>或--file=<script文件>：以选项中指定的script文件来处理输入的文本文件；

## 02. 命令部分  

-  a\ 在当前行下面插入文本。
-  i\ 在当前行上面插入文本。
-  c\ 把选定的行改为新的文本。
-  d 删除，删除选择的行。
-  D 删除模板块的第一行。
-  s 替换指定字符
-  h 拷贝模板块的内容到内存中的缓冲区。
-  H 追加模板块的内容到内存中的缓冲区。
-  g 获得内存缓冲区的内容，并替代当前模板块中的文本。
-  G 获得内存缓冲区的内容，并追加到当前模板块文本的后面。
-  l 列表不能打印字符的清单。
-  n 读取下一个输入行，用下一个命令处理新的行而不是用第一个命令。
-  N 追加下一个输入行到模板块后面并在二者间嵌入一个新行，改变当前行号码。
-  p 打印模板块的行。
-  P(大写) 打印模板块的第一行。
-  q 退出Sed。
-  b lable 分支到脚本中带有标记的地方，如果分支不存在则分支到脚本的末尾。
-  r file 从file中读行。
-  t label if分支，从最后一行开始，条件一旦满足或者T，t命令，将导致分支到带有标号的命令处，或者到脚本的末尾。
-  T label 错误分支，从最后一行开始，一旦发生错误或者T，t命，将导致分支到带有标号的命令处，或者到脚本的末尾。
-  w file 写并追加模板块到file末尾。  
-  W file 写并追加模板块的第一行到file末尾。  
-  ! 表示后面的命令对所有没有被选定的行发生作用。  
-  = 打印当前行号码。  
-  \# 把注释扩展到下一个换行符以前。  

## sed替换标记

-  g 表示行内全面替换。  
-  p 表示打印行。  
-  w 表示把行写入一个文件。  
-  x 表示互换模板块中的文本和缓冲区中的文本。  
-  y 表示把一个字符翻译为另外的字符（但是不用于正则表达式）
-  \1 子串匹配标记
-  & 已匹配字符串标记

## sed元字符集

-   ^ 匹配行开始，如：/^sed/匹配所有以sed开头的行。
-   $ 匹配行结束，如：/sed$/匹配所有以sed结尾的行。
-   . 匹配一个非换行符的任意字符，如：/s.d/匹配s后接一个任意字符，最后是d。
-   * 匹配0个或多个字符，如：/*sed/匹配所有模板是一个或多个空格后紧跟sed的行。
-   [] 匹配一个指定范围内的字符，如/[ss]ed/匹配sed和Sed。  
-   [^] 匹配一个不在指定范围内的字符，如：/[^A-RT-Z]ed/匹配不包含A-R和T-Z的一个字母开头，紧跟ed的行。
-   \(..\) 匹配子串，保存匹配的字符，如s/\(love\)able/\1rs，loveable被替换成lovers。
-   & 保存搜索字符用来替换其他字符，如s/love/**&**/，love变成**love**。
-   \< 匹配单词的开始，如:/\<love/匹配包含以love开头的单词的行。
-   \> 匹配单词的结束，如/love\>/匹配包含以love结尾的单词的行。
-   x\{m\} 重复字符x，m次，如：/0\{5\}/匹配包含5个0的行。
-   x\{m,\} 重复字符x，至少m次，如：/0\{5,\}/匹配至少有5个0的行。
-   x\{m,n\} 重复字符x，至少m次，不多于n次，如：/0\{5,10\}/匹配5~10个0的行。

# strace 命令  
用于追踪Linux命令的系统调用过程。当软件在一台机器上正常工作，但在另一台机器上却不能正常工作，同时抛出了有关文件、权限或者不能运行某某命令等模糊的错误信息时，strace 往往能大显身手。不幸的是，它不能诊断高等级的问题，例如数字证书验证错误等。这些问题通常需要组合使用 strace（有时候是 ltrace）和其它高级工具（例如使用 openssl 命令行工具调试数字证书错误）。

## 参数列表  

- -o 将输出保存到文件
- -s 查看更多参数，某些时候只能看到字符串的32个字节，使用`-s 128`的形式将其扩展
- -y 追踪文件或套接字，显示出每个文件描述符的具体指向
- -p 追踪正在运行的进程
- -f 追踪子进程，但 strace 显示的是单个调用事件流。当追踪多个进程的时候，你将会看到以 `<unfinished ...>`开始的初始调用，接着是一系列针对其它线程的调用，最后才出现以`<... foocall resumed>`结束的初始调用。
- -ff 将追踪子进程的所有调用分离到不同的文件中
- -e 进行过滤


# ps命令
# awk命令

# 扩展的知识  
## 01. linux下nproc  
sort命令--parallel提到的，nproc是操作系统级别对每个用户创建的进程数的限制,在Linux下运行多线程时,每个线程的实现其实是一个轻量级的进程,对应的术语是:light weight process(LWP), 怎么知道一个用户创建了多少个进程呢，默认的ps是不显示全部进程的，需要‘-L' 才能看到所有的进程。

举例1：查看所有用户创建的进程数,使用命令：
```shell
ps h -Led -o user | sort | uniq -c | sort -n
```  
举例2：查看hfds用户创建的进程数，使用命令:
```shell
ps -o nlwp,pid,lwp,args -u hdfs | sort -n
```

## 02. C++ 多进程编程函数posix_spawn  
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
5. <a href="https://man.linuxde.net/sort">sort命令 -k参数高级用法的出处</a>
6. <a href="https://mp.weixin.qq.com/s/Nzm7Ayw_vd5hbfBXFpDCYQ"> 在软件部署中使用 strace 进行调试 | Linux 中国 </a>