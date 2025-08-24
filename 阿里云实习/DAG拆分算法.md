背景：在提交DAG进行分析时，一项最主要的工作就是根据算子在各个引擎中的支持情况，将DAG拆分为多个不同的SubDAG，提交给不同的引擎执行。算子在每一个引擎上有一个接受状态EngineAcceptance，它代表了当前具有某些属性值的算子在某个引擎上的接受情况。

算法流程：

+ 按照拓扑排序给算子进行打标，主要标记每个算子应该交给哪个引擎执行，以及执行时是否需要BREAK
+ 按照第一步的打标结果，将算子进行分组，初步形成各个SubDAG的nodeGroup以及SubDAG之间的依赖关系
+ 按照第二步的依赖关系，简历SubDAG之间的输入输出关系。形成SubDAG Graph

旧分组算法：

+ 所有的输入节点，根据所属引擎分配不同的groupId

+ 按照拓扑顺序遍历，对于每一个有输入的节点，判断输入节点是否有和自己所属引擎不一致的，或者是否有BREAK，如果有，则给自己分配一个新的的groupId，否则划分到输入节点的groupId

  **缺陷**：某些情况下将本来属于同分组的节点分到不同的分组中，形成多个冗余的SubDAG，导致整体执行效率变低。例如

  ```python
  v0 = mt.random.rand(5,5)
  v1 = md.DataFrame(v0,columns=list("abcde"))
  v2 = v1["a"] + 1
  v3 = v1["a"] + 2
  ```

  v0作为唯一一个没有输入的节点，被分配到了groupId[v0] = 0，v1只有v0一个输入，且都属于SPE引擎，则groupId[v1] = 0。而按照拓扑排序顺序遍历到的v2，由于其唯一的输入属于SPE，自己属于MCSQL，因此分配到了新的groupId[v2] = 1，同理，groupId[v3] = 2。但是期望情况是，v2、v3应该属于同一个group，因为他们之间没有严格的BREAK或者DENY依赖

新分组算法：对每一个分组用（engine-type，groupId）进行唯一标识，不同的engine课亦有同样的groupId的分组，这样做的好处是：对于不同的engine-type之间的分组依赖，假设前驱节点属于engineA的节点分组（A，x），则属于engineB的后继节点n为（B，x+1）。对另一个属于（A，x）分组的节点的后继，如果某个后继n\`属于engineB，则其groupId为（B，x+1），此时n和n\`属于同一个group，即使他们之间没有任何依赖关系。这样尽可能减少SubDAG的数量，提升效率。
但这样分组会引入一个问题，即某些情况下，SubDAG会出现循环依赖的情况，从而出现环路影响后续计算
三色标记算法正是为了解决ProjectionWithSameIndexMergeRule中的环路设计的，因此在初步确定拆分的分组后，对预分组应该采用三色标记算法消除环路

改进后的算法包括预分组和三色标记分组两个阶段：

+ 预分组流程：
  + 所有输入节点，按照所属引擎分配不同的groupId为（engine-name，group-seq）
  + 按照拓扑顺序遍历，对于每一个有输入的节点x，检查：
    + 如果有和自己同引擎且accept的节点y，则将当前节点的groupId赋值为max（groupId[x]，groupId[y]）
    + 如果有和自己同引擎且break的节点y，则将当前节点的groupId赋值为max（groupId[x]，groupId[y+1]）
    + 否则，将自己的groupId设置为max（groupId[x]，0）

+ 三色标记分组流程