# JVM 堆内存溢出后，其他线程是否可继续工作？

**答案是还能运行**。

代码如下：

```
public class JvmThread {

    public static void main(String[] args) {
        new Thread(() -> {
            List<byte[]> list = new ArrayList<byte[]>();
            while (true) {
                System.out.println(new Date().toString() + Thread.currentThread() + "==");
                byte[] b = new byte[1024 * 1024 * 1];
                list.add(b);
                try {
                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();

        // 线程二
        new Thread(() -> {
            while (true) {
                System.out.println(new Date().toString() + Thread.currentThread() + "==");
                try {
                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```

结果展示：

```
Wed Nov 07 14:42:18 CST 2018Thread[Thread-1,5,main]==
Wed Nov 07 14:42:18 CST 2018Thread[Thread-0,5,main]==
Wed Nov 07 14:42:19 CST 2018Thread[Thread-1,5,main]==
Wed Nov 07 14:42:19 CST 2018Thread[Thread-0,5,main]==
Exception in thread "Thread-0" java.lang.OutOfMemoryError: Java heap space
at com.gosaint.util.JvmThread.lambda$main$0(JvmThread.java:21)
at com.gosaint.util.JvmThread$$Lambda$1/521645586.run(Unknown Source)
at java.lang.Thread.run(Thread.java:748)
Wed Nov 07 14:42:20 CST 2018Thread[Thread-1,5,main]==
Wed Nov 07 14:42:21 CST 2018Thread[Thread-1,5,main]==
Wed Nov 07 14:42:22 CST 2018Thread[Thread-1,5,main]==
```

 

![img](https://img2018.cnblogs.com/blog/885859/201912/885859-20191222141243708-77838005.png)

 

　　上图是JVM堆空间的变化。我们仔细观察一下在14:42:05~14:42:25之间曲线变化，你会发现使用堆的数量，突然间急剧下滑！这代表这一点，当一个线程抛出OOM异常后，它所占据的内存资源会全部被释放掉，从而不会影响其他线程的运行！

　　**总结**：发生OOM的线程一般情况下会死亡，也就是会被终结掉，该线程持有的对象占用的heap都会被gc了，释放内存。因为发生OOM之前要进行gc，就算其他线程能够正常工作，也会因为频繁gc产生较大的影响。