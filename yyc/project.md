**项目的一些thinking**
#### 1
关于登录
* question：有一个账号C，用户A登录了之后，用户B再进行登录，输入错误的密码，连续输入三次之后锁定账号，此时用户A再进行操作会被踢出去吗？  
如果要求踢出去，那么用户B如果是恶意用户怎么办，他只需要知道账号就可以一直锁定账号C，A就一直登陆不上系统进行操作。

#### 2
* question：批量导入
  在postman中调用批量导入接口时，没有问题。但是在页面（前端）调用的时候出现nginx的错误，'An error occurred.'。

#### 3
* question：关于Spring的@Async注解
  @Async 默认使用SimpleAsyncTaskExecutor，项目中使用默认的，生产上线程编号达到两百万时才被人发现，紧急处理
  Spring 已经实现的线程池
  1. SimpleAsyncTaskExecutor：不是真的线程池，这个类不重用线程，默认每次调用都会创建一个新的线程。
  2. SyncTaskExecutor：这个类没有实现异步调用，只是一个同步操作。只适用于不需要多线程的地方。
  3. ConcurrentTaskExecutor：Executor的适配类，不推荐使用。如果ThreadPoolTaskExecutor不满足要求时，才用考虑使用这个类。
  4. SimpleThreadPoolTaskExecutor：是Quartz的SimpleThreadPool的类。线程池同时被quartz和非quartz使用，才需要使用此类。
  5. ThreadPoolTaskExecutor ：最常使用，推荐。 其实质是对java.util.concurrent.ThreadPoolExecutor的包装。
