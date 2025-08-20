![image-20231120150432862](C:\Users\29278\AppData\Roaming\Typora\typora-user-images\image-20231120150432862.png)

Collection(interface)集合：

Collection继承了Iterable接口。

实现了Iterable接口的集合被允许使用for-each进行遍历。

实现了Iterable接口的集合必须提供一个被称为iterator的方法，该方法返回一个Iterator类型的对象。Iterator是一个接口。

List(interface)继承自Collection，有两种实现：

+ ArrayList(具体实现类，可实例化)，优点在于对于get和set的调用花费常数时间，缺点是插入和删除费时。
+ LinkedList(同上)，提供了表的双链表实现，便于插入和删除，不便于索引。

Stack(Interface):

​	实现类：stack

Queue(interface):

​	实现类：LinkedList；PriorityQueue
