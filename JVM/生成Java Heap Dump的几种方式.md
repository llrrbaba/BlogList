转自：https://blog.csdn.net/jijianshuai/article/details/79128033

### 生成Java Heap Dump的几种方式

#### Heap Dump 概述

Heap Dump的格式有很多种，而且不同的格式包含的信息也可能不一样。但总的来说，Heap Dump一般都包含了一个堆中的Java Objects, Class等基本信息。同时，当你在执行一个转储操作时，往往会触发一次GC，所以你转储得到的文件里包含的信息通常是有效的内容（包含比较少，或没有垃圾对象了） 。

#### Heap Dump 包含的信息

+ 所有的对象信息 
  对象的类信息、字段信息、原生值(int, long等)及引用值
+ 所有的类信息 
  类加载器、类名、超类及静态字段
+ 垃圾回收的根对象 
  根对象是指那些可以直接被虚拟机触及的对象
+ 线程栈及局部变量 
  包含了转储时刻的线程调用栈信息和栈帧中的局部变量信息

#### Heap Dump 获取方式

##### 1 使用 jmap 命令生成 dump 文件

> jmap -dump:live,format=b,file=d:\dump\heap.hprof <pid>

##### 2 使用 jcmd 命令生成 dump 文件

> jcmd <pid> GC.heap_dump d:\dump\heap.hprof

#####3 使用 JVM 参数获取 dump 文件

​	3.1 XX:+HeapDumpOnOutOfMemoryError 

​	当OutOfMemoryError发生时自动生成 Heap Dump 文件。

> 这可是一个非常有用的参数，因为当你需要分析Java内存使用情况时，往往是在OOM(OutOfMemoryError)发生时。

​	3.2 -XX:+HeapDumpBeforeFullGC 

​	当 JVM 执行 FullGC 前执行 dump。

​	3.3 -XX:+HeapDumpAfterFullGC 

​	当 JVM 执行 FullGC 后执行 dump。

​	3.4 -XX:+HeapDumpOnCtrlBreak 

​	交互式获取dump。在控制台按下快捷键Ctrl + Break时，JVM就会转存一下堆快照。

​	3.5 -XX:HeapDumpPath=d:\test.hprof 

​	指定 dump 文件存储路径。

> **注意：JVM 生成 Heap Dump 的时候，虚拟机是暂停一切服务的。如果是线上系统执行 Heap Dump 时需要注意**。

#### 使用其它工具获取dump文件

分析 Heap Dump 的工具都可以获取 Heap Dump 文件。 
比如：jdk 自带的工具 jvisualvm。 
其它工具：Eclipse memory analyzer（jmat）、JProfiler 等。

