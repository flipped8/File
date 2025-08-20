起步依赖原理：

​		maven的依赖传递

自动配置：

​		spring容器启动后，一些配置类、bean对象就自动存入到了IOC容器中，不需要手动声明。

​	原理（高频面试）：

​			方案一：显式声明ComponentScan组件扫描；使用繁琐，性能低

​			方案二：Import注解导入；使用Import导入的类会被加入到IOC容器中；导入形式有：普通类，配置类，							ImportSelector接口实现类。EnableXXX注解（第三方使用）。