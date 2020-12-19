# MyBatis中Sql获取与解析：

1、加载Mybatis的SQL文件xml在内存中，包括数据库配置、Mybatis映射文件，将Configuration、SqlSourceBuilder对象初始化；

2、解析xml文件数据，进行context初始化；
解析流程：
   1)、根据xml映射文件标签进行解析分类，如insert、delete、select、update，划分执行的操作类型；
   2)、根据文件标签属性如：id、parameterType、resultType等封装到MappedStatement对象中；
   3)、xml中sql语句(text文本内容)解析，如：select * from c_ar_meter where meter_no = #{meterNo}
   根据sql中包含${}、#{}来区别是DynamicSqlSource还是RawSqlSource，
   ${}利用DynamicSqlSource封装sql，#{}利用RawSqlSource封装sql；
   其中DynamicSqlSource和RawSqlSource都实现了接口SqlSource中getBoundSql方法，并返回BoundSql对象；
   但是接口SqlSource的实现类有4个：
      DynamicSqlSource: 处理包含${}的sql，也就是动态sql;
      RawSqlSource: 处理包含#{}的sql，也就是静态sql;
      ProviderSqlSource: 处理注解Annotation形式的sql;
      StaticSqlSource: sql的最终处理类;
      利用”?”替换掉参数，解析完成之后，最终返回SqlSource对象。
   4)、BoundSql boundSql = sqlSource.getBoundSql(参数对象)；获取到sql绑定;



3、简化版Demo:

```
Configuration configuration = sqlSession.getConfiguration();
SqlSourceBuilder sqlSourceParser = new SqlSourceBuilder(configuration);
Object paramObject = 参数对象；

//sql示例：select * from c_ar_meter where meter_no = #{meterNo}
SqlSource sqlSource = sqlSourceParser.parse(sql, paramObject.getClass(), new HashMap<>());
BoundSql boundSql = sqlSource.getBoundSql(paramObject);
boundSql.setAdditionalParameter("参数名", "参数值")

//msId表示用来标记MappedStatement，全局唯一
String msId = boundSql.getSql();
//mybatis mapper简易封装
MappedStatement ms = new MappedStatement.Builder(configuration, msId, sqlSource, SqlCommandType.UPDATE).build();
configuration.addMappedStatement(ms);

//更新执行
 sqlSession.update(msId, paramObject );
```

总结：
       对于的mybatis底层的实现原理，有了充分的认识以及深刻的学习，了解到MyBatis底层就是PreparedStatement直接操作，
       让程序员不用频繁的写PreparedStatement系列操作，达到充分解耦。让程序员只关注自己的业务实现即可，高度提高效率。