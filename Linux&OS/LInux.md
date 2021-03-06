# Linux 基础

## 父子进程

1.定义  

1) fork()  
通过复制父进程来创建一个新的进程（子进程）
// pid = fork() 子进程会返回0， 父进程会返回子进程的pid
2) exec()  用于让子进程执行自己的程序  
执行  
3) wait()  
主进程阻塞等待子进程结束，若子进程结束返回子进程的pid，并销毁子进程号  

2.特点  
1）fork后会产生两个进程，子进程有其唯一的PID  
2）fork后子进程的代码段、数据段、堆栈区和父进程一模一样（即创建时直接映射到父进程的物理地址）  
3）当主进程和子进程有改变的时候，才开始分配新的栈、堆和数据段  
4）当执行exec后将子进程的代码段进行替换

3.孤儿进程  
当父进程退出后，它的子进程还在运行，那么子进程会变成孤儿进程，孤儿进程会被init（进程号为1）所收养，并由init进程对它们完成状态收集工作  

4.僵尸进程  
一个进程fork创建子进程，如果子进程退出，而父进程没有调用wait或者waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中（因为父进程需要调用后知道子进程的终止状态）  

## 链接  

1. 硬链接  
    定义：一个innode号有多个文件名  
    用法：link/ln old new  
    特点：  
    a.删除一个文件，不影响其他文件  
    b.只能链接已经存在的文件（不是目录）  
2. 软连接（符号链接）  
    定义：一个文件的data中保存了指向另一个文件的路径  
    用法：ln -s  
    特点：  
    a.软链接有自己的属性和权限等  
    b.可以对不存在的文件或者目录进行创建  
    c.删除软连接不会删除原始文件，而删除原始文件会导致软链接失效  

## linux下内存分析

Linux系统在内存分配上：内存充足时，尽量使用内存来缓存一些文件，从而加快进程的运行速度，而当内存不足时，会通过相应的内存回收策略收回cache内存，供进程使用  

### 系统整体使用

1.top  
2.free -m 可以看到简单的使用情况  
3./proc/meminfo 详细信息  

### 进程

/proc ：系统内核的映像  

1. cat /proc/pid/smaps 内存详细信息  
2. cat /proc/pid/status 进程的整体信息  

## 特殊符号

```c
&& // 前一条执行正确再执行下一条
|| // 前一条执行失败后再下一条

& // 表示后台执行
| // 管道
```

## 网络

1.traceroute  
traceroute ip地址  
查看数据包到某地址经过的网络链路  
2.lsof   
列出当前系统打开的文件，可以用来查看网络连接  
-i 网络连接 :80显示端口 tcp显示  
3.iftop  
网络详细信息  
4.telnet  
远程登录地址  
telnet ip  
5.curl  
下载网页html内容  
curl http://www.baidu.com  

## 搜索

1.find  
find ./ -name "*.cc" 搜索文件名  
find ./ -name "*.hpp" -mtime 0 //(-1 昨天修改, 0 今天修改, +1 昨天之前修改的)
2.grep  
grep -nir xx 搜索内容  

## 文本操作

1.head -n 1 获取第一行  
  tail -n 1 获取最后一行  
2.awk '{print $1}' 获取第一列  
3.输出重定向  
stdout(1) stderr(2)  
\>out.txt 2>&1 //实际为1>out.txt 2>&1，其中1>简写为>，2>&1指将2通过管道输出到1的输出中  
其中 >> 是追加, > 是覆盖  

## 系统信息

1.du  
-h // 按MB显示  
-s // 只计算总量  
2.htop  
系统详细信息  
3.env  
查看环境变量，可以配合grep进行相关的展示  
4.查看终端历史命令  
vi ~/.bash_history  

## 编译相关

1.nm  
查看符号表  
nm libmrtc-base.so | c++filt // 转换为C++风格显示函数名  

## 应用

1.释放缓存  
echo 1 > /proc/sys/vm/drop_caches  
2.崩溃后继续执行  
sh末尾加上/bin/bash  
3.core 存储位置修改  
echo "/data/core-%e-%t-%p" > /proc/sys/kernel/core_pattern  
4.配置环境变量  
1）全局变量修改  
/etc/profile  
source /etc/profile  
2）用户变量修改  
~/.bashrc  
source ~/.bashrc  
3）参考  
<http://www.mashangxue123.com/Linux/2613038403.html>  
加载顺序  
/etc/profile  
/etc/paths  
~/.bash_profile , ~/.bash_login , ~/.profile 按照从前往后的顺序读取,只会读取一个  
~/.bashrc  
修改  
sudo vi ~/.bashrc  
export PATH=$PATH:/opt/local/bin:/opt/local/sbin  
source ~/.bash_profile // 生效  
echo $PATH // 查看是否生效

## 其他  

1. 写时复制  
1）定义：多个调用者同时请求相同的资源，会共同获得相同的指针指向相同的资源，当某个调用者试图修改资源内容时，系统才会真正复制一份副本给修改者进行修改，其他调用者资源仍然不变  
2）应用  
a.fork()后的子进程与主进程共享同一份资源，执行exec后才会给子进程分配空间  
b.string中允许写时复制，C++11中取消了  

2. xargs  
获取前面执行的结果