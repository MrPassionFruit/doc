**一、Redis内存统计，比较重要的部分**

​	1、used_memory：Redis分配的内存总量，单位字节；

​	2、used_memory_rss：Redis进程占据操作系统的内存，单位字节；

​	3、mem_fragmentation_ratio：内存碎片比率，

​			其中，内存碎片比率 = used_memory_rss / used_memory，值越大，内存碎片比例越大；

​			内存碎片是Redis在分配、回收物理内存过程中产生的，如果对数据的更改频繁，而且数据之间

​	的大小相差很大，可能导致redis释放的空间在物理内存中并没有释放，但redis又无法有效利用。就会形成内存	碎片；



​	内存碎片清除的方式：

​			方式一：4.0版本之前，只能重启Redis来解决内存碎片问题，因为重启之后，Redis重新从备份文件中读取	数据，在内存中进行重排，为每个数据重新选择合适的内存单元，减小内存碎片。

​			方式二：4.0版本之后提供2种方式，一种命令方式进行内存回收，命令MEMORY PURGE，此方式会阻塞主	进程；另一种方式，在配置文件中配置activedefrag yes来进行内存碎片的自动清理，目前该功能还是实验性的。



**二、Redis数据类型**

​	1、字符串，长度不能超过512MB；

​	2、列表，内部编码可为压缩列表、双端链表

​			双端链表：由一个list结构和多个listNode结构组成；

​			压缩列表：压缩列表是Redis为了节约内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型数据	结构。压缩列表必须满足2个条件：列表元素数量小于512个，列表所有字符串对象都不足64字节。

​			优缺点对比：压缩表可以节省内存空间，但是在进行修改、删除时，复杂度较高

​			总结：节点数量少时，可以使用压缩列表；节点数多时，使用双端链表；

​	3、哈希

​			内部编码可以是压缩列表（ziplist）和哈希表（hashtable）；

​			hashtable：一个hashtable由1个dict结构、2个dictht结构、1个dictEntry指针数组（称为bucket）和
多个dictEntry结构组成。其中：

​					dictEntry结构用于保存键值对；

​					bucket是一个数组，数组的每个元素都是指向dictEntry结构的指针；

​					dict结构中存在属性dictht数组，方便执行rehash，所有的数据都存在dictht[0]中，当rehash时，			dictht[0]中的数据存到dictht[1]中，然后在将dictht[1]刷回dictht[0]，清空dictht[1]；

​	4、Set：

​			内部编码可以是整数集合（intset）或哈希表（hashtable）；

​			整数集合必须满足：集合中元素数量小于512个，且所有元素都是整数值。

​	5、ZSet：

​			内部编码可以是压缩列表（ziplist）或跳跃表（skiplist）；

​			跳跃表是一种有序数据结构，通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点

​	的目的；

​			![image-20200518201756325](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200518201756325.png)

​			

​			跳跃表性质：

​					 (1) 由很多层结构组成；
 					(2) 每一层都是一个有序的链表；
 					(3) 最底层(Level 1)的链表包含所有元素；
 					(4) 如果一个元素出现在 Level i 的链表中，则它在 Level i 之下的链表也都会出现；
 					(5) 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素。



**三、缓存淘汰策略**

​		1、volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰；

​		2、volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰；

​		3、volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰；

​		4、allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰；

​		5、allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰；

​		6、no-enviction（驱逐）：禁止驱逐数据；

LRU思路：

​		比如：用一个链表保存缓存数据，

​				1、新数据插入表头；

​				2、将链表中被访问的数据移到表头；

​				3、当表满的时候，表尾的数据丢弃；



**四、Redis事务常用命令**

​		multi：开启事务，后续命令将放入队列中；

​		discard：清除队列中的命令；

​		watch：开启事务监控；

​		unwatch：清除事务监控；

​		exec：执行multi的队列；

