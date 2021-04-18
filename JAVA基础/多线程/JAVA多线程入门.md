
## Java多线程入门

文章主要涉及线程的启动，如何使多线程暂停，如何使多线程停止，线程的优先级级线程安全相关的问题。

### 1.1 进程和多线程的概念及线程的优点

**进程**：进程是操作系统结构的基础，是一次程序的执行，是一个程序及其数据在处理机上顺序执行时所发生的活动，是程序在一个数据集合上运行的过程，它是系统进行资源分配和调度的独立单位。（来源：百度百科）

看了这段话是不是十分抽象，不好理解，但是如果你看到图所示的内容，你还对进程不理解吗？ NO, NO, NO

![](https://www.hualigs.cn/image/607290f94d074.jpg)

难道一个正在操作系统中运行的**exe程序**可以理解成一个进程吗？没错！是他！是他！就是他！

那么在**Windows任务管理器**列表中，完全可以将运行在内存中的exe文件理解成进程，**进程是受操作系统管理的基本运行单元**。

**那什么是线程呢？**

**线程可以理解成是进程中独立运行的子任务**，比如 QQ.exe运行时有好多子任务在同时运行。比如：视频聊天线程，下载文件线程，传输数据线程等。这些不同的任务完全可以同时在运行，其中每一项任务可以理解成不同的线程在工作。

那么这样的优点是什么呢？例如我们使用多任务操作系统Windows后，**可以最大限度地利用CPU的空闲时间来处理其他的任务**，比如一边让操作系统处理正在由打印机打印的数据，一边使用Word编辑文档，而CPU在这些任务之间不停地切换，由于速度非常快，给读者的感受就是这些任务似乎在同时运行。

为了更好理解多线程的优势，首先我们通过单利模型图来理解一下单任务的缺点。

![](https://www.hualigs.cn/image/60729ed6a8451.jpg)

任务1和任务2是两个完全独立，互不相干的任务，任务一是在等待远程服务器返回数据，以便进行后期的处理，这是CPU一直处于等待状态，一直在空运行。而任务2一直处于等待状态必须等任务1返回数据才能运行，这样系统的运行效率大幅降低。单任务的特点就是**排队执行，也就是同步**，就像在cmd中输入一条命令后，必须等待这条命令执行完才可以执行下一条命令一样。所以单线程的缺点是：**CPU利用率大幅降低**。

![](https://www.hualigs.cn/image/60729edf29d09.jpg)

从图中我们可以发现，CPU完全可以在任务1和任务2之间来回切换，使任务2不必等待10秒后再运行，系统的运行效率大大得到提升。

**注意一点！！！！！**

***多线程是异步的，千万不要把IDEA里代码的顺序当成线程执行的顺序，线程被调用时随机的。***

### 1.2 使用多线程

#### 1.2.1 继承Thread类

在Java的JDK开发包中，已经自带了对多线程的支持，实现多线程编程的方式主要有两种：一种是继承**`Thread`**类，一种是实现**`Runable`**接口。

创建多线程之前我们先看看Thread的结构，如下：

```javascript
public class Thread implements Runnable 
```

从上面的源代码中我们可以发现，**`Thread`**类实现了**`Runnable`**接口，他们之间具有多态关系。

其实，使用继承**`Thread`** 类的方式实现多线程时，最大的局限性就是不支持多继承，因为在Java语言的特点就是**单继承**，所以为了实现多继承完全可以采用实现**`Runnable`**接口的方式。总的来说，没有什么本质的区别。

首先我们创建一个自定义的线程呢类 **`MyThread.java`**，继承**`Thread`**类并且重写run()方法，代码如下：

```java
public class MyThread extends Thread{
    @Override
    public void run() {
        System.out.println("MyThread");
    }
}
```

运行类代码如下：

```java
public class Test {
    public static void main(String[] args) {
        MyThread myThread=new MyThread();
        myThread.start();
        System.out.println("运行结束");
    }
}
```

运行结果如下：

>运行结束
>MyThread

从运行的结果来看， **`MyThread.java`**中的run方法执行的时间比较晚，这也说明**在使用多线程技术时，代码的运行结果与代码执行顺序或调用顺序是无关的。**

为什么会出现这样的结果呢？是因为线程是一个子任务，**CPU以不确定的方式或是以随机的时间来调用线程中的run方法**，所以会出现先打印 “ 运行结束 ”，后输出 “ MyThread” 这样的结果。

上面我们提出线程调用的随机性，下面我们创建**`MyThread.java`** 来演示线程的随机性。

创建自定义线程类**`MyThread.java`**，代码如下：

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        try {
            for (int i = 0; i < 5; i++) {
                int time = (int) Math.random() * 1000;
                Thread.sleep(time);
                System.out.println("run=" + Thread.currentThread().getName());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}
```

再创建运行类**`Test.java`**,代码如下：

```java
public class Test {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.setName("myThread");
        myThread.start();
        try {
            for (int i = 0; i < 5; i++) {
                int time = (int) Math.random() * 1000;
                Thread.sleep(time);
                System.out.println("main=" + Thread.currentThread().getName());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}
```

代码运行结果：

>main=main
>run=myThread
>main=main
>run=myThread
>main=main
>run=myThread
>main=main
>run=myThread
>main=main
>run=myThread

在代码中，为了展示线程具有随机性，所以使用随机数的形式来使线程得到挂起的效果，从而表现出CPU执行那个线程具有不确定性。

说明：**`MyThread.java`**中的start()方法通知“**线程规划器**”，此线程已经准别就绪，等待调用线程对象的run方法，这个过程其实就是让CPU安排一个时间来调用**`MyThread.java`**类中的run方法，也就是使线程得到运行。

在强调一点，执行start()方法的顺序不代表线程启动的顺序，创建测试类说明一下，代码如下：

```java
public class MyThread extends Thread {
    private int i;

    public MyThread(int i) {
        this.i = i;
    }

    @Override
    public void run() {
        System.out.println("i=" + i);

    }
}
```

运行类**`Test.java`**,代码如下：

```java
public class Test {
    public static void main(String[] args) {
        MyThread myThread1 = new MyThread(1);
        MyThread myThread2 = new MyThread(2);
        MyThread myThread3 = new MyThread(3);
        MyThread myThread4 = new MyThread(4);
        MyThread myThread5 = new MyThread(5);
        myThread1.start();
        myThread2.start();
        myThread3.start();
        myThread4.start();
        myThread5.start();
    }
```

程序运行后的结果如图：

>i=1
>i=2
>i=5
>i=3
>i=4

#### 1.2.2 实现Runnable接口

如果我们创建的线程类已经有一个父类了，这时候就不能继承**`Thread`**类了，因为在Java中不支持多继承，所以我们需要实现**`Runnable`**接口来实现多线程。

创建一个实现**`Runnable`**接口的类

```java
public class MyRunnable implements Runnable{

    @Override
    public void run() {
        System.out.println("运行中！！！！！");
    }
}
```

如何使用**`MyRunnable.java`**类呢，我们看一下**`Thread.java`**类的构造函数，如下图所示：

![](https://www.hualigs.cn/image/6072daa6d8c36.jpg)

在**`Thread.java`**类中的8个构造函数中，有两个构造函数**Thread**(Runnable target)和**Thread**(Runnable target, [String](chm://a7ad5e114792014d375eb3258ab76253/java/lang/String.html) name)可以传递**`Runnable`**接口，说明构造函数支持传入一个**`Runnable`**接口的对象，运行类代码如下：

```java
public class Test {
    public static void main(String[] args) {
       MyRunnable myRunnable=new MyRunnable();
       Thread thread=new Thread(myRunnable);
       thread.start();
        System.out.println("运行结束！！！");
    }
}
```

运行结果如图：

>运行结束！！！
>运行中！！！！！

另外需要说明一点，`Thread.java`类也是实现**`Runnable`**接口，如下：

```java
public class Thread implements Runnable 
```

那也就意味着构造函数**Thread**(Runnable target)不光可以传入**`Runnable`**接口对象，还可以传入一个**`Thread.java`**类的对象，这样做完全可以将一个**`Thread.java`**对象的run()方法交给其他的线程进行调用。

#### 1.2.3 实例变量与线程安全

自定义线程类中的实例变量针对其他线程可以有**共享和不共享**之分，下面我们分开来说明这两点：

**不共享数据的情况**：

不共享数据的情况如下图展示说明：

![](https://www.hualigs.cn/image/6072e0fd4ef2d.jpg)

下面我们通过一个示例来看下数据不共享情况，创建一个**`MyThread.java`**类代码如下：

```java
public class MyThread extends Thread {
    private int count = 5;

    public MyThread(String name) {
        this.setName(name);
    }

    @Override
    public void run() {
        while (count > 0) {
            count--;
            System.out.println("由 " + this.currentThread().getName() + " " + "计算, count " + count);
        }
    }
}
```

运行类**`Test.java`**代码如下：

```java
public class Test {
    public static void main(String[] args) {
        MyThread myThread=new MyThread("A");
        MyThread myThread1=new MyThread("B");
        MyThread myThread2=new MyThread("C");
        myThread.start();
        myThread1.start();
        myThread2.start();
    }
}
```

不共享数据运行结果如下：

>由 B 计算, count 4
>由 C 计算, count 4
>由 A 计算, count 4
>由 C 计算, count 3
>由 B 计算, count 3
>由 C 计算, count 2
>由 A 计算, count 3
>由 C 计算, count 1
>由 B 计算, count 2
>由 C 计算, count 0
>由 A 计算, count 2
>由 B 计算, count 1
>由 A 计算, count 1
>由 B 计算, count 0
>由 A 计算, count 0

我们总共创建了3个线程，每个线程都有各自的count变量，自己减少自己的count变量的值，这样的情况就是变量不共享。

那么，如果想实现3个线程共同对一个count变量进行减法操作的目的，该如何设计呢？

**共享数据的情况**

共享数据的情况如下图：

![](https://www.hualigs.cn/image/6072e6d88d343.jpg)

共享数据的情况就是多个线程同时访问一个变量，比如实现投票功能的软件时，多个线程可以同时处理同一个人的票数。

下面我们通过代码实例演示一下共享数据的情况，创建一个**`MyThread.java`**类代码如下：

```java
public class MyThread extends Thread {
    private int count = 3;

    @Override
    public void run() {
        count--;
        System.out.println("由 " + this.currentThread().getName() + " " + "计算, count " + count);

    }
}
```

运行类**`Test.java`**代码如下：

```java
public class Test {
    public static void main(String[] args) {
        MyThread myThread=new MyThread();
        Thread thread=new Thread(myThread,"A");
        Thread thread1=new Thread(myThread,"B");
        Thread thread2=new Thread(myThread,"C");
        thread.start();
        thread1.start();
        thread2.start();
    }
}
```

运行结果：

>由 A 计算, count 0
>由 C 计算, count 0
>由 B 计算, count 0

从运行结果我们可以看出，线程A，线程B，线程C 的count值都是2，说明A,B,C同时对count进行处理，产生了“非线程安全”的问题。

那我们修改代码如下 即在run() 方法前面加上**`synchronized`**关键字：

```java
public synchronized void run() {
    count--;
    System.out.println("由 " + this.currentThread().getName() + " " + "计算, count " + count);
}
```

重新运行程序就不回产生值一样的情况了，结果显示如下：

>由 A 计算, count 2
>由 C 计算, count 1
>由 B 计算, count 0

​      通过在run()方法前面加上`synchronized`关键字，使多个线程在执行run()方法时，以排队的形式进行处理。但一个线程调用run前，先判断run方法有没有被上锁，如果上锁，说明有其他线程正在调用run方法，必须等其他线程对run方法调用结束后才可以执行run方法。这样排队调用run方法的目的，也就达到按顺序对count变量减1的效果了。同时**`synchronized`**可以在任何方法上加锁，而加锁的这段代码叫做“**互斥区**” 或“**临界区**”。

另外说明，当一个线程想要执行同步方法里面的代码时，线程首先去拿这把锁，如果能够拿到这把锁，那么这个线程就可以执行**`synchronized`**如果拿不到这把锁，那么线程就不断的尝试拿这把锁，直到能够拿到为止。

这里我们引出一个概念，“**非线程安全**”。非线程安全主要是指***多个线程对同一个对象中的同一个实例变量进行操作时会出现值被更改，值不同步的情况，进而影响程序的执行流程。***

下面我们来实现一个非线程安全的实例，**`loginServlet.java`**代码如下：

```java
public class loginServlet {
    private static String usernameRef;
    private static String passwordRef;

    public static void doPost(String username, String password) {
        try {
            usernameRef = username;
            if (username.equals("username=" + username)) {
                Thread.sleep(5000);
            }
            passwordRef = password;
            System.out.println("username=" + usernameRef + " " + " password=" + passwordRef);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}
```

线程**`ALogin.java`**代码如下：

```java
public class ALogin extends Thread{
    @Override
    public void run() {
       loginServlet.doPost("a","aa");
    }
}
```

线程**`BLogin.java`**代码如下：

```java
public class BLogin extends Thread{
    @Override
    public void run() {
       loginServlet.doPost("b","bb");
    }
}
```

代码实例运行结果：

>username=a  password=bb
>username=b  password=bb

由运行结果我们可以看出，出现了线程不安全的问题，解决这个问题的方法我们使用**`synchronized`**关键字修饰别调用的方法即可。

`public static synchronized  void doPost(String username, String password) `

### 1.3 多线程常用方法

#### 1.3.1 CurrentThread() 方法

CurrentThread() 方法返回**代码段正在被哪个线程调用的信息**，下面通过一个示例来说明：

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName());
    }
}
```

运行结果：

>main

结果说明，main 方法被名为main的线程调用。

继续实验，创建 **`MyThread.java`**代码如下：

```java
public class MyThread extends Thread {
    public MyThread() {
        System.out.println("构造方法的打印："+Thread.currentThread().getName());
        this.setName("MyThread");
    }

    @Override
    public void run() {
        System.out.println("run方法的打印："+Thread.currentThread().getName());
    }
}
```

运行类**`Test.java`**代码如下：

```java
public class Test {
    public static void main(String[] args) {
        MyThread myThread=new MyThread();
        myThread.start();
    }
}
```

运行结果：

>构造方法的打印：main
>run方法的打印：MyThread

从运行结果我们可以发现，**`MyThread.java`**类的构造函数是被main线程调用的，而run方法是被名称为MyThread的线程调用的。

#### 1.3.2 isAlive()方法

方法isAlive()的功能是**判断当前的线程是否处于活动状态**。

创建 **`MyThread.java`**代码如下：

```java
public class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("run="+this.isAlive());
    }
}
```

运行类**`Test.java`**代码如下：

```java
public class Test {
    public static void main(String[] args) {
        MyThread myThread=new MyThread();
        System.out.println("begin==" + myThread.isAlive());
        myThread.start();
        // Thread.sleep(1000);
        System.out.println("end==" + myThread.isAlive());
    }
}
```

运行结果：

>begin==false
>end==true
>run=true

方法isAlive()的作用是测试线程是否处于活动状态，那什么是活动状态呢？***活动状态就是线程已经启动且尚未终止。即线程处于正在运行或准备开始运行的状态，就认为线程是存活的。***

说明一下，如下代码：

`System.out.println("end==" + myThread.isAlive());`

虽然上面打印的值是true,但是此值是不确定的，打印true是因为MyThread线程还未执行完毕，所以输出true,如果修改代码把**`Test.java`**中 Thread.sleep(1000)代码放开，运行结果输出是false,因为MyThread线程在1秒内就执行完毕。

另外，在使用isAlive()方法时，如果将线程对象以构造函数的方式传递给Thread对象进行start()启动时，运行的结果和前面的实例是有差异的，造成这样差异的原因是来自于Thread.currentThread()和this的差异。

#### 1.3.3 sleep() 方法

方法sleep()的作用是在指定的毫秒数内让当前“ **正在执行的线程**”休眠（）暂停执行，这个正在执行的线程是指this.currentThread()返回的线程，前面的实例也提到过，这里代码不做说明。

#### 1.3.4 getId() 方法

getId() 方法的作用是取得线程的唯一的唯一标示。

运行类**`Test.java`**代码如下：

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+ ":"+ Thread.currentThread().getId());
    }
}
```

运行结果：

>main: 1

#### 1.3.5 yield()方法

yield()方法的作用是放弃当前的CPU资源，将它让给其他的任务去占用CPU时间。但是放弃的时间是不确定的，有可能刚刚放弃，马上又获得CPU时间片。

创建 **`MyThread.java`**代码如下：

```java
public class MyThread extends Thread {

    @Override
    public void run() {
        long beginTime = System.currentTimeMillis();
        int count = 0;
        for (int i = 0; i < 1000000; i++) {
            //Thread.yield();
            count+=count;
        }
        long endTime = System.currentTimeMillis();
        System.out.println("用时："+(endTime-beginTime)+ "毫秒！");
    }
}
```

运行类**`Test.java`**代码如下：

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread=new MyThread();
        myThread.start();
    }
}
```

程序运行后结果：

>用时：3毫秒！

去掉注释代码，再次运行，结果如下：**说明将CPU让给其他资源导致速度变慢。**

>用时：323毫秒！

### 1.4 停止线程

***停止一个线程意味着在线程处理完任务之前停掉正在做的操作*，也就是放弃当前的操作。**

在Java中有一下三种方式可以终止正在运行的线程：

- 使用退出标示，使线程正常退出，也就是当run方法完成后线程终止。
- 使用stop方法强行终止线程，因为stop和suspend及resume一样，都是作废的方法，使用起来起来kennel产生不可预料的结果。
- 使用interrupt方法中断线程（常用）

***大多数停止一个线程的操作使用Thread.interrupt()方法，尽管方法的名称是“ 停止，中止”的意思，但是这个方法不会停止一个正在运行的线程，还需要加入一个判断才可以完成线程的停止操作。***

#### 1.4.1 停止不了的线程

使用**interrupt()**方法停止线程，但是**interrupt()**放大不像for+break语句那样，可以马上就停止循环，调用**interrupt()**方法**仅仅是在当前线程中打了一个停止的标记，并不是真正的停止线程。**

创建 **`MyThread.java`**代码如下：

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i <= 10000; i++) {
            System.out.println("i="+i);
        }
    }
}
```

运行类**`Test.java`**代码如下：

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread=new MyThread();
        myThread.start();
        myThread.interrupt();
    }
}
```

部分运行结果：

>i=9997
>i=9998
>i=9999
>i=10000

从运行的结果来看，调用**interrupt**方法并没有停止线程，那么如何停止线程呢？

#### 1.4.2 判断线程是否是停止状态

判断线程的状态是不是停止的，在Java的JDK中，**`Thread.java`**类中提供了两种方法：

this.isInterrupted()：测试当前线程是否已经中断。
this.isInterrupted(); 测试线程是否已经中断。

```java
public static boolean interrupted() 
```

```java
public boolean isInterrupted()
```

一个是静态的方法一个不是静态的方法。

下面我们说明如何是main线程产生中断的效果呢？ 创建**`Test.java`**类，代码如下：

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {       
        Thread.currentThread().interrupt();
        System.out.println("是否停止1："+Thread.interrupted());
        System.out.println("是否停止2："+Thread.interrupted());
    }
}
```

程序运行后结果：

>是否停止1：true
>是否停止2：false

从运行的结果来看，**interrupted**方法判断当前线程是否是停止状态。但是为什么第二个布尔值为false呢，这是由于 interrupted方法有清除状态的功能，所以第二次调用的返回值是false。而**isInterrupted**判断线程是否中断不清除状态标示。

接下来我们通过一个示例来说明如何停止一个线程，创建 **`MyThread.java`**代码如下：

```java
public class MyThread extends Thread {

    @Override
    public void run() {
        int count = 0;
        while (!Thread.currentThread().isInterrupted()) {
            count++;
        }
        System.out.println("循环次数：" + count + "，线程中断，中断信号：" + Thread.currentThread().isInterrupted());

    }
}
```

创建**`Test.java`**类，代码如下：

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread=new MyThread();
        myThread.start();
        Thread.sleep(5);
        myThread.interrupt();

    }
}
```

程序运行结果：

>循环次数：88528，线程中断，中断信号：true

**以上结果可以看到循环 88528 次后，线程收到了中断信号（即 `Thread.currentThread().isInterrupted()` 返回的结果为 true），循环条件不满足条件，退出循环，执行完程序，自动停止线程，这种就属于通过 interrupt 正确停止线程的情况。**

### 1.5 暂停线程

#### 1.5.1 suspend与resume方法的使用

暂停线程意味着此线程还可以恢复运行，在Java多线程中，可以使用**suspend**方法暂停线程，使用**resume**方法恢复线程的执行。

创建 **`MyThread.java`**代码如下：

```java
public class MyThread extends Thread {
    private  long i=0;

    public long getI() {
        return i;
    }

    public void setI(long i) {
        this.i = i;
    }

    @Override
    public void run() {
      while (true){
          i++;
      }
    }
}
```

创建**`Test.java`**类，代码如下：

```java
public class Test {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread=new MyThread();
        myThread.start();
        Thread.sleep(1000);
        //A段
        myThread.suspend();
        System.out.println("A= "+System.currentTimeMillis()+ " i="+myThread.getI());
        Thread.sleep(1000);
        System.out.println("A= "+System.currentTimeMillis()+ " i="+myThread.getI());

        myThread.resume();
        Thread.sleep(1000);

        myThread.suspend();
        System.out.println("B= "+System.currentTimeMillis()+ " i="+myThread.getI());
        Thread.sleep(1000);
        System.out.println("B= "+System.currentTimeMillis()+ " i="+myThread.getI());
    }
}
```

程序运行结果：

>A= 1618220660051 i=690427887
>A= 1618220661056 i=690427887
>B= 1618220662061 i=1410759540
>B= 1618220663066 i=1410759540

从程序运行结果来看，线程的确是被暂停了，而且还可以恢复成运行的状态。

### 1.6 线程的优先级

在操作系统中，线程可以划分优先级，优先级较高的线程得到CPU资源较多，也就是CPU优先执行优先级较高的线程对象中的任务。

设置线程优先级有助于“线程规划器”，确定下一次选择哪个线程来优先执行。

设置线程优先级使用setPriority() 方法，此方法在JDK的源代码如下：

```java
public final void setPriority(int newPriority) {
    ThreadGroup g;
    checkAccess();
    if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
        throw new IllegalArgumentException();
    }
    if((g = getThreadGroup()) != null) {
        if (newPriority > g.getMaxPriority()) {
            newPriority = g.getMaxPriority();
        }
        setPriority0(priority = newPriority);
    }
}
```

在Java中，线程优先级分为 1～～10这10个等级，如果小于1或者大于10，则JDK抛出异常为 **throw new IllegalArgumentException()**

```java
public final static int MIN_PRIORITY = 1;
public final static int NORM_PRIORITY = 5;
public final static int MAX_PRIORITY = 10;
```

#### 1.6.1 优先级的继承特性

在Java中，线程的优先级具有继承性，比如A线程启动B线程，则B线程的优先级和A线程是一样的。

创建 **`MyThread1.java`**代码如下：

```java
public class MyThread1 extends Thread{
    @Override
    public void run() {
        System.out.println("MyThread1 run Priority="+this.getPriority());
        MyThread2 myThread2=new MyThread2();
        myThread2.start();
    }
}
```

创建 **`MyThread2.java`**代码如下：

```java
public class MyThread2 extends Thread{
    @Override
    public void run() {
        System.out.println("MyThread2 run Priority="+this.getPriority());
    }
}
```

创建**`Test.java`**类，代码如下：

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(" main thread begin  priority="+Thread.currentThread().getPriority());
        //Thread.currentThread().setPriority(8);
        System.out.println(" main thread end priority="+Thread.currentThread().getPriority());
        MyThread1 myThread1=new MyThread1();
        myThread1.start();
    }
}
```

运行结果：

>main thread begin  priority=5
>main thread end priority=5
>MyThread1 run Priority=5
>MyThread2 run Priority=5

将代码  **Thread.currentThread().setPriority(8)** 的注释去掉，再次运行**`Test.java`**文件，显示如下：

> main thread begin  priority=5
> main thread end priority=8
> MyThread1 run Priority=8
> MyThread2 run Priority=8

#### 1.6.2 优先级具有规则性

虽然使用setPriority()方法可以设置线程的优先级，但还没有看到设置优先级带来的效果。

创建 **`MyThread1.java`**代码如下：

```java
public class MyThread1 extends Thread{
    @Override
    public void run() {
        long beginTimeMillis = System.currentTimeMillis();
        long addResult=0;
        for (int i = 0; i < 10; i++) {
            for (int j = 0; j < 50000; j++) {
                Random random=new Random();
                random.nextInt();
                addResult=addResult+j;
            }
        }
        long endTimeMillis = System.currentTimeMillis();
        System.out.println( " @@@@@@@@@@@@@@@@@@  thread 1 use time =  "+ (endTimeMillis-beginTimeMillis) );
    }
}
```

创建 **`MyThread2.java`**代码如下：

```java
public class MyThread2 extends Thread{
    @Override
    public void run() {
        long beginTimeMillis = System.currentTimeMillis();
        long addResult=0;
        for (int i = 0; i < 10; i++) {
            for (int j = 0; j < 50000; j++) {
                Random random=new Random();
                random.nextInt();
                addResult=addResult+j;
            }
        }
        long endTimeMillis = System.currentTimeMillis();
        System.out.println( " @@@@@@@@@  thread 2 use time =  "+ (endTimeMillis-beginTimeMillis) );
    }
}
```

创建**`Test.java`**类，代码如下：

```java
public class Test {
    public static void main(String[] args) {
        for (int i = 0; i <5 ; i++) {
            MyThread1 myThread1=new MyThread1();
            myThread1.setPriority(10);
            myThread1.start();
            MyThread2 myThread2=new MyThread2();
            myThread2.setPriority(1);
            myThread2.start();
        }

    }
}
```

程序运行的结果：

> @@@@@@@@@@@@@@@@@@  thread 1 use time =  412
> @@@@@@@@@  thread 2 use time =  426
> @@@@@@@@@@@@@@@@@@  thread 1 use time =  438
> @@@@@@@@@  thread 2 use time =  449
> @@@@@@@@@@@@@@@@@@  thread 1 use time =  449
> @@@@@@@@@@@@@@@@@@  thread 1 use time =  455
> @@@@@@@@@  thread 2 use time =  459
> @@@@@@@@@  thread 2 use time =  461
> @@@@@@@@@@@@@@@@@@  thread 1 use time =  462
> @@@@@@@@@  thread 2 use time =  463

从运行的结果我们发现，高优先级的线程总是大部分先执行完，但不代表优先级高的全部执行完。另外，不要以为**MyThread1**线程先被main线程调用就先执行完，出现这样的结果是因为**MyThread1**线程的优先级高。当线程优先级差距较大时，谁先执行完和代码的调用顺序无关。**同时说明线程的优先级具有一定的规则性，也就是CPU尽量将执行资源让给优先级比较高的线程。**

#### 1.6.3 优先级具有随机性

因为优先级具有随机性，也就是优先级比较高的线程不一定每一次都先执行完。

我们将上面**`Test.java`**类代码myThread2.setPriority(5)，运行代码结果如下：

> @@@@@@@@@@@@@@@@@@  thread 1 use time =  440
> @@@@@@@@@@@@@@@@@@  thread 1 use time =  464
> @@@@@@@@@@@@@@@@@@  thread 1 use time =  465
> @@@@@@@@@  thread 2 use time =  466
> @@@@@@@@@  thread 2 use time =  472
> @@@@@@@@@  thread 2 use time =  473
> @@@@@@@@@  thread 2 use time =  475
> @@@@@@@@@@@@@@@@@@  thread 1 use time =  476
> @@@@@@@@@  thread 2 use time =  476
> @@@@@@@@@@@@@@@@@@  thread 1 use time =  476

那么，我们得出一个结论，**不要把线程的优先级与运行结果的顺序作为衡量的标准，优先级较高的不一定每一次都先执行完。也就是说优先级与打印的顺序无关，因为它们的关系具有不确定性和随机性。**

### 1.7 守护线程

在Java线程中有两种线程，一种是用户线程，另一种是守护线程。

守护线程是一种特殊的线程，它的特性有“陪伴”的含义，当进程中不存在非守护线程了。则守护线程自动销毁。典型的守护线程就是**垃圾回收线程**。

通俗的说：“守护线程”：任何守护线程都是整个JVM中非守护线程的“保姆”，只有当前JVM实例中存在任何一个非守护线程没有结束，守护线程就在工作，只有当最后一个非守护线程结束时，守护线程才随JVM一同结束工作。Daemon的作用是为其他线程的运行提供便利服务，守护线程最典型的应用就是GC(垃圾回收器)。

创建 **`MyThread.java`**代码如下：

```java
public class MyThread extends Thread {
    private  int i=0;
    @Override
    public void run() {
        try {
            while (true){
                i++;
                System.out.println("i= "+i);
                Thread.sleep(1000);
            }
        }catch(InterruptedException e){
            e.printStackTrace();
        }

    }
}
```

创建**`Test.java`**类，代码如下：

```java
public class Test {
    public static void main(String[] args) throws  InterruptedException{
       MyThread myThread=new MyThread();
       myThread.setDaemon(true);
       myThread.start();
       Thread.sleep(5000);
        System.out.println("我离开 myThread 对象也不打印了，也停止了");
    }
}
```

程序运行的结果：

>i= 1
>i= 2
>i= 3
>i= 4
>i= 5
>我离开 myThread 对象也不打印了，也停止了

