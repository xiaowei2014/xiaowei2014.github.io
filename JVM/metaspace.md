# Metaspace

JDK1.7中，存储在永久代的部分数据就已经转移到了Java Heap或者是 Native Heap。

native heap：符号引用(Symbols)；

java heap：字面量(interned strings)；类的静态变量(class statics)。

我们可以通过一段程序来比较 JDK 1.6 与 JDK 1.7及 JDK 1.8 的区别，以字符串常量为例：

![image-20200125112008285](C:\Users\zhangxiaowei\AppData\Roaming\Typora\typora-user-images\image-20200125112008285.png)

JDK 1.6 的运行结果：

![img](https://images2015.cnblogs.com/blog/820406/201603/820406-20160327005929386-409283462.png)

JDK 1.7的运行结果：

![img](https://images2015.cnblogs.com/blog/820406/201603/820406-20160327010033823-1341228280.png)

JDK 1.8的运行结果：

![img](https://images2015.cnblogs.com/blog/820406/201603/820406-20160327010143776-1612977566.png)

　　从上述结果可以看出，JDK 1.6下，会出现“PermGen Space”的内存溢出，而在 JDK 1.7和 JDK 1.8 中，会出现堆内存溢出，并且 JDK 1.8中 PermSize 和 MaxPermGen 已经无效。因此，可以大致验证 JDK 1.7 和 1.8 将字符串常量由永久代转移到堆中，并且 JDK 1.8 中已经不存在永久代的结论。

　　元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制，但可以通过以下参数来指定元空间的大小：

　　-XX:MetaspaceSize，初始空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。
　　-XX:MaxMetaspaceSize，最大空间，默认是没有限制的。

　　除了上面两个指定大小的选项以外，还有两个与 GC 相关的属性：
　　-XX:MinMetaspaceFreeRatio，在GC之后，最小的Metaspace剩余空间容量的百分比，减少为分配空间所导致的垃圾收集
　　-XX:MaxMetaspaceFreeRatio，在GC之后，最大的Metaspace剩余空间容量的百分比，减少为释放空间所导致的垃圾收集

![image-20200125112359032](C:\Users\zhangxiaowei\AppData\Roaming\Typora\typora-user-images\image-20200125112359032.png)

现在我们在 JDK 8下重新运行一下代码段 4，不过这次不再指定 PermSize 和 MaxPermSize。而是指定 MetaSpaceSize 和 MaxMetaSpaceSize的大小。输出结果如下：

![img](https://images2015.cnblogs.com/blog/820406/201603/820406-20160327010233933-699106123.png)

从输出结果，我们可以看出，这次不再出现永久代溢出，而是出现了元空间的溢出。