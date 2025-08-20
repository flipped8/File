tomcat架构：

​	<img src="C:\Users\29278\AppData\Roaming\Typora\typora-user-images\image-20231108110346549.png" alt="image-20231108110346549" style="zoom: 50%;" />

<img src="C:\Users\29278\AppData\Roaming\Typora\typora-user-images\image-20231108111240175.png" alt="image-20231108111240175" style="zoom:33%;" />

Tomcat 的核心功能有两个，分别是负责接收和反馈外部请求的连接器 Connector，和负责处理请求的容器 Container。其中连接器和容器相辅相成，一起构成了基本的 web 服务 Service。每个 Tomcat 服务器可以管理多个 Service。

<img src="C:\Users\29278\AppData\Roaming\Typora\typora-user-images\image-20231108112358744.png" alt="image-20231108112358744" style="zoom:33%;" />

container负责对内处理业务逻辑，其内部由engine、host、context和wrapper组成。

服务器有且仅有一个，代表服务。

connector代表端口，默认有两个，一个是http协议，用来处理http请求；一个是ajp协议。connector的作用是监听端口。

engine用来处理请求，host代表主机。

[tomcat与servlet - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/465936851)

[牛逼！硬核图解 Tomcat 整体架构 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/260044529)

[Spring boot，Tomcat容器之间关系以及请求执行流程_springboot和tomcat什么联系-CSDN博客](https://blog.csdn.net/qq_22162093/article/details/106107779)