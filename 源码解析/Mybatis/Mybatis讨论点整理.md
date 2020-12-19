1、Mybatis的插件运行原理，如何编写一个插件？

​	采用动态代理方式；

​	插件可以拦截的四大对象：

​		Executor          *//执行增删改查操作* 

​		StatementHandler *//处理sql语句预编译，设置参数等相关工作；* 

​		ParameterHandler *//设置预编译参数用的* 

​		ResultSetHandler *//处理结果集*

​	![image-20200523153532705](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523153532705.png)

![image-20200523153555596](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523153555596.png)

SimpleExecutor是一种常规执行器，每次执行都会创建一个statement，用完后关闭。
ReuseExecutor是可重用执行器，将statement存入map中，操作map中的statement而不会重复创建立statement。
BatchExecutor是批处理型执行器。
![image-20200523153910173](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523153910173.png)

![image-20200523153928158](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523153928158.png)



自定义插件方式：

![image-20200523154109254](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523154109254.png)

![image-20200523154123967](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523154123967.png)



2、Mybatis是如何进行分页的？分页插件的原理是什么？

![image-20200523161336870](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523161336870.png)

3、 批量处理的动态SQL操作Demo

![image-20200523154241928](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523154241928.png)

4、MyBatis缓存原理及自定义缓存适配器？

​	 

```xml
 //开启二级缓存
 <cache flushInterval="20000" eviction="FIFO" size="512" readOnly="true"/>
```

自定义缓存必须实现org.apache.ibatis.cache.Cache接口，并在Mapper中加入：

```xml
<cache type="自定义缓存类"/>
```

5、Mybatis是如何判断应该使用哪个databaseId对应的标签的？
   1、SqlSessionFactory配置项初始化

![image-20200523151523435](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523151523435.png)

![image-20200523151600462](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523151600462.png)

2、Mybatis Mapper文件加载

![image-20200523151754532](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523151754532.png)

![image-20200523151833314](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523151833314.png)

![image-20200523151945664](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523151945664.png)

![image-20200523152016036](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523152016036.png)

![image-20200523152045253](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523152045253.png)

![image-20200523152120306](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523152120306.png)

![image-20200523152204497](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523152204497.png)

![image-20200523152235397](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523152235397.png)

![image-20200523152320773](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523152320773.png)

![image-20200523152800051](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523152800051.png)

![image-20200523152825391](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523152825391.png)



6、Hibernate实体对象四大状态？

​	  瞬时态（Transient)：创建session之后，由 new 操作符创建，且尚未与Hibernate Session 关联的对象被认定											为瞬时的；

​      持久态（Persistent)：存在于session作用范围内的对象；

​      脱管态（Detached)：session关闭之后，对象就变为脱管；

​      删除态（Deleted)：调用session的delete方法之后，对象就边为删除态；

![image-20200523170259221](C:\Users\xuefengwang\AppData\Roaming\Typora\typora-user-images\image-20200523170259221.png)