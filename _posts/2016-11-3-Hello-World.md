---
layout: post
title: "Spring Cloud Config"
comments: true
description: "spring cloud config"
keywords: " spring cloud config"
---


现今这个时候，微服务大行其道，互联网应用遍地都是，随便开发个什么应用首要考虑的都是要可伸缩，扩展性要好。   
当我们的后台服务一点点增多，各个服务的配置也越来越多，随之而来的就是一个配置管理问题，各自管各自的开发时没什么问题，到了线上之后管理就会很头疼，到了要大规模更新的时候就更烦了。   
我们的后台服务就是如此，各种语言开发的都有，在慢慢的迭代过程的我们发现配置中心是一个比较好的解决方案，作为 Spring 的拥趸我们自然就看中了 Spring Cloud Config。     
一般服务器的应用都有以下几种类型:
 ![图片](https://dn-coding-net-production-pp.qbox.me/b6e1ba88-36a7-4fa6-ba93-2b0e501e6c1b.png) 
其中当属业务部分最多也最繁杂。
当应用变得越来越庞大和复杂时，单机就肯定不能满足需求了，然后就要考虑分布式了，接下去还可能会应用不同的语言来开发应用。   

 ![图片](https://dn-coding-net-production-pp.qbox.me/c2b3c8e9-4054-4eaa-9071-1300a731340e.png) 
比如 nginx 毫无疑问的是用的最多的反向代理组件，使用 OpenResty 便要用到 lua，再比如前端要 seo ，一个解决办法就是使用 nodejs，到了后端分布式，那就更繁多了，可能会需要把业务一个个拆分成不同的服务单独运行...   
然后就发现散落一地的配置文件，**properties**、**yml**、**json** …

为了解决这个问题，我们采取了这么一种方案，通过 etcd 来收集所有的配置，然后统一的写到一个文件中去，任何应用都来访问这一个文件来找到自己的配置并应用。
 ![图片](https://dn-coding-net-production-pp.qbox.me/3905cf7f-9fde-4460-b7d6-1f8724a7dda9.png) 

这很好的解决了配置散落的问题，可以集中管理了，但是又一个问题来了，各种类型的配置在一个文件里看起来好乱，而且在初始化时必须要完成这个文件才能启动应用。

### 所以下一个解决方案便想到用一个配置中心来搞定。
  ![图片](https://dn-coding-net-production-pp.qbox.me/9393a355-cd47-432a-85bf-05e6e8766b7c.png) 

# Spring Cloud Config 项目
- 提供 **服务端** 和 **客户端** 支持
- **集中式** 管理分布式环境下的应用配置
- 基于 **Spring** 环境，无缝 与 Spring 应用集成
- 可用于 **任何** 语言开发的程序
- 默认实现基于 **git** 仓库，可以进行 **版本管理**
- **可替换** 自定义实现

### Spring Cloud Config Server 作为配置中心服务端
* 拉取配置时更新 git 仓库副本，保证是最新结果
* 支持数据结构丰富，yml, json, properties 等
* 配合 eureke 可实现服务发现，配合 cloud bus 可实现配置推送更新
* 配置存储基于 git 仓库，可进行版本管理
* 简单可靠，有丰富的配套方案

### Spring Cloud Config Client 默认客户端实现
- SpringBoot 项目不需要改动任何代码，加入一个启动配置文件指明使用 ConfigServer 上哪个配置文件即可


# 简单使用示例
1. 新建一个 git 仓库，添加一个配置文件。例如想要一个 billing的服务，性质是开发，运行环境是测试环境。
>那么就新建一个 testing 的分支，然后提交一个 billing-dev.properties 的文件
>```
devMode = true
spring.application.name = billing
spring.jdbc.host = localhost
spring.jdbc.port = 3306
spring.jdbc.user = root
spring.jdbc.password = 123qwe
loging.file = demo
>```

2. 新建一个标准 maven 项目
>![图片](https://dn-coding-net-production-pp.qbox.me/fb006d98-a00b-40d6-930a-64e14a3ea9dc.png) 
>**ConfigServer.java**
>
>```
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServer.class, args);
    }
}
>``` 
> **application.yml**
>```
server:
  port: 8888
spring:
  cloud:
    config:
      server:
        git:
          uri:https://git.coding.net/tiangao/demo-config-server.git
          clone-on-start: true
>```
>
> **pom.xml** 
>
>```
><?xml version="1.0" encoding="UTF-8"?>
><project xmlns="http://maven.apache.org/POM/4.0.0"
>         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
>    <modelVersion>4.0.0</modelVersion>
>    <artifactId>config-server</artifactId>
---
layout: post
title: "Java 文件句柄泄露问题之 file-leak-detector"
comments: false
published: false
description: " java file leak"
keywords: "java file leak file-leak-detector"
---

> 维护 WEB-IDE 免不了要管理很多的文件， 自从我们 线上的系统增加了资源回收功能，便一直受一个问题困扰，后台线程解绑目录时偶尔报错，看症状因为是某些文件被占用了，目录不能解绑。但是由于系统中很多地方都有打开文件，各种包也存在复杂的的引用关系，在搜查几遍代码后并没有发现什么明显的异常。

由于这个功能清理的是既没在线又没有在离线列表中的磁盘绑定目录，那么很可能是文件句柄泄露了，还有一种原因可能是 JVM 延迟释放文件句柄，不过实际是什么原因还需要用数据说话。

经过一番搜索，发一个工具叫 file-leak-detector， 可以监控什么线程在什么时候打开了哪儿的文件，看起来好酷，官网在这里：  
[http://file-leak-detector.kohsuke.org](http://file-leak-detector.kohsuke.org)

## 使用方式

### 监听 HTTP 端口方式启动   
以 javaagent 方式启动一个 jar 文件，输出在 http 19999 端口。
```
$java -javaagent:./file-leak-detector-1.8-jar-with-dependencies.jar=http=19999 -jar ide-backend.jar
```
然后直接在浏览器访问刚刚启动时配置的 http端口：
 ![图片](https://dn-coding-net-production-pp.qbox.me/fc6fdcc0-bf03-499a-bb46-2834e6aa9ba9.png) 
可以看到当前所有打开中的文件的堆栈信息。

### 配置参数启动

配置线程数量限制,在文件句柄持有数超过设定数值时输出所有文件打开时的堆栈信息到 System 的 err 日志中。
```
$ java -javaagent:path/to/file-leak-detector.jar=threshold=200 ...your usual Java args follows...
```

### Attach 方式启动

启动后直接被加载到运行中的 JAVA 进程里。
```
$ java -jar path/to/file-leak-detector.jar 1500 threshold=200,strong
```
strong 代表的含义是把记录信息的应用变成强引用，防止被 GC 回收掉，不设置在内存不足时文件记录会丢失。

## 实际使用体验

首先我们在测试服务器上部署端口来监控，然后进行各种测试，最后确实找到几处未关闭的代码。
```
$java -javaagent:./file-leak-detector-1.8-jar-with-dependencies.jar=http=19999 -jar xxx.jar
```
不过有一点比较不爽，绑定的地址是固定的 localhost, 远程的就不能访问。
```
╭─tiangao@tgmbp  ~/git/tiangao  ‹master*›
╰─$ curl 192.168.31.227:19999
curl: (7) Failed to connect to 192.168.31.227 port 19999: Connection refused
```
这个先放一边，官网说还可以 attach 到正在运行的进程中，这点才是我们到线上监控所需要的，有些问题只有在线上才会出现。

不过官网里并没有发现怎么挂到正在运行中的 java 程序并开启 http 端口输出，而且监听的端口只有 localhost。这就让我们感觉有点怪异，
也许有安全性的考量吧，只好去看看源码，才知道怎么个用法，为了更方便还改了下监听的 host，以便远程可以访问。

**AgentMain.java**
``` 
private static void runHttpServer(int port) throws IOException {
    final ServerSocket ss = new ServerSocket();
    ss.bind(new InetSocketAddress("0.0.0.0", port));
    System.err.println("Serving file leak stats on http://0.0.0.0:"+ss.getLocalPort()+"/ for stats");
    ...
}
```
改之后使用如下所示:
```
root@staging-1:~# java -jar file-leak-detector-1.8-jar-with-dependencies.jar 612 http=19999
Connecting to 612
Activating file leak detector at /root/file-leak-detector-1.8-jar-with-dependencies.jar
```
`612` 是 java 服务的进程号，19999 是监听的 `http` 端口号。  

执行后输出类似如下内容时即表示 `attach` 到进程成功。
```
╭─tiangao@tgmbp  ~/git/WebIDE-Backend/target  ‹master*›
╰─$ java -jar file-leak-detector-1.8-jar-with-dependencies.jar 93739
Connecting to 93739
Activating file leak detector at /Users/shitiangao/git/WebIDE-Backend/target/file-leak-detector-1.8-jar-with-dependencies.jar
```

然后通过地址加端口就可以访问,就可以显示进程在 `attach` 之后打开的文件以及相应堆栈信息。
```
3 descriptors are open
#1 /opt/coding-ide-home/ide-backend.jar by thread:qtp873134840-16 on Tue Nov 29 15:05:34 CST 2016
	at java.io.RandomAccessFile.<init>(RandomAccessFile.java:244)
	at org.springframework.boot.loader.data.RandomAccessDataFile$FilePool.acquire(RandomAccessDataFile.java:252)
	at org.springframework.boot.loader.data.RandomAccessDataFile$DataInputStream.doRead(RandomAccessDataFile.java:174)
	at org.springframework.boot.loader.data.RandomAccessDataFile$DataInputStream.read(RandomAccessDataFile.java:152)
```

如此改动测试后在本地好用，但是一到线上部署就报错了：
```
pid: 13546
Connecting to 13546
Exception in thread "main" java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.kohsuke.file_leak_detector.Main.run(Main.java:54)
	at org.kohsuke.file_leak_detector.Main.main(Main.java:39)
Caused by: com.sun.tools.attach.AttachNotSupportedException: Unable to open socket file: target process not responding or HotSpot VM not loaded
	at sun.tools.attach.LinuxVirtualMachine.<init>(LinuxVirtualMachine.java:106)
	at sun.tools.attach.LinuxAttachProvider.attachVirtualMachine(LinuxAttachProvider.java:63)
	at com.sun.tools.attach.VirtualMachine.attach(VirtualMachine.java:208)
	... 6 more
```

目测原因是 JVM 运行时反射加载不到类。

第一感觉需要设置一下 JAVA_HOME, 然而结果证明并不是这个原因。

万能的 google & stackoverflow 找到了解法：  
[java - AttachNotSupportedException due to missing java_pid file in Attach API](http://stackoverflow.com/questions/5769877/attachnotsupportedexception-due-to-missing-java-pid-file-in-attach-api)

执行 attach 的用户需要和 Java 服务运行用户是同一个，另外 JAVA_HOME 环境变量还是需要的。

终于成功了，接下来就是等待错误的再次发生，然后分析堆栈信息了。

***如此好用的工具是让我们对其原理很好奇***。

## 工作原理

项目源码并不是太多，先看 main ：   
```
public static void main(String[] args) throws Exception {
        Main main = new Main();
        CmdLineParser p = new CmdLineParser(main);
        try {
            p.parseArgument(args);
            main.run();
        } catch (CmdLineException e) {
            System.err.println(e.getMessage());
            System.err.println("java -jar file-leak-detector.jar PID [OPTSTR]");
            p.printUsage(System.err);
            System.err.println("\nOptions:");
            AgentMain.printOptions();
            System.exit(1);
        }
    }
```

来到 run() 方法，
```
    public void run() throws Exception {
        Class api = loadAttachApi();

        System.out.println("Connecting to "+pid);
        Object vm = api.getMethod("attach",String.class).invoke(null,pid);

        try {
            File agentJar = whichJar(getClass());
            System.out.println("Activating file leak detector at "+agentJar);
            // load a specified agent onto the JVM
            api.getMethod("loadAgent",String.class,String.class).invoke(vm, agentJar.getPath(), options);
        } finally {
            api.getMethod("detach").invoke(vm);
        }
    }
```

通过 `loadAttachApi()` 得到 `VirtualMachine 类`，然后再用反射获取 `attach()` 方法，紧接着执行 `attach()` 到指定进程 id 上，得到 vm 的实例后执行 `loadAgent()` 方法，第一个参数为 **agentJar** 包的路径，第二个 **options** 是附加参数。

`loadAttachApi()` 方法如下：

```
private Class loadAttachApi() throws MalformedURLException, ClassNotFoundException {
    File toolsJar = locateToolsJar();

    ClassLoader cl = wrapIntoClassLoader(toolsJar);
    try {
        return cl.loadClass("com.sun.tools.attach.VirtualMachine");
    } catch (ClassNotFoundException e) {
        throw new IllegalStateException("Unable to find tools.jar at "+toolsJar+" --- you need to run this tool with a JDK",e);
    }
}
```

问题来了，`VirtualMachine` 是个什么功能的类？ `attach()` `loadAgent()` 又是什么作用呢？

这个就涉及到 JVM 层面提供的功能，在这之前也没有研究过，只好看看大拿的研究。  

[InfoQ JVM源码分析之javaagent原理完全解读](http://www.infoq.com/cn/articles/javaagent-illustrated)  

关键类 Instrument:

[Package java.lang.instrument](http://docs.oracle.com/javase/7/docs/api/java/lang/instrument/package-summary.html)

**简单总结，JVM 暴露了一些动态操作已加载类型的接口，javaagnet 就是利用这些接口的一个实现，通过 agent 类的固定方法可以执行一些操作，比如对已经加载的类注入字节码，最常用的是用来监控运行时，进行一些疑难 bug 追踪。**

此项目里 TransformerImpl 类就是字节码修改的实现类。   

关键源码：    
```
instrumentation.addTransformer(new TransformerImpl(createSpec()),true);
        
instrumentation.retransformClasses(
        FileInputStream.class,
        FileOutputStream.class,
        RandomAccessFile.class,
        Class.forName("java.net.PlainSocketImpl"),
        ZipFile.class);
```
可以看到注册的类有 FileInputStream、FileOutputStream、RandomAccessFile、ZipFile 和 PlainSocketImpl。
   
```
static List<ClassTransformSpec> createSpec() {
    return Arrays.asList(
        newSpec(FileOutputStream.class, "(Ljava/io/File;Z)V"),
        newSpec(FileInputStream.class, "(Ljava/io/File;)V"),
        newSpec(RandomAccessFile.class, "(Ljava/io/File;Ljava/lang/String;)V"),
        newSpec(ZipFile.class, "(Ljava/io/File;I)V"),

        /*
            java.net.Socket/ServerSocket uses SocketImpl, and this is where FileDescriptors
            are actually managed.

            SocketInputStream/SocketOutputStream does not maintain a separate FileDescritor.
            They just all piggy back on the same SocketImpl instance.
         */
        new ClassTransformSpec("java/net/PlainSocketImpl",
                // this is where a new file descriptor is allocated.
                // it'll occupy a socket even before it gets connected
                new OpenSocketInterceptor("create", "(Z)V"),

                // When a socket is accepted, it goes to "accept(SocketImpl s)"
                // where 's' is the new socket and 'this' is the server socket
                new AcceptInterceptor("accept","(Ljava/net/SocketImpl;)V"),

                // file descriptor actually get closed in socketClose()
                // socketPreClose() appears to do something similar, but if you read the source code
                // of the native socketClose0() method, then you see that it actually doesn't close
                // a file descriptor.
                new CloseInterceptor("socketClose")
        ),
        new ClassTransformSpec("sun/nio/ch/SocketChannelImpl",
                new OpenSocketInterceptor("<init>", "(Ljava/nio/channels/spi/SelectorProvider;)V"),
                new CloseInterceptor("kill")
        )
    );
}
```

`ClassTransformSpec` 定义：   
```
    /**
     * Creates {@link ClassTransformSpec} that intercepts
     * a constructor and the close method.
     */
    private static ClassTransformSpec newSpec(final Class c, String constructorDesc) {
        final String binName = c.getName().replace('.', '/');
        return new ClassTransformSpec(binName,
            new ConstructorOpenInterceptor(constructorDesc, binName),
            new CloseInterceptor()
        );
    }
```
关键真相在这里，实现了一个方法拦截适配器，在注册类的某些方法执行后运行 `Listener` 类的 `open()` 方法来记录信息。
```
    /**
     * Intercepts the this.open(...) call in the constructor.
     */
    private static class ConstructorOpenInterceptor extends MethodAppender {
        /**
         * Binary name of the class being transformed.
         */
        private final String binName;

        public ConstructorOpenInterceptor(String constructorDesc, String binName) {
            super("<init>", constructorDesc);
            this.binName = binName;
        }

        @Override
        public MethodVisitor newAdapter(MethodVisitor base, int access, String name, String desc, String signature, String[] exceptions) {
            final MethodVisitor b = super.newAdapter(base, access, name, desc, signature, exceptions);
            return new OpenInterceptionAdapter(b,access,desc) {
                @Override
                protected boolean toIntercept(String owner, String name) {
                    return owner.equals(binName) && name.startsWith("open");
                }

                @Override
                protected Class<? extends Exception> getExpectedException() {
                    return FileNotFoundException.class;
                }
            };
        }

        protected void append(CodeGenerator g) {
            g.invokeAppStatic(Listener.class,"open",
                    new Class[]{Object.class, File.class},
                    new int[]{0,1});
        }
    }
 ```
 
最后的 `append()` 方法可以说是整个流程中最核心的地方，`Listener#open()` 方法如下所示：
 ```
 public static synchronized void open(Object _this, File f) {
    put(_this, new Listener.FileRecord(f));
    Iterator i$ = ActivityListener.LIST.iterator();

    while(i$.hasNext()) {
        ActivityListener al = (ActivityListener)i$.next();
        al.open(_this, f);
    }
}
```
 最后说 一下 **`Listener`** 这个类，这也是这个工具的一个关键的类实现，有许多静态方法，所有监控的打开的文件相关内容都在 `Listener` 中保存，内容输出的操作也在其中。

这是类中的属性和方法:   
 ![图片](https://dn-coding-net-production-pp.qbox.me/b34f6f31-d3fb-42eb-a3d5-535fdd45edfb.png) 

TABLE 保存打开中的文件，默认是 weak 引用，内存不足时这个对象会被回收掉，以防止程序不会因为监控导致的内存不足而异常退出。   
当参数 strong 存在时会 new 一个 LinkedHashMap, 让监控内容的容器不会被回收掉。
 ```
 /**
 * Files that are currently open, keyed by the owner object (like {@link FileInputStream}.
 */
private static Map<Object,Record> TABLE = new WeakHashMap<Object,Record>();
 ```
 Record 中有三个字段，一个是用来保存堆栈信息的异常类型，一个是线程名，最后一个是时间。
 ```
/**
 * Remembers who/where/when opened a file.
 */
public static class Record {
    public final Exception stackTrace = new Exception();
    public final String threadName;
    public final long time;
    ...
}
 ```

到这里已经差不多了，其他细节实现也就不赘述了。

## 小结

**file-leak-detector** 查找文件句柄泄露问题，就是用 JVM 提供的接口，以 agent 方式 attach 进正在运行的 JAVA 进程，修改 `FileStream` 等类型的字节码，在 open & close 文件时加入拦截操作，记录线程和堆栈，然后在 http 或者 系统日志中输出记录。   
最后通过这些信息查找是哪里导致的问题，然后做针对性的修复。


