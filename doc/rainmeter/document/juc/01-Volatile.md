# ============   volatile   =============

volatile是Java虚拟机提供的轻量级的同步机制

## 1 Volatile特性

- 保证可见性
- 禁止指令重排
- 不保证原子性

### 1.1 可见性

**JVM**给每个线程分配一个自己的内存空间（栈）,同时分配一个主内存，多个线程操作主内存的数据时，实际上是对主内存中的数据进行一份数据拷贝，拷贝的数据在各自独立的内存空间进行修改，然后再将修改后的值赋值到主内存中。多线程环境下，操作共享数据时，易造成线程安全问题，而 **Volatile**  关键字正是通过 **可见性** 这一特性保证各个线程获取到的都是最新的值从而避免了线程安全问题。

下面通过代码来证明：

~~~java
public class MyData {
    int a = 0;
    //提供一个修改的方法
    public void update() {
        a = 20;
    }
}
~~~

上面提供了一个类，**update**方法修改变量的值，此时变量并未经过 **Volatile**修饰

开启一个线程来修改变量的值，来检验主线程是否被告知该数据已经被修改

~~~java
public class VolatileDemo {
    public static void main(String[] args) {
        MyData data = new MyData();
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " started");
            try {
                TimeUnit.SECONDS.sleep(3);
                data.update();
                System.out.println("thread a has updated,value = " + data.a);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "AAA").start();

        while (data.a == 0) {
        }
        System.out.println( "main is over,value = " + data.a);
    }
}
~~~

执行上述代码，打印的结果如下

~~~text
AAA started
thread a has updated,value = 20
~~~

很明显 ，主线程并未被告知 共享变量 的值已经被修改，使用 **Volatile** 对该变量进行修饰后，打印结果

~~~java
AAA started
thread a has updated,value = 20
main is over,value = 20
~~~

### 1.2 不保证原子性

**Volatile**修饰的变量并不保证原子性

在上面的 **MyData**中添加下面方法

~~~java
    public void add(){
        a++;
    }
~~~

开启20个线程，每个线程执行1000次 add 操作，如果原子性可以得到保证的话，运行的结果为为 20000

~~~java
public class VolatileDemo {
    public static void main(String[] args) {
        MyData data = new MyData();
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    data.add();
                }
            }).start();
        }
        while (Thread.activeCount() > 2) {
            Thread.yield();
        }
        System.out.println("value is : " + data.a);
    }
}
~~~

多次运行结果都 小于 20000,因为存在丢失问题

那么如何来保证原子性呢，使用原子类 **java.util.concurrent.atomic.AtomicInteger**

~~~java
    public AtomicInteger atomicInteger = new AtomicInteger();
    public void atomicAdd() {
        atomicInteger.getAndIncrement();
    }
~~~

在上面的20个线程中调用该方法

~~~java
data.atomicAdd();
~~~

最后打印出得结果为 20000

### 1.3 禁止指令重排

最为经典的例子，单例设计模式

~~~~java
public class SingleTon {
    private SingleTon() {}
    //禁止指令重排
    private static volatile SingleTon instance = null;
    private static SingleTon getInstance() {
        if (instance == null) {
            synchronized (SingleTon.class) {
                if (instance == null) {
                    instance = new SingleTon();
                }
            }
        }
        return instance;
    }
}
~~~~

