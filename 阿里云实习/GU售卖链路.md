

GU Quota在完成生产后会在计算集群上增加一个Ray WorkGroup，一个GU Quota对应一个Ray WorkGroup。

GU Quota对应Ray WorkGroup，Ray集群存在是GU Quota创建的前置条件。现状是Ray与CU Quota一一对应，本次更改为Ray 与\<Tenant，CLuster>一一对应

关于CU Quota调度：

+ session创建会导致CU QUota的调度，CU Quota经由框架Quota Selector调度选择出来
+ MaxFrame Task将CU Quota信息转换为OTS Ray表主键\<TenantId，CulsterName>，从OTS Ray表中查询出Ray Endpoint。
+ MaxFrame Task向Ray提交请求，创建作业AM（session Actor）

关于GU Quota调度：

+ submit UDF算子会触发GU Quota的调度
+ FrameDriver是运行在MaxFrame Task内的一个module，负责解析UDF算子。当FrameDriver解析出UDF算子之后，需要调用Quota Selector提供的接口获得GU Quota Info
+ 如果GU Quota Info不存在，这里拦截处理错误码：GU Quota Not Exists
+ 如果GU Quota Info存在，需要将其转换为OTS Ray表主键\<TenantId、WorkGroup、ClusterName、GuQuotaName>，从OTS表中查询GU Work Group的状态
+ 如果GU WorkGroup不存在，或者GU WorkGroup状态不等于Running，那么返回错误码，GU Quota创建中
+ 步骤四五校验通过，则向AM（session actor）提交UDF算子
+ session actor将UDF转换成物理执行计划（sub step graph）提交到EGS运行

GU Quota生产事件

+ TopConcole调用控制集群提供的库存接口获得库存，控制集群将请求转发到伏羲SLA Controller。这里需要由伏羲团队提供库存接口的定义
+ 如果库存不足则流程终止，如果有库存，那么允许用户付款开通GU Quota。TopConsole调用控制集群提供的接口创建Quota
  + TopConsole收到EGS扩容完成的消息后，创建MaxFrame Task完成GU WorkGroup定义。注意，需要传递的参数有MaxFrame给出
  + MaxFrame Task会先检查Ray集群是否存在，如果不存在则发送MFS CRD到计算集群同时创建\<Ray集群，GU WorkGroup>。如果存在，则进入下一步
  + MaxFrame Task会发送MFS CRD到计算集群，创建GU WorkGroup
  + 完成GU WorkGroup的创建后，MaxFrame Service Operator会更新OTS Ray表

GU Quota退订事件

+ MC TopConsole收到退订事件后，处理Quota退订
  + 创建MaxFrame Task删除Quota
  + MaxFrame Task通过查询OTS获得GU WorkGroup的状态。如果不存在则直接返回成功给MC TopConsole。如果GU WorkGroup存在，那么创建MFS CRD给MFS Operator
  + MFS Operator更新Ray，删除GU WorkGroup
  + TopConsole检查MaxFrame Task状态，如果成功则继续调用接口框架删除Quota





**MAXFRAME售卖链路实现细节**

生产GU Quota

调用条件：Top Concle识别到EGS资源完成了购买

内部逻辑：

+ 将tenantId、clusterName作为主键，查询Ray meta表，检查是否存在了Ray Cluster
+ 如果不存在转为下面步骤，存在直接执行下一步
  + 同时创建ray cluster以及GU WorkGroup
  + 利用租户tenantId查询租户下所有的Quota
  + 从上述结果中找到后付费Quota，后付费Quota的name和cluster作为下一步的input。
  + 利用GU quotaName，GU cluster，GU quotaMonopolyMachineGroup获得机型信息
  + 通过kube task 发送一个MFS CRD到计算集群
  + 通过kube task发送一个MFS crd到计算集群，仅仅新建一个workgroup。新CRD结构
  + 等待ray&workgroup创建完成，发送mq消息

退订GU Quota

调用条件：用户退订一级GU Quota操作

内部逻辑：

+ 将tenantId、clusterName作为主键查询Ray Meta表，检查是否已经存在了ray cluster，如果不存在，则直接返回成功
+ 利用root quota name、root quota id查询quota meta，将所有二级quota都找出来，针对每个二级quota做如下操作
  + 将tenantId cluster rayName GU quota name查询workgroup，如果不存在则直接成功，否则进行下一步
  + 通过kube task发送一个MFS CRD到计算集群，仅仅删除一个workGroup。新CRD结构





联调问题记录

+ 发布售卖patch后，无法按照quota维度管理ray，不利于测试以及watch dog
+ 创建ray报错：无法找到MFS CRD
+ 调用maxframe task传递的参数中，quotaName和quotaId需要改为二级quota的信息
+ 创建的二级quota minCU错误
+ 目前后付费quota有两个二级quota，一个backgroup quota，另一个是普通quota。默认创建的ray cluster不能映射到backgroup quota
+ 伏羲pod调度不起来。讨论后fix方案：maxframe task改用动态值，把动态值传给operator
+ 后付费额度不够，cu pod/head都拿不到资源
+ deploy状态的MFS禁止update，这个限制有点苛刻了，只要ray crd存在就允许update会更好
+ guquota的嵌套层数过多operator无法解析出来，c++及operator部分都要修改guquota的parse格式