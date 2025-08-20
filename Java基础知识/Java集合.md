Java集合类库将接口（interface）与实现（implementation）分离；

Queue（interface）：

​	两种实现方式：循环数组（ArrayDeque）；链表（LinkedList）；

​	当使用队列时，一旦已经构造了集合，就不需要知道究竟使用了哪种实现，只有在构造集合时，才会	使用具体的类，可以使用接口类型存放集合引用。

​		例：Queue<Customer> expressLane = new CircularArrayQueue<>（100）；

​	利用这种方法，一旦改变了想法就可以很轻松的使用另一种不同的实现。

​	循环链表比链表更高效，但是一个有界集合。

​	以Abstract开头的类是为类库实现者设计的。

Collection（interface）集合类：

​	不允许有重复对象。

​	iterator方法用于返回一个实现了Iterator接口的对象，可以使用这个对象一次访问集合中的元素。

​	add方法向集合中添加元素，成功返回true，失败返回false。

Iterator（interface）迭代器：

​	next方法，反复调用来逐个访问集合中的每个元素，到达集合末尾抛出NoSuchElementException

​	hasNext方法，查看是否有下一个元素，先于next方法使用

​	如果想要查看集合中所以元素，就请求一个迭代器，当hasNext方法返回true时就反复调用next方	法。（用“for  each”可以更简练的表示同样的操作，编译器将foreach循环转成带有迭代器的循环）

​	for each循环可以处理任何实现了Iterable接口的对象，这个接口至包含一个抽象方法iterator

​	Collection接口扩展了Iterable接口，对于标准类库中的任何集合都可以使用“for  each”循环

​	可以认为Java迭代器位于两个元素之间，调用next时，会越过下一个元素并返回刚才越过的函数的	引用

链表LinkedList（具体集合）：

​	