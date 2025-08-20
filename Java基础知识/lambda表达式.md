lambda表达式是一个可传递的代码块。

表达形式(1)：参数，箭头(->)以及一个表达式。如果代码要完成的计算无法放在一个表达式里，就可以像写方法一样，把代码放在{}中，并包含显式的return语句。无需指定lambda表达式的返回值类型，可以由上下文推导得出
![image-20231121104351032](C:\Users\29278\AppData\Roaming\Typora\typora-user-images\image-20231121104351032.png)
即使lambda表达式没有参数，仍然要提供空括号，就像无参方法一样。
![image-20231121104610122](C:\Users\29278\AppData\Roaming\Typora\typora-user-images\image-20231121104610122.png)
如果可以推断出lambda表达式的参数类型，就可以忽略其类型。
![image-20231121104709444](C:\Users\29278\AppData\Roaming\Typora\typora-user-images\image-20231121104709444.png)
lambda表达式将赋给一个字符串比较器，所以参数必然是字符串。
如果方法只有一个参数，而且这个参数的类型可以推导得出，那么甚至可以省略小括号
![image-20231121105004422](C:\Users\29278\AppData\Roaming\Typora\typora-user-images\image-20231121105004422.png)
对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个lambda表达式，这种接口称为函数式接口。

lambda表达式中捕获的变量必须实际上是事实最终变量，即初始化后就不在为他赋新值。