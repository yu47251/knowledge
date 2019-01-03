# JVM 知识点

## 1 垃圾回收机制
- 垃圾回收机制，主要针对：堆的内存进行回收。因为此两个区域的内存在JVM运行过程中会有积累，占用内存，不回收会越来越多。
- 程序计数器、虚拟机栈、本地方法栈这三个内存区域跟随线程生死，所以无需回收。

### 1.1 对象存活判断
#### 1.1.1 计数器：
- 每增加一个引用计数器+1，引用消失计数器-1
- 优点：效率高。
- 缺点：无法解决对象间的相互引用问题。

#### 1.1.2 可达性分析
通过GC ROOTS对象作作为起点，从这些起点向下去搜索，所走过的路径叫做引用链，当一个对象到GC ROOTS没有任何引用链连接时，则证明此对象不可达。
##### 1.1.2.1 GC ROOTS对象
- 虚拟机栈(栈桢中的本地变量表)中的引用的对象
- 方法区中的类静态属性引用的对象
- 方法区中的常量引用的对象
- 本地方法栈中JNI的引用的对象

##### 1.1.2.2 最终判定对象是否被回收
在可达性分析算法中不可达的对象，也并非就判定为“死亡”了；判定一个对象最终是否死亡，至少经历两次标记过程：

- 不可达的对象会被第一次标记；
- 第一次标记的对象，判断是否重写了FINALIZE方法，如果重写了而且没有执行过，则执行FINALIZE方法。
- 在FINALIZE()中，若对象重新和引用链上的任一对象建立关联，则不会被第二次标记，它则不会被回收。

### 1.2 垃圾收集器
因为每次在进行FULL GC的时候会触发STW(STOP THE WORLD)，垃圾收集器的优化的主要目的是为了减少STW的次数和减少每次STW的持续时间。
#### 1.2.1 Serial
- JDK1.3开始使用
- 对于新生代的使用的是复制算法，
- 对于老年代使用的是标记压缩算法。
- 设置方式：-XX:+UseSerialGC
#### 1.2.2 ParNew
- Serial的多线程版本, JDK1.4开始使用
- 新生代和老年代的算法与Serial算法一致，
- 设置方式：-XX:+UseParNewGC
- 设置线程数--XX:ParallelGCThreads
#### 1.2.3 Parallel Scavenge
- JDK1.4开始使用
- 关注吞吐量（运行用户程序所占的比重），主要针对CPU密集型应用，可以高效利用CPU 
- -XX:+UseParallelGC
- -XX:MaxGCPauseMillis 最大GC暂停毫秒数 
- -XX:GCTimeRatio GC时间所占比重 
- 收集器将尽力保证GC时间不超过设定值，但是这是以牺牲吞吐量和新生代空间换取的：新生代越小收集时间越小，导致GC更频繁，吞吐量也就下降了。 
- -XX:+UseAdaptiveSizePolicy 开关参数，开启后不需要手动指定-Xmn、-XX:SurvivorRatio、
- -XX:PretenureSizeThreshold等参数，jvm会动态调整
#### 1.2.4 Parallel Old
- JDK1.6开始使用
- Parallel Scavenge收集器的老年代版本，多线程和“标记-整理”算法。
- 在注重吞吐量和CPU资源敏感场合，优先考虑 Parallel Scavenge和Parallel Old。
- 设置方式：-XX:+UseParallelOldGC
#### 1.2.5 Serial Old
- JDK1.4, 与Parallel Scavenge联合使用
- Serial收集器的老年代版本，单线程收集器，使用“标记-整理”算法，主要用于Client模式。
- 另外一个主要作用：作为CMS收集器的后备预案，当发生Concurrent Mode Failure时使用。
#### 1.2.6 CMS
JDK1.5开始使用，关注最短停顿时间为目标的收集器，注重服务响应速度，更好的用户体验。

- 使用“标记-清除”算法。
- 分为4个步骤初始标记、并发标记、重新标记、并发清除 
- 其中初始标记、重新标记仍需要“Stop The World”，耗时长的并发标记和并发清除是可以与用户线程一起工作。 

并发低停顿收集器缺点： 

- 对CPU资源非常敏感。默认启动线程数是（CPU数+3）/ 4。当CPU在4个以上时，并发回收最多占用不超过25%的CPU资源，但是当CPU不足4个时，差不多占用一半的CPU资源，如果此时负载较大时，对用户线程来说很有压力。 
- 存在失败风险，从而导致另外一次fullGC，并且会临时使用Serial Old收集器。因为CMS工作时，需要预留足够的内存空间给用户线程使用，默认在68%时激活GC。 
- 清除算法容易导致空间碎片。碎片过多会导致在老年代还有很多剩余空间时大对象无法分配，不得不提前触发FullGC。 
- -XX:+UseCMSCompactAtFullCollection 开启后，在FullGC后进行碎片整理，会增大停顿时间 
- -XX:CMSFullGCsBeforeCompaction 设置多少次FullGC后进行一次压缩处理。
- -XX:CMSInitiatingOccupancyFraction 触发GC的百分比，适当提高，可以降低回收次数，设的太高很容易导致“Concurrent Mode Failure”，性能反而下降 
## 2 内存模型