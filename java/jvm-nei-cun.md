# JVM内存

### Method Area 方法区

方法区是被所有线程共享，所有字段和方法字节码，以及一些特殊方法如构造函数，接口代码也在此定义。简单 说，所有定义的方法的信息都保存在该区域，此区属于共享区间。

存放在方法区的:静态变量+常量+类信息\(构造方法/接口定义\)+运行时常量池存在方法区中 

ps:只要是被线程私有独享的一律没有会后,只有是线程共享才能有回收

### Stack 栈

栈也叫栈内存，主管Java程序的运行，是在线程创建时创建，它的生命期是跟随线程的生命期，线程结束栈内存也 就释放，对于栈来说不存在垃圾回收问题，只要线程一结束该栈就Over，生命周期和线程一致，是线程私有的。 

**栈中存储什么:**

* 8种基本类型的变量+对象的引用变量+实例方法都是在函数的栈内存中分配。

#### 栈的运行原理

栈中的数据都是以栈帧（Stack Frame）的格式存在，栈帧是一个内存区块，是一个数据集，是一个有关方法 \(Method\)和运行期数据的数据集，当一个方法A被调用时就产生了一个栈帧 F1，并被压入到栈中

![](../.gitbook/assets/image%20%289%29.png)

两个栈相关的常见错误

1. StackOverflowError：递归过深，递归没有出口。 
2. OutOfMemoryError：JVM空间溢出，创建对象速度高于GC回收速度。

### Heap 堆

一个JVM实例只存在一个堆内存，堆内存的大小是可以调节的。类加载器读取了类文件后，需要把类、方法、常变 量放到堆内存中，保存所有引用类型的真实信息，以方便执行器执行，堆内存分为三部分：

* Young Generation Space 新生区 
* Young/New Tenure generation space 养老区 
* Old/ Tenure Permanent Space 永久区 
* Perm Heap堆\(Java7之前\)

#### 新生区和养老区

新生区是类的诞生、成长、消亡的区域，一个类在这里产生，应用，最后被垃圾回收器收集，结束生命。新生区又 分为两部分： 伊甸区（Eden space）和幸存者区（Survivor pace） ，所有的类都是在伊甸区被new出来的。幸存 区有两个： 0区（Survivor 0 space）和1区（Survivor 1 space）。当伊甸园的空间用完时，程序又需要创建对 象，JVM的垃圾回收器将对伊甸园区进行垃圾回收\(Minor GC\)即轻量垃圾回收，将伊甸园区中的不再被其他对象所 引用的对象进行销毁。然后将伊甸园中的剩余对象移动到幸存 0区。若幸存 0区也满了，再对该区进行垃圾回收， 然后移动到 1 区。那如果1 区也满了呢？再移动到养老区。若养老区也满了，那么这个时候将产生 MajorGC（FullGC\)即重量垃圾回收，进行养老区的内存清理。若养老区执行了Full GC之后发现依然无法进行对象 的保存，就会产生OOM异常“OutOfMemoryError”。

#### 永久区

永久存储区是一个常驻内存区域，用于存放JDK自身所携带的 Class,Interface 的元数据，也就是说它存储的是运行 环境必须的类信息，被装载进此区域的数据是不会被垃圾回收器回收掉的，关闭 JVM 才会释放此区域所占用的内 存。

 如果出现java.lang.OutOfMemoryError: PermGen space，说明是Java虚拟机对永久代Perm内存设置不够。一般 出现这种情况，都是程序启动需要加载大量的第三方jar包。例如：在一个Tomcat下部署了太多的应用。或者大量 动态反射生成的类不断被加载，最终导致Perm区被占满。 

* Jdk1.6及之前： 有永久代, 常量池1.6在方法区
* Jdk1.7： 有永久代，但已经逐步“去永久代”，常量池1.7在堆 
* Jdk1.8及之后： 无永久代，常量池1.8在元空间

实际而言，方法区（Method Area）和堆一样，是各个线程共享的内存区域，它用于存储虚拟机加载的：类信息 +普通常量+静态常量+编译器编译后的代码等等，虽然JVM规范将方法区描述为堆的一个逻辑部分，但它却还有一 个别名叫做Non-Heap\(非堆\)，目的就是要和堆分开。 

对于HotSpot虚拟机，很多开发者习惯将方法区称之为“永久代\(Parmanent Gen\)” ，但严格本质上说两者不同，或 者说使用永久代来实现方法区而已，永久代是方法区\(相当于是一个接口interface\)的一个实现，jdk1.7的版本中， 已经将原本放在永久代的字符串常量池移走。 

ps:直白一点其实就是方法区就是永久带的一种落地实现 

常量池（Constant Pool）是方法区的一部分，Class文件除了有类的版本、字段、方法、接口等描述信息外，还有 一项信息就是常量池，这部分内容将在类加载后进入方法区的运行时常量池中存放。

![](../.gitbook/assets/image%20%281%29.png)

## 堆内存调优

JVM垃圾收集\(Java Garbage Collection \)本次均以JDK1.8+HotSpot为例:

### Java7

![](../.gitbook/assets/image%20%2814%29.png)

### Java8

![](../.gitbook/assets/image%20%2813%29.png)

### Java内存调优计算

|  |  |
| :--- | :--- |
| -Xms | 设置初始分配大小，默认为物理内存的1/64 |
| -Xmx | 最大分配大小，默认为物理内存的1/4 |
| -XX:+PrintGCDetails | 输出GC的详情记录 |
