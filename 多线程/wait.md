# 线程间通信 Object/wait(),notify() 和 Lock/Condition/await(),signal()

https://www.cnblogs.com/myseries/p/12103723.html

**一：Object/wait(), notify(), notifyAll()**

　notify()方法唤醒等待锁对象的其他线程，这个notify()方法需要先执行完自己线程的后续代码，才会接着执行被唤醒的那个线程的代码：

```
public class TestWaitAndnotify {

    public static void main(String[] args) {
        demo2();
    }
    
    public static void demo2 () {
        final Object lock = new Object();
        Thread A = new Thread(new Runnable(){

            @Override
            public void run() {
                System.out.println("INFO: A 等待锁 ");
                synchronized (lock) {
                    System.out.println("INFO: A 得到了锁 lock");
                    System.out.println("A1");
                    try {
                        System.out.println("INFO: A 准备进入等待状态，放弃锁 lock 的控制权 ");
                        lock.wait();//挂起线程A 放弃锁 lock 的控制权
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("INFO: 有人唤醒了 A, A 重新获得锁 lock");
                    System.out.println("A2");
                    System.out.println("A3");
                }
            }
        });
        
        Thread B = new Thread(new Runnable() {

            @Override
            public void run() {
                 System.out.println("INFO: B 等待锁 ");
                synchronized (lock) {
                    System.out.println("INFO: B 得到了锁 lock");
                    System.out.println("B1");
                    System.out.println("B2");
                    System.out.println("B3");
                    System.out.println("INFO: B 打印完毕，调用 notify 方法 ");
                    lock.notify(); // notify()方法唤醒正在等待lock锁的线程A
                    try {
                        Thread.sleep(10000); // 睡眠10秒钟
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("线程 B do notify method 完毕");
                }
            }
        });
        
        A.start();
        B.start();
    } 
}
```

![img](https://img2018.cnblogs.com/blog/885859/201912/885859-20191227093853611-1024696486.png)

 

打印完：INFO: B 打印完毕，调用 notify 方法 后面等待了10秒钟才开始执行。wait()方法是要等待notify()后续的代码才能开始。

 

**一：Lock/condition/await(), signal(), signalAll()**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ProducerAndConsumer1 {
    
    private final int MAX_LEN = 3;
    private Queue<Integer> queue = new LinkedList<Integer>();

    final Lock lock = new ReentrantLock();
    final Condition producerSignal = lock.newCondition();
    final Condition consumerSignal = lock.newCondition();

    class Producer extends Thread {
        @Override
        public void run() {
            producer();
        }
        
        private void producer() {
            while(true) {
                lock.lock();
                try {
                    if(queue.size() == MAX_LEN) {
                        System.out.println("当前队列满");
                        try {
                            consumerSignal.signal();
                            producerSignal.await();    
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.add(1);
                    consumerSignal.signal();
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("生产者生产了一条任务，当前队列长度为" + queue.size());
                    
                } finally {
                    lock.unlock();
                }
            }

        }
    }
    
    class Consumer extends Thread {
        @Override
        public void run() {
            consumer();
        }
        
        private void consumer() {
            while(true) {
                lock.lock();
                try {
                    if(queue.size() == 0) {
                        System.out.println("当前队列为空");
                        try {
                            producerSignal.signal();
                            consumerSignal.await();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    queue.poll();
                    producerSignal.signal();
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("消费者消费了一条任务，当前队列长度为" + queue.size());
                    
                } finally {
                    lock.unlock();
                }
            }

        }
    }
    
    public static void main(String[] args) {
        ProducerAndConsumer1 pac = new ProducerAndConsumer1();
        
        Producer producer = pac.new Producer();
        Consumer consumer = pac.new Consumer();
        producer.start();
        consumer.start();
    }

}
```

![img](https://img2018.cnblogs.com/blog/885859/201912/885859-20191226180509642-574603645.png)