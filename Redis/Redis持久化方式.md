# 一、Redis持久化方式

## 		1、RDB方式

​			  RDB方式是通过快照（ snapshotting ）完成的，在指定的时间间隔内将内存中的数据集快照写入磁盘；

​			触发时机：

​						1、可在redis.conf中配置规则；

​						2、执行命令

​								save：执行该命令会阻塞当前Redis服务器，执行save命令期间，Redis不能处理其他命令；

​								gbsave：执行该命令时，Redis会在后台异步进行快照操作；

​								flushall：执行该命令时，Redis会在后台异步处理。

​						3、第一次执行主从复制时；

​			优缺点：

​						1、生成RDB文件的时候，主进程会fork()一个子进程来处理所有保存工作，主进程不需要进行任何							  磁盘IO操作；

​						2、RDB 恢复数据集的速度也要比 AOF 恢复的速度要快；

​						3、可能数据丢失，在RDB操作期间，若数据有修改，则修改的数据不会保存。

## 		2、AOF(append only file)方式

​			  redis会将每一个收到的写命令都通过write函数追加到AOP文件中；

​			  如图：

​			 ![image-20200519112648564](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200519112648564.png)

​			AOF文件重写：

​					由于命令是直接保存在AOF文件中的，会导致AOP文件越来越大；运用bgrewriteaof命令，进行AOF文			件重写，重写时，将整个内存中的数据库内容用命令的方式重写一个新的aof文件；

​			

​			优缺点：

​						1、AOF数据安全，不会影响客户端的读写；

​						2、对于同一份数据来说，AOF日志文件通常比RDB数据快照文件更大；

​	

RDB与AOP的比较：

![img](https://pics5.baidu.com/feed/8326cffc1e178a82c532308ef2117b8ba977e8ae.jpeg?token=fea28817e45f0e091b5be3854d856fbb&s=BD48B55F1C784C095E61DCEB0300D036)

