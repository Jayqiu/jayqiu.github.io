#  Android虚拟机DVM、ART和JAVA虚拟机JVM的区别  


Dalvik和标准Java虚拟机（JVM）之间的首要差别之一，就是Dalvik基于寄存器，而JVM基于栈。

Dalvik和Java之间的另外一大区别就是运行环境——Dalvik经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个 Dalvik应用作为一个独立的Linux进程执行。独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。

## JVM和DVM的主要区别如下:

       Java：
             .java文件 -> .class文件 -> .jar文件
        Android：

             .java文件 –> .class文件 -> .dex文件


![](https://jayqiu.github.io/blog/2017/img/20171104122858683.png)


如上图所示，.jar文件里面包含多个.class文件，每个.class文件里面包含了该类的头信息（如编译版本）、常量池、类信息、域、方法、属性等等，当JVM加载该.jar文件的时候，会加载里面的所有的.class文件，这样会很慢，而移动设备的内存本来就很小，不可能像JVM这样加载，所以它使用的不是.jar文件，而是.apk文件，该文件里面只包含了一个.dex文件，这个.dex文件里面将所有的.class里面所包含的信息全部整合在一起了，这样再加载就很快了。.class文件存在很多的冗余信息，dex工具会去除冗余信息，并把所有的.class文件整合到.dex文件中。减少了I/O操作，提高了类的查找速度。


b) 基于的架构不一样

Java基于栈的架构.栈是内存上面的一段连续的存储空间

Android基于寄存器的架构.寄存器是CPU上面的一块存储空间

所以，CPU直接访问自己上面的一块空间的数据的效率肯定要大于访问内存上面的数据


## ART 与Dalvik比较  

Dalvik就是android 4.4及职权使用的虚拟机。

ART之所以会比Dalvik快，是因为ART执行的是本地机器指令，而Dalvik执行的是Dex字节码，通过解释器执行。尽管Dalvik也会对频繁执行的代码进行JIT生成本地机器指令来执行，但毕竟在应用程序运行的过程中将Dex字节码翻译成本地机器指令也会影响到应用程序本身的执行，因此即使Dalvik使用了JIT，也在一定程度上也比不上直接就可以执行本地机器指令的运行时。



1、ART有很多GC方式可以选择，而Dalvik只有一种GC方式。 
2、ART通过额外记录新分配的对象来支持更好的轻量级Sticky GC。 
ART中新增的轻量级回收kGcTypeSticky只回收上次GC后再Allocation Space中新分配的垃圾对象，Heap 类的成员函数RecordAllocation首先是记录当前已经分配的内存字节数以及对象数，为了Sticky GC，还要接着再将新分配的对象压入到Heap类的成员变量allocation_stack_描述的Allocation Stack中去，而Dalvik虚拟机直接将新分配出来的对象记录在Live Bitmap中。如果不能成功将新分配的对象压入到Allocation Stack中，就说明上次GC以来，新分配的对象太多了，因此这时候就需要执行一个Sticky GC，将Allocation Stack里面的垃圾进行回收，然后再尝试将新分配的对象压入到Allocation Stack中，直到成功为止。 
3、大内存块额外单独管理，可以提高内存分配和回收效率。 
4、减少Suspend时间。 
对于并行GC，在Handle Dirty Object阶段是在挂起ART运行时线程的前提下进行的，因此，如果把所有的Dirty Card 都放在Handle Dirty Object阶段处理，那么就会可能会造成应用程序停顿时间过长。于是，ART运行时就在并行Marking阶段也帮忙着处理Dirty Card，通过这种方式尽量减少在Handle Dirty Object阶段需要处理的Dirty Card，以达到减少应用程序因为GC造成的停顿时间。另外ART在垃圾回收过程中，也减少了挂起非GC线程的次数。