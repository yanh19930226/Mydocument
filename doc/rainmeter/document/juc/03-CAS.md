# ================   CAS   ================

## 1 概念

比较并交换

jdk5增加了并发包java.util.concurrent.*,其下面的类使用CAS算法实现了区别于 **synchronized** 同步锁的一种乐观锁。JDK 5之前Java语言是靠synchronized关键字保证同步的，这是一种独占锁，也是是悲观锁

## 2 算法

CAS是一种无锁算法，CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做 

## 3 缺点

- 循环时间长，开销大
- 只能保证一个共享变量的原子操作
- ABA问题

### 3.1 ABA问题

CAS算法实现的一个特点就是要取出主内存中某个时刻的数据进行比对并进行替换，那么在这个时间差内就会导致数据的变化， 举个例子:

 线程一把数据A变为了B，然后又重新变成了A。此时另外一个线程读取的时候，发现A没有变化，就误以为是原来的那个A。这就是有名的ABA问题 . 尽管线程1的CAS操作成功，但是不代表这个过程就是没问题的。 再如：

一个小偷，把别人家的钱偷了之后又还了回来，还是原来的钱吗，你老婆出轨之后又回来，还是原来的老婆吗？ABA问题也一样，如果不好好解决就会带来大量的问题。最常见的就是资金问题，也就是别人如果挪用了你的钱，在你发现之前又还了回来。但是别人却已经触犯了法律 

代码演示ABA问题

~~~java
static AtomicReference<Integer> atomicReference = new AtomicReference(100);
new Thread(() -> {
    atomicReference.compareAndSet(atomicReference.get(), 200);
    atomicReference.compareAndSet(atomicReference.get(), 100);
}, "线程一").start();

new Thread(() -> {
    try {
        TimeUnit.SECONDS.sleep(2);
        atomicReference.compareAndSet(atomicReference.get(), 2020);
        System.out.println( " 线程二 get: " + atomicReference.get());
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}).start();
~~~



**如何去解决这个ABA问题呢 ？**

通过时间戳原子引用来解决

### 3.2 时间戳原子引用

通过添加版本号（类似时间戳）来解决ABA问题

**java.util.concurrent.atomic.AtomicStampedReference<V>**