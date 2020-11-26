**项目的一些thinking**
#### 1
关于登录
* **question**：有一个账号C，用户A登录了之后，用户B再进行登录，输入错误的密码，连续输入三次之后锁定账号，此时用户A再进行操作会被踢出去吗？  
如果要求踢出去，那么用户B如果是恶意用户怎么办，他只需要知道账号就可以一直锁定账号C，A就一直登陆不上系统进行操作。

#### 2
* **question**：批量导入
  在postman中调用批量导入接口时，没有问题。但是在页面（前端）调用的时候出现nginx的错误，'An error occurred.'。
  ##### 未解决

#### 3
* **question**：关于Spring的@Async注解
  @Async 默认使用SimpleAsyncTaskExecutor，项目中使用默认的，生产上线程编号达到两百万时才被人发现，紧急处理
  Spring 已经实现的线程池
  1. SimpleAsyncTaskExecutor：不是真的线程池，这个类不重用线程，默认每次调用都会创建一个新的线程。
  2. SyncTaskExecutor：这个类没有实现异步调用，只是一个同步操作。只适用于不需要多线程的地方。
  3. ConcurrentTaskExecutor：Executor的适配类，不推荐使用。如果ThreadPoolTaskExecutor不满足要求时，才用考虑使用这个类。
  4. SimpleThreadPoolTaskExecutor：是Quartz的SimpleThreadPool的类。线程池同时被quartz和非quartz使用，才需要使用此类。
  5. ThreadPoolTaskExecutor ：最常使用，推荐。 其实质是对java.util.concurrent.ThreadPoolExecutor的包装。

#### 4
* **question**：关于springboot日志问题。
  需要对日志做处理，日志输出路径，日志防爆盘。项目里日志框架用的logback框架。
  在logback-spring.xml文件里，
  <property name="LOG_HOME" value="${logPath:-/log/xxx}/xxx" /> 可以配置一个路径映射，<file>${LOG_HOME}/run/run.log</file>
  防爆盘处理，
  <fileNamePattern>${LOG_HOME}/%d{yyyy-MM}/run.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern> 打包文件名
  <maxFileSize>20MB</maxFileSize> 日志文件最大size
  <maxHistory>30</maxHistory> 最多多少个日志文件
  <totalSizeCap>200MB</totalSizeCap>  最大总共打包size
  
  **ps**：**question**：最近发现个有趣的问题，日志压测的时候，13号压测发现日志压缩的时候都是10号的日志
    原因：logback一直用的一个线程写，意思就是，从10号项目启动的时候，这个日志文件就在编辑状态，所以日志文件的编辑日期一直停留在10号。

#### 5
* **question**：关于mybatis的映射问题，实体属性是驼峰，数据库字段是下划线。
  * 数据库返回结果，映射到实体：
    要么实体用下划线形式的属性名
    要么mybatis加个配置，以springboot为例：
    ~~~
      mybatis:
        configuration:
          mapUnderscoreToCamelCase: true
    ~~~
  * mapper映射传入参数为对象时：
    在对象的属性上添加@JsonProperty注解没有用，需要mapper.xml里的传入字段名是跟对象属性名是一样的
    
#### 6
* **question**：logback的配置项totalSizeCap没有生效。  
  * 
