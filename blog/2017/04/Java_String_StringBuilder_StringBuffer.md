# Java中的String，StringBuilder，StringBuffer三者的区别
在学习Java的过程中经常,遇到了这样一个问题，就是String,StringBuilder以及StringBuffer这三个类之间有什么区别呢，自己从网上搜索了一些资料，有所了解了之后在这里整理一下


执行速度，在这方面运行速度快慢为：StringBuilder > StringBuffer > String


　String最慢的原因：

　　String为字符串常量，而StringBuilder和StringBuffer均为字符串变量，即String对象一旦创建之后该对象是不可更改的，但后两者的对象是变量，是可以更改的

```java
String str="abc";
System.out.println(str);
str=str+"de";
System.out.println(str);
```

如果运行这段代码会发现先输出“abc”，然后又输出“abcde”，好像是str这个对象被更改了，其实，这只是一种假象罢了，JVM对于这几行代码是这样处理的，首先创建一个String对象str，并把“abc”赋值给str，然后在第三行中，其实JVM又创建了一个新的对象也名为str，然后再把原来的str的值和“de”加起来再赋值给新的str，而原来的str就会被JVM的垃圾回收机制（GC）给回收掉了，所以，str实际上并没有被更改，也就是前面说的String对象一旦创建之后就不可更改了。所以，Java中对String对象进行的操作实际上是一个不断创建新的对象并且将旧的对象回收的一个过程，所以执行速度很慢。

而StringBuilder和StringBuffer的对象是变量，对变量进行操作就是直接对该对象进行更改，而不进行创建和回收的操作，所以速度要比String快很多。


在线程安全上，StringBuilder是线程不安全的，而StringBuffer是线程安全的

如果一个StringBuffer对象在字符串缓冲区被多个线程使用时，StringBuffer中很多方法可以带有synchronized关键字，所以可以保证线程是安全的，但StringBuilder的方法则没有该关键字，所以不能保证线程安全，有可能会出现一些错误的操作。所以如果要进行的操作是多线程的，那么就要使用StringBuffer，但是在单线程的情况下，还是建议使用速度比较快的StringBuilder。


![](https://jayqiu.github.io/blog/2017/img/2014_04-11_09.jpg)


### 总结一下
　　String：适用于少量的字符串操作的情况,不可变字符串；

　　StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况,可变字符序列、效率高、线程不安全；

　　StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况,可变字符串、效率低、线程安全；