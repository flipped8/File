Mybatis-Plus只能单表查询，多表查询要自己实现。

通过扫描实体类，并基于反射获取实体类信息作为数据库表信息。

俗成约定：

+ 类名驼峰转下划线作为表名
+ 名为id的字段作为主键
+ 变量名驼峰转下划线作为表的字段名

常见注解：

+ @TableName：用来指定表名
+ @TableId：用来指定表中的主键字段信息
+ @TableField：用来指定表中的普通字段信息。成员变量以is开头，且是布尔值也要使用；成员变量名与数据库关键字冲突；成员变量不是数据库字段，@TableField（exist = false）。

IdType常见类型：

+ AUTO：数据库自增长
+ INPUT：通过set方法自行输入
+ ASSIGN_ID：分配ID

![image-20240314165739039](C:\Users\29278\AppData\Roaming\Typora\typora-user-images\image-20240314165739039.png)

![image-20240314195240737](C:\Users\29278\AppData\Roaming\Typora\typora-user-images\image-20240314195240737.png)

条件构造器Wrapper：

指定SQL需要拼接的查询条件

baseMapper

wrapper--核心：lambdaQueryWrapper

自定义sql

IService

lambdaQuery

resultMap

分页查询：

+ 先配置Interceptor对象，然后设置page对象。