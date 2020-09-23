**关于volatile实现防止指令重排**
  * instance = new DoubleCheckIdGenerator(); 并不是一个原子操作，其在JVM中至少做了三件事：
      1. 给instance在堆上分配内存空间。(分配内存)
      2. 调用DoubleCheckIdGenerator的构造函数等来初始化instance。（初始化）
      3. 将instance对象指向分配的内存空间。（执行完这一步instance就不是null了）
  * 在没有volatile修饰时，执行顺序可以是1,2,3，也可以是1,3,2。假设有两个线程，当一个线程先执行了3，还没执行2，此时第二个线程来到第一个check，发现instance不为null，就直接返回了，这就出现问题，这时的instance并没有完全完成初始化。

**使用 this 和 super 要注意的问题：**
 * 在构造器中使用 super() 调用父类中的其他构造方法时，该语句必须处于构造器的首行，否则编译器会报错。另外，this 调用本类中的其他构造方法时，也要放在首行。
 * this、super不能用在static方法中。  
*简单解释一下：*
被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享。而 this 代表对本类对象的引用，指向本类对象；而 super 代表对父类对象的引用，指向父类对象；所以， this和super是属于对象范畴的东西，而静态方法是属于类范畴的东西。
