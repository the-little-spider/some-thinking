* 关于volatile实现防止指令重排
  * instance = new DoubleCheckIdGenerator(); 并不是一个原子操作，其在JVM中至少做了三件事：
      1. 给instance在堆上分配内存空间。(分配内存)
      2. 调用DoubleCheckIdGenerator的构造函数等来初始化instance。（初始化）
      3. 将instance对象指向分配的内存空间。（执行完这一步instance就不是null了）
  * 在没有volatile修饰时，执行顺序可以是1,2,3，也可以是1,3,2。假设有两个线程，当一个线程先执行了3，还没执行2，此时第二个线程来到第一个check，发现instance不为null，就直接返回了，这就出现问题，这时的instance并没有完全完成初始化。
