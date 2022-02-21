# Java-Study

#### 介绍
**作者自己的手写笔记，这只是一小部分(大概只占了30%)，全都是一人完成，如有错误请见谅。该文档将会持续完善。**

## Java并发编程（多线程高并发）

### 创建线程的三种方式

#### 继承于Thread类

```java
public class createThreadTest1 {

    public static void main(String[] args) {

        thread01 thread01 = new thread01();
        thread01.start(); //调用start方法开启线程
        
    }


}
class thread01 extends Thread{


    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+"===>正在运行");

        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}
```

 



#### 实现Runnable接口（推荐）

**因为Java不支持多继承，所以用实现接口的方法可扩展性会更高，让唯一的继承留个更加有用的类**

```java

public class createThreadTest1 {

    public static void main(String[] args) {
 
        Thread thread02 = new Thread(new thread02()); 
        thread02.start();


    }


}

class thread02 implements Runnable{


    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+"===>正在运行");

        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}
```





**也可以这样：**

```java
  //方法3：匿名开启线程
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("匿名开启线程");
                
            }
        }).start();
```



**可以用lambda表达式**

```java
   new Thread(() ->{
            System.out.println("666");


      }).start();
```



#### 实现Callable接口

**callable实现比较复杂**

```java

public class createThreadTest1 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        thread03 thread03 = new thread03();
        FutureTask<String> futureTask=new FutureTask<>(thread03);
        new Thread(futureTask).start();
        String getMsg = futureTask.get(); //这段代码必须在开启线程之后写
        System.out.println(getMsg);
    }

class thread03 implements Callable<String>{


    @Override
    public String call() throws Exception {
        return "callable";

    }
}
```

### Thread常用方法



```java
public class threadMethod {

    public static void main(String[] args) {
        thread01 thread01=new thread01();
        Thread thread = new Thread(thread01);
        thread.setName("t1");//1.设置线程名字
        thread.setPriority(5);//2.设置线程优先级（一般不建议设置），优先级高不一定先执行。。。。
        System.out.println("thread.getState()==>"+thread.getState());//3.获取当前线程状态
        boolean alive1 = thread.isAlive();//4.查看当前线程是否活着
        System.out.println("alive==>"+alive1);

        thread.start(); //开启线程
        try {
            Thread.sleep(500);//5.线程休眠500ms
            boolean alive = thread.isAlive();//查看当前线程是否活着
            System.out.println("alive==>"+alive);

        } catch (InterruptedException e) {
            e.printStackTrace();
        }


    }




}
class thread01 implements Runnable{

    @Override
    public void run() {
        System.out.println("id==>"+Thread.currentThread().getId()); //获取当前线程id
        System.out.println("name==>"+Thread.currentThread().getName());//获取当前线程名
        System.out.println("Priority===>"+Thread.currentThread().getPriority());//获取线程优先级
        System.out.println("state==>"+Thread.currentThread().getState());//获取当前线程状态

        System.out.println("alive==>"+Thread.currentThread().isAlive());
    }
}
```

#### join方法

**注意：join方法只能在开启线程之后在使用，不然就会无效，也就是先start（）再join（）**



**join方法的用处，可以让子线程去处理或者计算一些值，计算完之后main线程可以使用计算完的值**

```java
public class threadJoinTest {

    private static int count=0;
    public static void main(String[] args) {

        System.out.println("main=========");

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("进入run方法");
                for (int i = 0; i < 5; i++) {
                     count++;
                    System.out.println("子线程==="+count);
                }

            }



        });

        t1.start();
        try {
            //注意：join方法必须要在start方法后面，不然不生效，因为调用了start方法才会创建线程。然后再线程插队
            t1.join(); //线程插队。也就是只有t1线程执行完，join方法后面的代码才能执行
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("main线程==="+count);


    }
}

```











### 计数器

```java
public class threadTimer {

    public static void main(String[] args) {
        timer timer = new timer();
        timer.setNum(10);
        new Thread(timer).start();

    }


}
class timer implements Runnable{

    private int num;
    private boolean isRun=true;

    public void setNum(int num) {
        this.num = num;
    }


    @Override
    public  void run() {

            while (isRun){
                if(num<=0){
                    isRun=false;
                    System.out.println("程序结束！");
                    return;
                }
                System.out.println("倒计时，还剩"+num+"秒");
                try {
                    Thread.sleep(1000);
                    num--;
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
    }
}

```



### 模拟并发（多线程）抢票=>超卖问题

#### 单线程抢票，没有安全问题

```java
public class threadTicket {
    /**
     * 模拟高并发抢票线程不安全问题
     */

    public static void main(String[] args) {
        ticket ticket = new ticket();
        new Thread(ticket).start();


    }


}
class ticket implements Runnable{

    private int ticket=100; //抢100张票

    @Override
    public void run() {
        while (true){
            if(ticket<=0){
                System.out.println("票被抢完了");
                return;
            }
            ticket--;
            System.out.println(Thread.currentThread().getName()+"==>抢到票了，还剩"+ticket+"张票");
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}
```



#### 多线程抢票出现安全问题

```java
public class threadTicket {
    /**
     * 模拟高并发抢票线程不安全问题
     */

    public static void main(String[] args) {
        ticket ticket = new ticket();
        for (int i = 0; i < 10; i++) { //开启10个线程抢票
            new Thread(ticket).start();
        }


    }


}
class ticket implements Runnable{

    private int ticket=100; //抢100张票

    @Override
    public void run() {
        while (true){
            if(ticket<=0){
                System.out.println("票被抢完了");
                return;
            }
            ticket--;
            System.out.println(Thread.currentThread().getName()+"==>抢到票了，还剩"+ticket+"张票");
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}

```



![image-20210314145848698](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210314145848698.png)


**出现很多人抢到同一张票，出现了线程不安全问题**



#### 解决多线程抢票线程不安全问题

**其实多线程抢票为啥会出现线程不安全问题，原因就是'  i--  ' 语句。i--不是原子操作，肯定会出线程安全问题**

**i--  ,其实有几个过程   == 1.先读取i的值 2.让i-1  3.赋值给i  4.保存到主内存**







**方法一：加锁，使用synchronized关键字**

**缺点：synchronized关键字和Lock显式锁是JVM级别的，对于同一个JVM进程中有效，但是对于分布式环境的多进程是无效的，因为分布式环境是多进程的，也就是不属于同一个JVM进程，这时候只能采用分布式锁了，比如Redis分布式锁**

**同步语句块。**

```java
public class threadTicket {
    /**
     * 模拟高并发抢票线程不安全问题
     */

    public static void main(String[] args) {
        ticket ticket = new ticket();
        for (int i = 0; i < 15; i++) {
            new Thread(ticket).start();
        }


    }


}
class ticket implements Runnable{

    private int ticket=10; //抢10张票

    @Override
    public void run() {

        synchronized (this){

                if(ticket<=0){
                    System.out.println("票被抢完了");
                    return;
                }
                ticket--;
                System.out.println(Thread.currentThread().getName()+"==>抢到票了，还剩"+ticket+"张票");
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

        }



    }
}

```

**和上面一样**

```java
public class threadTicket {
    /**
     * 模拟高并发抢票线程不安全问题
     */

    public static void main(String[] args) {
        ticket ticket = new ticket();
        for (int i = 0; i < 15; i++) {
            new Thread(ticket).start();
        }


    }


}

class ticket implements Runnable {

    private int ticket = 30; //抢10张票
    private static final Object obj = new Object(); //加上static，不管是什么对象，只要是这个类的，就只有一个obj

    @Override
    public void run() {

        while (true) {

            synchronized (obj) {

                if (ticket <= 0) {
                    System.out.println("票被抢完了");
                    return;
                }
                ticket--;
                System.out.println(Thread.currentThread().getName() + "==>抢到票了，还剩" + ticket + "张票");
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }


    }
}

```



**方式二：使用并发包下的原子类Atomicxxxx**

**Atomic原子类，保证了线程中变量的原子性，又因为源码里面有volatile关键字，使得这个value对于线程是可见的，保证了变量的可见性。**



```java
public class threadTicket {
    /**
     * 模拟高并发抢票线程不安全问题
     */

    public static void main(String[] args) {
        ticket ticket = new ticket();
        for (int i = 0; i < 10; i++) { //开启10个线程抢票
            new Thread(ticket).start();
        }


    }


}
class ticket implements Runnable{

//    private  int ticket=100; //抢100张票
    private AtomicInteger atomicInteger=new AtomicInteger(5);

    @Override
    public void run() {
        while (true){

            if(atomicInteger.get()<=0){
                System.out.println("票被抢完了");
                return;
            }
//            ticket--;
            int andDecrement = atomicInteger.getAndDecrement();
            System.out.println(Thread.currentThread().getName()+"==>抢到票了，还剩"+andDecrement+"张票");
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

```

### 多线程的原子性、可见性、有序性



#### 原子性

**保证原子性有两个方法：1.使用锁  2.CAS指令**

**众所周知，i++ i--  不是原子操作，那么也就不具有原子性，在多线程环境下，面对多个线程并发访问是线程不安全的，容易出问题的**

**方法一：synchronized关键字或者lock显式锁**

```java
public class threadTicket {
    /**
     * 模拟高并发抢票线程不安全问题
     */

    public static void main(String[] args) {
        ticket ticket = new ticket();
        for (int i = 0; i < 15; i++) {
            new Thread(ticket).start();
        }


    }


}

class ticket implements Runnable {

    private int ticket = 30; //抢10张票
    private static final Object obj = new Object(); //加上static，不管是什么对象，只要是这个类的，就只有一个obj

    @Override
    public void run() {

        while (true) {

            synchronized (obj) {

                if (ticket <= 0) {
                    System.out.println("票被抢完了");
                    return;
                }
                ticket--;
                System.out.println(Thread.currentThread().getName() + "==>抢到票了，还剩" + ticket + "张票");
                try {
                    Thread.sleep(200);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }


    }
}
```



**方法二：用CAS指令，不过java并发包提供了实现CAS指令的工具，也就是AtomicXXX，底层使用了volatile，保证了变量对于线程的可见性**

```java
public class threadTicket {
    /**
     * 模拟高并发抢票线程不安全问题
     */

    public static void main(String[] args) {
        ticket ticket = new ticket();
        for (int i = 0; i < 10; i++) { //开启10个线程抢票
            new Thread(ticket).start();
        }


    }


}
class ticket implements Runnable{

//    private  int ticket=100; //抢100张票
    private AtomicInteger atomicInteger=new AtomicInteger(5);

    @Override
    public void run() {
        while (true){

            if(atomicInteger.get()<=0){
                System.out.println("票被抢完了");
                return;
            }
//            ticket--;
            int andDecrement = atomicInteger.getAndDecrement();
            System.out.println(Thread.currentThread().getName()+"==>抢到票了，还剩"+andDecrement+"张票");
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
```



#### 可见性（演示不出来）



#### 有序性

### 多线程锁问题

**锁对象不同不能同步**



```java
public class syncLockTest {

    //创建两个线程对象，synchronized(this)就无法发挥作用了，因为subthread1和subthread2是两个不同的对象
    private static subThread1 subThread1=new subThread1();
    private static subThread1 subThread2=new subThread1();
    public static void main(String[] args) {

        Thread t1 = new Thread(subThread1); //传入两个不同的对象
        Thread t2 = new Thread(subThread2);

        t1.setName("111");
        t2.setName("222");
        t1.start();
        t2.start();

    }
}
class subThread1 implements Runnable{

    private  int count=10;

    @Override
    public void run() {

        sm1();

    }

    public synchronized void sm1(){
        while (true){

                if(count>0){
                    count--;
                    System.out.println(Thread.currentThread().getName()+"===>还剩"+count);
                }else {
                    break;
                }
        }
    }
}
```

**这段代码虽然加了锁，但是也是有线程安全问题的**

**因为subthread1和subthread2是不同的对象，所以synchronized(this)就会无效，锁不住**



**解决方法：把sm1()方法定义为static方法，这样synchronized就会变成锁住这个线程类，而不是当前对象，不管是什么对象，只要是这个线程类创建的对象就共用一把锁，达到线程安全**



```java
public class syncLockTest {

    //创建两个线程对象，synchronized(this)就无法发挥作用了，因为subthread1和subthread2是两个不同的对象
    private static subThread1 subThread1=new subThread1();
    private static subThread1 subThread2=new subThread1();
    public static void main(String[] args) {

        Thread t1 = new Thread(subThread1); //传入两个不同的对象
        Thread t2 = new Thread(subThread2);

        t1.setName("111");
        t2.setName("222");
        t1.start();
        t2.start();

    }




}
class subThread1 implements Runnable{

    private static   int count=10;

    @Override
    public void run() {

        sm1();
    }

    //相当于锁住类
    public synchronized static void sm1(){
        while (true){

                if(count>0){
                    count--;
                    System.out.println(Thread.currentThread().getName()+"===>还剩"+count);
                }else {
                    break;
                }
        }
    }
    
}

```

#### 多线程出现异常自动释放锁

```java
public class lockException {

    //出现异常自动释放锁

    private static subThread subThread=new subThread();

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(subThread).start();
        }
    }
}
class subThread implements Runnable{

   

    @Override
    public void run() {


            synchronized (this){
                while (true){
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    throw new RuntimeException();
                }

            }
    }
}
```





#### 死锁（重要）

**死锁的原理：有两个线程分别是线程A、B ，两把锁1、2 ，线程A拿到了锁1，线程B拿到了锁2，线程A没有释放锁1，且必须要拿到锁2才能释放锁1（也就是执行完线程A所有代码），反之，线程B没有释放锁2，但是有一定要获得锁1才能执行完线程B的代码，然后两个线程为了对方的锁一直在僵持，互不相让，这就造成了死锁**

**tryLock解决死锁**

```java
public class dieLock {
    //死锁

    private static subThread3 subThread3=new subThread3();
    public static void main(String[] args) {

        Thread t1 = new Thread(subThread3);
        Thread t2 = new Thread(subThread3);
        t1.setName("t1");
        t2.setName("t2");

        t1.start();
        t2.start();
    }
}
class subThread3 implements Runnable{

    private static final Object obj1=new Object();
    private static final Object obj2=new Object();

    @Override
    public void run() {
        if(Thread.currentThread().getName().equals("t1")){
            synchronized (obj1){
                System.out.println(Thread.currentThread().getName()+"获得了锁1,想去获得锁2");
                synchronized (obj2){
                    System.out.println(Thread.currentThread().getName()+"获得了锁2");

                }


            }
        }else {
            synchronized (obj2){
                System.out.println(Thread.currentThread().getName()+"获得了锁2,想去获得锁1");
                synchronized (obj1){
                    System.out.println(Thread.currentThread().getName()+"获得了锁1");

                }

            }
        }
    }
}
```





### 原子类AtomicXXX

**让两个子线程和一个main线程去共同减少count**

**方式一：**

**输出的结果可能是子线程1一直在减少count，其他的线程在原地等待，其实这是正常现象，因为synchronized关键字是非公平锁，为了保证效率，它会让拿到锁的那个线程更容易再次拿到锁，只有公平锁才会让这些线程都拿到锁，Lock实现类可以通过构造方法去让锁变成公平锁**

```java
public class atomicIntegerTest {

    private static int count=20;
    private static final Object lock=new Object();
    public static void main(String[] args) {

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true){
                    synchronized (lock){
                        if(count<=0){
                            System.out.println("子线程没有count了");
                            break;
                        }
                        count--;
                        System.out.println("子线程减值==还剩="+count);
                    }

                }
            }
        });
        t1.start();
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true){
                    synchronized (lock){
                        if(count<=0){
                            System.out.println("子线程没有count了");
                            break;
                        }
                        count--;
                        System.out.println("子线程减值==还剩="+count);
                    }

                }
            }
        });
        t2.start();


          while (true){
              synchronized (lock){
                  if(count<=0){
                      System.out.println("主线程没有count了");
                      break;
                  }
                  count--;
                  System.out.println("主线程减值==还剩="+count);

              }
          }
    }

}
```



**方式二：**

#### 原子类（AtomicInteger/AtomicLong）

 ```java
public class atomicIntegerTest1 {

    private static  AtomicInteger count=new AtomicInteger(10);
    public static void main(String[] args) {

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true){
                        if(count.get()<=0){
                            System.out.println("子线程1没有count了");
                            break;
                        }
                        count.getAndDecrement();
                        System.out.println("子线程1减值==还剩="+count.get());
                    }
            }
        });
        t1.start();
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true){

                        if(count.get()<=0){
                            System.out.println("子线程2没有count了");
                            break;
                        }
                        count.getAndDecrement();
                        System.out.println("子线程2减值==还剩="+count.get());
                    }


            }
        });
        t2.start();

        while (true){

                if(count.get()<=0){
                    System.out.println("主线程没有count了");
                    break;
                }
                count.getAndDecrement();
                System.out.println("主线程减值==还剩="+count.get());

        }
    }

}
 ```



#### 多线程操作数组线程不安全

```java
public class atomicIntegerArrayTest {


    public static void main(String[] args) {

        atomicArrayThread atomicArrayThread = new atomicArrayThread();

        Thread t1 = new Thread(atomicArrayThread);
        Thread t2 = new Thread(atomicArrayThread);

        t1.start();
        t2.start();


    }

}
class atomicArrayThread implements Runnable{
    private int arr[]=new int[5];

    @Override
    public void run() {
        while (true){
            if(arr[2]==10){
                System.out.println("break");
                break;
            }
            for (int i = 0; i < arr.length; i++) {
                arr[i]++;
            }
            System.out.println(Arrays.toString(arr));
        }


    }
}
```



##### 解决方案（加锁和原子数组）

**方法一：加锁**

```java
public class atomicIntegerArrayTest {


    public static void main(String[] args) {

        atomicArrayThread atomicArrayThread = new atomicArrayThread();

        Thread t1 = new Thread(atomicArrayThread);
        Thread t2 = new Thread(atomicArrayThread);

        t1.start();
        t2.start();

    }

}
class atomicArrayThread implements Runnable{
    private int arr[]=new int[5];

    @Override
    public void run() {
        while (true){
            synchronized (this){ //因为new Thread传入的对象是相同的，所以this调用的对象也是一样，那么synchronized(this)就有效
                if(arr[2]==10){
                    System.out.println("break");
                    break;
                }
                for (int i = 0; i < arr.length; i++) {
                    arr[i]++;
                }
                System.out.println(Arrays.toString(arr));
            }

        }

    }
}
```





**方法二：采用原子数组(弄不出）**

```java

```





### 线程通信

**线程通信有很多种，比如wait()/notify()机制，管道流（pipeXXX）通信，可重入锁的Condition对象的await()/signal()方法等等**

#### wait()/notify()机制实现线程通信

**wait()方法和sleep()方法都是阻塞方法，但是也有区别，wait()方法会释放锁、sleep()方法不会释放锁，并且wait方法必须要有锁对象，由锁对象进行调用wait方法，而sleep方法不用锁对象，wait方法是Object类的，sleep方法是Thread类的。**



**注意：notify/notifyAll和wait方法都是要用同一个锁对象去调用才能唤醒对方。。。。。。**

#### 生产者消费者模式

##### 一个生产者和一个消费者操作值

**需求：我们要实现一个生产者生产好一份菜（value值）就去通知消费者，消费者去拿菜，如果生产者没有生产好，那消费者就等待**

```java
public class ThreadCommunicationTest1 {
    /**
     * 线程通信
     * 一生产者一消费者模式
     */

    private static final Object lock = new Object(); //虽然对象不同，不能用锁this，但是可以锁住一个final对象
    private static String value = "";

    public static void main(String[] args) {
        setThread1 setThread = new setThread1();
        getThread1 getThread = new getThread1();
        Thread t1 = new Thread(setThread);
        Thread t2 = new Thread(getThread);

        t1.start();
        t2.start();

    }

    //生产者
    static class setThread1 implements Runnable {

        @Override
        public void run() {
            while (true){
                synchronized (lock) {

                    if (value == null || value.equals("")) {
                        value = "菜品：" + System.currentTimeMillis();
                        System.out.println(value+"====做好了");
                        try {
                            Thread.sleep(50); //做好之后，不马上去通知顾客，先缓一缓
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        lock.notify();
                    } else {
                        System.out.println("生产者等待菜被拿走");
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }

        }
    }

    //消费者
    static class getThread1 implements Runnable {
        
        @Override
        public void run() {
            while (true){
                synchronized (lock) {
                    if (value == null || value.equals("")) {
                        System.out.println("消费者等待上菜");
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    } else {
                        System.out.println("我收下了=="+value);
                        value="";
                        try {
                            Thread.sleep(50);//吃完之后，不马上点餐
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        lock.notify();
                    }
                }
            }

        }
    }

}
```



##### 多个生产者和多个消费者操作值

**多个线程的生产者消费者模式必须不能用notify了，要用notifyAll**

**多生产者多消费者模式下，不用notify而用notifyAll原因是，如果有三个厨师分别为ABC，A厨师上了一道菜，顾客看到自己的菜到了，然后就把菜拿走了，如果用notify意思就是只告诉厨师A，其他厨师不知道这个顾客拿走了菜，以为自己没有上这道菜，便再次为这个顾客做这道菜，这样显然是不可能的，所以我们要notifyAll，通知所有厨师，说明这个顾客已经拿走菜了，不用再上他的菜，这样问题就解决了。**

```java
public class ThreadCommunicationTest2 {

    private static String value="";
    private static final Object lock=new Object();
    public static void main(String[] args) {
        setValue2 setValue2 = new setValue2();
        getValue2 getValue2 = new getValue2();

        //生产者线程
        Thread t1 = new Thread(setValue2);
        Thread t2 = new Thread(setValue2);
        Thread t3 = new Thread(setValue2);

        //消费者线程
        Thread t4 = new Thread(getValue2);
        Thread t5 = new Thread(getValue2);
        Thread t6 = new Thread(getValue2);


        t1.start();
        t2.start();
        t3.start();
        t4.start();
        t5.start();
        t6.start();

    }

    //生产者线程
    static class setValue2 implements Runnable{


        @Override
        public void run() {
            while (true){ //这里用个死循环，让生产者消费者一直运作
            synchronized (lock){
                    if(value==null||value.equals("")){ //值没有就生产
                        value="菜品："+Thread.currentThread().getName()+"===>"+System.currentTimeMillis();
                        System.out.println(value+"=====>生产好了");
                        try {
                            Thread.sleep(100);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        lock.notifyAll();
                    }else{
                        try {
                            lock.wait(); //菜只能放一个，所以菜满了要等待顾客拿走
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                }

            }

        }
    }

    static class getValue2 implements Runnable{


        @Override
        public void run() {
            while (true){

                synchronized (lock){

                    if(value==null||value.equals("")){

                        try {
                            lock.wait();//如果没有菜就等待
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }else {
                        System.out.println("消费者拿走了"+value);
                        value=""; //菜拿走了，让value等于空字符串
                        try {
                            Thread.sleep(100);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        lock.notifyAll();//通知所有厨师
                    }
                }
            }
        }
    }

}
```



##### 一个生产者和一个消费者操作栈（用List集合去模拟）

```java
public class ThreadCommunicationTest3 {
    /**
     * 一个生产者和一个消费者操作栈（List去模拟）
     */
    private static List<String> list=new ArrayList<>();
    private static final int MAX_SIZE=1; //指定list最大容量
    private static final Object lock=new Object();
    public static void main(String[] args) {
        setValue3 t1 = new setValue3();
        getValue3 t2 = new getValue3();

        new Thread(t1).start();
        new Thread(t2).start();
       
    }

    static class setValue3 implements Runnable{


        @Override
        public void run() {
            while (true){
                synchronized (lock){
                    if(list.size()<MAX_SIZE){
                        String value="(生产者)菜品："+System.currentTimeMillis();
                        list.add(value);
                        System.out.println(Thread.currentThread().getName()+"==>"+value);
                        try {
                            Thread.sleep(50);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        lock.notify();
                    }else {
                        try {
                            lock.wait(); //菜到达指定数量是就不上菜了，等待客人拿走菜
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }

        }
    }
    static class getValue3 implements Runnable{


        @Override
        public void run() {

            while (true){
                synchronized (lock){

                    if(list.size()<=0){

                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }else {
                        String rm = list.remove(0);
                        System.out.println(Thread.currentThread().getName()+"拿走了"+rm);
                        try {
                            Thread.sleep(50);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        lock.notify();
                    }


                }
            }

        }
    }


}

```



##### 多个生产者和多个消费者操作栈（用List集合去模拟）

```java
public class ThreadCommunicationTest4 {

    private static List<String> list=new ArrayList<>();
    private static final Object lock=new Object();
    private static final int MAX_SIZE=3;

    public static void main(String[] args) {
        setValue4 setValue4 = new setValue4();
        getValue4 getValue4 = new getValue4();
        Thread t1 = new Thread(setValue4);
        Thread t2 = new Thread(setValue4);
        Thread t3 = new Thread(setValue4);

        Thread t4 = new Thread(getValue4);
        Thread t5 = new Thread(getValue4);
        Thread t6 = new Thread(getValue4);

        t1.start();
        t2.start();
        t3.start();
        t4.start();
        t5.start();
        t6.start();

    }

    static class setValue4 implements Runnable{


        @Override
        public void run() {

            while (true){
                synchronized (lock){
                    if(list.size()<MAX_SIZE){
                        String value=Thread.currentThread().getName()+"==>"+"菜品："+System.currentTimeMillis();
                        list.add(value);
                        System.out.println(value+" 已经做好了");
                        try {
                            Thread.sleep(50);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        lock.notifyAll();
                    }else {

                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                }

            }



        }
    }

    static class getValue4 implements Runnable{


        @Override
        public void run() {
            while (true) {
                synchronized (lock){
                    if(list.size()==0){
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }else {
                        String rm = list.remove(0);
                        System.out.println("消费者已经拿走了"+rm);
                        try {
                            Thread.sleep(50);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        lock.notifyAll();
                    }


                }


            }



        }
    }

}

```





#### 利用管道流通信

```java
public class pipeStreamTest {
    /**
     * 利用管道流通信
     */
    private static PipedInputStream inputStream=new PipedInputStream();
    private static PipedOutputStream outputStream=new PipedOutputStream();

    static {
        try {
            inputStream.connect(outputStream); //建立连接
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args) throws IOException, InterruptedException {
        inputThread inputThread = new inputThread();
        outputThread outputThread = new outputThread();
        Thread t1 = new Thread(inputThread);
        Thread t2 = new Thread(outputThread);
        t2.start(); //执行写操作
        Thread.sleep(10);
        t1.start();





    }

    static class inputThread implements Runnable{


        @Override
        public void run() {

            try {
                 byte bytes[]=new byte[inputStream.available()];
                 inputStream.read(bytes);
                 String str=new String(bytes,"UTF-8");
                System.out.println("管道输入流已读取===>"+str);

            } catch (IOException e) {
                e.printStackTrace();
            }


        }
    }

    static class outputThread implements Runnable{


        @Override
        public void run() {
            String msg="hello world";
            try {
                outputStream.write(msg.getBytes());
                System.out.println("管道输出流已写入===>"+msg);
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }


}

```

#### Condition通信（用到了显式锁）

**注意：======notify和signal其实差不多，都是用来通知正在等待的线程的，但是notify是随机通知，signal是定向通知**

```java
public class conditionTest {
    /**
     * 利用lock锁里面的condition实现线程通信
     *
     */

    private static ReentrantLock reentrantLock=new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        Condition condition = reentrantLock.newCondition(); //得到condition对象

        new Thread(new Runnable() {
            @Override
            public void run() {

                try {
                    reentrantLock.lock();
                    //使用await和signal必须要锁起来，和wait/notify一样
                    System.out.println("正在等待=========await");
                    condition.await();
                    System.out.println("await结束");

                }catch (Exception e){

                }finally {
                    if(reentrantLock.isHeldByCurrentThread()){
                        reentrantLock.unlock();
                    }
                }



            }
        }).start();

        Thread.sleep(200); //让await先执行，这里模拟一下延时


        new Thread(new Runnable() {
            @Override
            public void run() {

                try {
                    reentrantLock.lock();
                    //使用await和signal必须要锁起来，和wait/notify一样
                   condition.signal();//相当于notify
//                    condition.signalAll(); //相当于notifyAll
                    System.out.println("已通知===");
                }catch (Exception e){

                }finally {
                    if(reentrantLock.isHeldByCurrentThread()){
                        reentrantLock.unlock();
                    }
                }



            }
        }).start();
    }
}
```







### ThreadLocal

```java
public class threadLocalTest {

    /**
     * ThreadLocal（注意：只能存储一对键值对）
     * 原理：创建一个ThreadLocal容器。当我们往里面set值，他会把当前线程作为key去设置值
     * 当我们通过get获取值，ThreadLocal底层会通过key值=Thread.currentThread，去找对应的value
     * ===================
     * 总的来说：也就是每个线程之间的threadLocal互不干扰，里面的值也独立
     *
     *
     */
    private static ThreadLocal<String> threadLocal=new ThreadLocal<>();
    public static void main(String[] args) throws InterruptedException {
        threadLocal.set("主线程=====hello");

        new Thread(new Runnable() {
            @Override
            public void run() {
                threadLocal.set("子线程====world");
                System.out.println(threadLocal.get());
            }
        }).start();

        Thread.sleep(10);

        System.out.println(threadLocal.get());


    }

}

```





### Lock显式锁

**我们使用Lock锁都是使用它的实现类，常用的：可重入锁，可重入读写锁（读锁、写锁）**

**注意：======显式锁最好在finally进行释放锁**

**synchronized和ReentrantLock都是可重入锁**



#### ReentrantLock的使用

**1.创建一个锁对象**

```java
private static ReentrantLock lock=new ReentrantLock(); //创建一个锁对象
```

**我们进入ReentrantLock源码里面看看。**

```java
 public ReentrantLock() {
        sync = new NonfairSync(); //说明这个ReentrantLock默认是非公平锁，和synchronized一样。原因：公平锁会牺牲性能
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) { //说明显式锁Lock可以去指定锁的公平性。通过构造方法去传入true或者false
        sync = fair ? new FairSync() : new NonfairSync();
    }
```



**其实显式锁可以synchronized差不多，只是synchronized是自动释放锁，lock是手动释放，lock对我们技术水平要求的比较高而已，因为释放锁不是随便释放的，错误的释放锁会导致程序受到很大的影响**



```java
public class reentrantLockTest1 {

    /**
     * 可重入锁ReentrantLock
     * @param args
     */
    private static ReentrantLock lock=new ReentrantLock(); //创建一个锁对象
    private static int count=10;
    public static void main(String[] args) {
        lockThread1 lockThread1 = new lockThread1();
        Thread t1 = new Thread(lockThread1);
        Thread t2 = new Thread(lockThread1);
        t1.start();
        t2.start();
    }

    static class lockThread1 implements Runnable{

        @Override
        public void run() {

            while (true){

                lock.lock(); //给代码上锁
                try {
                    if(count<=0){
                        System.out.println("程序结束======");
                        break;
                    }
                    count--;
                    System.out.println(Thread.currentThread().getName()+",count="+count);

                }catch (Exception e){

                }finally {
                    if(lock.isHeldByCurrentThread()){ //如果这个锁被当前线程拥有
                        lock.unlock(); //释放锁
                    }
                }
            }
 
        }
    }

}

```



#### 可重入锁特性

```java
public class reentrantLockTest2 {
    /**
     * 可重入锁的特性：假如A线程获得了锁1，此时锁1还没有被释放，这个线程又可以继续去获得这个锁。这就是锁的可重入性
     * =====获得了多少个锁就要释放多少次
     */

    private static ReentrantLock lock=new ReentrantLock();
    public static void main(String[] args) {

        lockThread2 lockThread2 = new lockThread2();

        Thread t1 = new Thread(lockThread2);
        Thread t2 = new Thread(lockThread2);

        t1.start();
        t2.start();

    }


    static class lockThread2 implements Runnable{


        @Override
        public void run() {

            try {
                lock.lock();
                System.out.println("获得了锁lock");
                lock.lock();
                System.out.println("再次获得锁lock");


            }catch (Exception e){

            }finally {
                if(lock.isHeldByCurrentThread()){
                    lock.unlock(); //获得了多少个锁就要释放多少次
                    lock.unlock();
                }
            }



        }
    }


}

```



#### tryLock方法（解决死锁）

**tryLock方法可以有效的解决死锁，因为他会在得不到锁的时候放弃获取，死锁的原因就是互相持有对方想要的锁，而都不肯释放**

**进入reentrantLock找到trylock构造方法**

```java
public boolean tryLock() {
        return sync.nonfairTryAcquire(1);
    }

public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
```

**这两个方法的区别是，第一个不加参数的意思就是尝试获取锁，如果获取不到立刻放弃获取。第二个会等待指定时间，如果过了指定时间还是没有获取到这个锁的使用权，则放弃获取**

```java
public class trylockTest {

    private static ReentrantLock lock=new ReentrantLock(false);
    private static int count=20;
    public static void main(String[] args) {
        tryLockThread tryLockThread = new tryLockThread();
        Thread t1 = new Thread(tryLockThread);
        Thread t2 = new Thread(tryLockThread);
        t1.start();
        t2.start();


    }
    static class tryLockThread implements Runnable{


        @Override
        public void run() {

            try {
               out:  while (true){
                    boolean flag = lock.tryLock(); //返回值就是获取到该锁没有
                   if(count<=0)
                       break ;
                     if(flag){

                         count--;
                         System.out.println(Thread.currentThread().getName()+"获得锁了==="+count);

                     }else {
                         System.out.println(Thread.currentThread().getName()+"没有获得锁");
                     }

                }

            }catch (Exception e){

            }finally {
                if(lock.isHeldByCurrentThread()){
                    lock.unlock();
                }
            }
        }
    }


}
```





```java
public class trylockTest {

    private static ReentrantLock lock=new ReentrantLock(false);
    public static void main(String[] args) {
        tryLockThread tryLockThread = new tryLockThread();
        Thread t1 = new Thread(tryLockThread);
        Thread t2 = new Thread(tryLockThread);
        t1.start();
        t2.start();


    }
    static class tryLockThread implements Runnable{


        @Override
        public void run() {

            try {
                boolean flag = lock.tryLock(1, TimeUnit.SECONDS);
                System.out.println(Thread.currentThread().getName()+"==="+flag);
                Thread.sleep(1001);
            }catch (Exception e){

            }finally {
                if(lock.isHeldByCurrentThread()){
                    lock.unlock();
                }
            }
        }
    }


}
```





#### 读写锁（ReadWriteLock）

##### 读读共享

**只有读锁时，是共享的，也就是可以同时多个线程进入readLock内**

```java
public class readLockTest {
    /**
     * 只有读锁时，是共享的，也就是可以同时多个线程进入readLock内
     */

    //获取读锁
    private static ReentrantReadWriteLock.ReadLock readLock=new ReentrantReadWriteLock().readLock();
    public static void main(String[] args) {
        readThread1 readThread1 = new readThread1();
        Thread t1 = new Thread(readThread1);
        Thread t2 = new Thread(readThread1);
        Thread t3 = new Thread(readThread1);
        t1.start();
        t2.start();
        t3.start();

    }

    static class readThread1 implements Runnable{


        @Override
        public void run() {
            try {
                readLock.lock();
                System.out.println(Thread.currentThread().getName()+"===>"+System.currentTimeMillis());
                Thread.sleep(1000);
                System.out.println("睡眠结束");
            }catch (Exception e){

            }finally {
                readLock.unlock();
            }


        }
    }

}

```



##### 写写互斥

**写锁就相当于互斥锁（synchronized、lock）**

```java
public class writeLockTest {
    /**
     * 写写互斥（写锁就相当于互斥锁（synchronized、lock））
     */

    //创建写锁
    private static ReentrantReadWriteLock.WriteLock writeLock=new ReentrantReadWriteLock().writeLock();

    public static void main(String[] args) {
        writeLockThread writeLockThread = new writeLockThread();
        Thread t1 = new Thread(writeLockThread);
        Thread t2 = new Thread(writeLockThread);
        Thread t3 = new Thread(writeLockThread);

        t1.start();
        t2.start();
        t3.start();


    }
    static class writeLockThread implements Runnable{


        @Override
        public void run() {
            try {
                writeLock.lock();
                System.out.println(Thread.currentThread().getName()+"====>"+System.currentTimeMillis());
                Thread.sleep(200);
                System.out.println("睡眠结束");
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                if(writeLock.isHeldByCurrentThread()){
                    writeLock.unlock();
                }
            }

        }
    }

}


```



##### 读写互斥

**同时有读锁和写锁就会互斥**



```java
public class readWriteLockTest {
    /**
     * 读写锁最好是一个线程用读锁，一个线程用写锁，要是多一个线程用锁，可能会失效，也就是不具有排他性
     */

    private static ReentrantReadWriteLock readWriteLock=new ReentrantReadWriteLock();
    private static Lock readlock=null;
    private static Lock writeLock=null;
    static {
        readlock=readWriteLock.readLock();
        writeLock=readWriteLock.writeLock();
    }
//    private static ReentrantReadWriteLock.ReadLock readlock=new ReentrantReadWriteLock().readLock();
//    private static ReentrantReadWriteLock.WriteLock writeLock=new ReentrantReadWriteLock().writeLock();
    public static void main(String[] args) {
        readWriteThread readWriteThread = new readWriteThread();

        Thread t1 = new Thread(readWriteThread);
        t1.start();

        try {
            writeLock.lock();
            System.out.println("===正在写===");
            Thread.sleep(2000);

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            System.out.println("===写完了===");
            writeLock.unlock();
        }
    }

    static class readWriteThread implements Runnable{


        @Override
        public void run() {
            try {
                readlock.lock();
                System.out.println(Thread.currentThread().getName()+"==读锁:"+System.currentTimeMillis());
                Thread.sleep(2000);
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                System.out.println(Thread.currentThread().getName()+"===释放锁"+System.currentTimeMillis());
                readlock.unlock();
            }

        }
    }

    }
```





### 线程池ThreadPool

#### 线程池的简单使用

**固定大小的线程池**

```java
public class threadPoolTest01 {
    /**
     * 线程池的作用：1.可以控制并发，通过设置线程池的线程数量
     * 2.因为线程的创建和销毁是会耗性能的，线程池里面的线程是可以复用的，也就是假如线程池创建了1号2号线程。
     * 里面有100个runnable方法，也就是要执行50次，一次2个线程去执行，在这过程中，线程1号2号不会被销毁
     */
    private static int i;
    public static void main(String[] args) {
        //线程池的使用
        //固定线程数量的线程池
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        for (int j = 0; j < 100; j++) {
            executorService.execute(new Runnable() { //一个execute执行一个线程执行，所以我们为了演示，可以在外面用for
                @Override
                public void run() {

                    System.out.println(Thread.currentThread().getId()+"====>");
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }



            });
        }
  			executorService.shutdown(); //关闭线程池

    }

 
}

```



**缓存线程池和任务线程池**

```java
public class threadPoolTest02 {
    public static void main(String[] args) {
//        ExecutorService executorService = Executors.newCachedThreadPool();
//        for (int i = 0; i < 20; i++) {
//            executorService.execute(new Runnable() {
//                @Override
//                public void run() {
//                    System.out.println(Thread.currentThread().getId());
//                    try {
//                        Thread.sleep(100);
//                    } catch (InterruptedException e) {
//                        e.printStackTrace();
//                    }
//                }
//            });
//        }
//
//        executorService.shutdown();

        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(5);
//        scheduledExecutorService.schedule(new Runnable() {
//            @Override
//            public void run() {
//                System.out.println(Thread.currentThread().getId());
//
//            }
//        },2, TimeUnit.SECONDS);    //推迟2秒执行线程方法

        scheduledExecutorService.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getId());
            }
        },5,1,TimeUnit.SECONDS); //这里的意思是，开始等5秒才会第一次执行这个方法，过后一直都是1秒执行一次



    }
}

```
 
### Java设计模式

### 创建型

#### 单例模式

**单例模式步骤：**

**1.构造器私有**

**2.创建对象（比如饿汉式就是先new，懒汉式就是判断为null再new）**

**3.返回实例对象（都是一样）**



##### 饿汉式

**所谓的饿汉式，也就是提前new好对象，再提供一个方法，返回这个new好的对象实例。还有要把构造器私有化，通过方法new对象，控制到只有一个实例对象**



**饿汉式缺点：虽然是线程安全，但是会浪费资源，因为我们的饿汉式是一开始就new对象，如果没有人使用到这个对象，那就是一种资源的浪费，所以下面我们要讲懒汉式，懒汉式线程不安全，但是不会造成资源浪费，因为只有用到了才会new对象**



**代码实现：**

**1：实体类只需要正常写：**

```java
public class person {

    private String id;
    private String name;
    private int age;

    public person() {
    }

    public person(String id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

```



**2：单例类：**

```java
public class hungry {

    /**
     * 单例模式-饿汉式
     */
    //1.创建static final对象
    private static final person person=new person();

    //2.构造器私有,为了让人只能用类去调用getInstance方法获取实例，而不能new这个单例的对象
    private hungry(){

    }


    //3.返回对象
    public static person getInstance(){
        return person;
    }


}

```

**3.测试：**

```java
 public static void main(String[] args) {

         
        //测试单例
        person p1 = hungry.getInstance();
        person p2 = hungry.getInstance();

        //hashcode相同证明是同一个对象
        System.out.println("p1.hashCode:"+p1.hashCode());
        System.out.println("p2.hashCode:"+p2.hashCode());
        System.out.println("p1==p2:"+(p1==p2));

    }
```

**4.结果：**

```tex
p1.hashCode:1690716179
p2.hashCode:1690716179
p1==p2:true
```





##### 饿汉式在JDK中的Runtime类运用

**查看Runtime源码：**

```java
public class Runtime {
    private static final Runtime currentRuntime = new Runtime();

    private static Version version;

    /**
     * Returns the runtime object associated with the current Java application.
     * Most of the methods of class {@code Runtime} are instance
     * methods and must be invoked with respect to the current runtime object.
     *
     * @return  the {@code Runtime} object associated with the current
     *          Java application.
     */
    public static Runtime getRuntime() {
        return currentRuntime;
    }

    /** Don't let anyone else instantiate this class */
    private Runtime() {}
```



**这就是一种标准的单例模式-饿汉式**



##### 懒汉式（不加锁，不安全）

**懒汉式是线程不安全的，但是他具有懒加载机制，也就是用到才new对象，这样不会造成资源的浪费**

```java
public class lazyNotSync {

    /**
     * 单例模式-懒汉式（不加锁） 线程不安全
     */
    //1.首先，单例模式要把构造器私有
    private lazyNotSync(){

    }

    //2.懒汉式的对象置为null
    private static person person=null;

    //3.返回实例
    public static person getInstance(){

        if(person==null){
            person=new person(); //这个new对象并赋值不是原子操作，所以在多线程是不安全的
            return person;
        }else {
            return person;
        }

    }



}
```



##### 测试懒汉式不加锁的线程不安全性

```java
class lazyNotSyncMain{

    public static void main(String[] args) throws InterruptedException {
        //这里不能用ArrayList，因为ArrayList是线程不安全的，所以在多线程的add，会导致少添加的情况
        CopyOnWriteArrayList<Object> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 5; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    list.add(lazyNotSync.getInstance());
                }
            }).start();
        }

        Thread.sleep(100);
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i).hashCode());
        }


    }
}
```



**结果hashcode不一致，说明不是一个对象：**

```tex
1929600551
1690716179
1053782781
1211888640
564160838
```







##### 懒汉式（加锁，安全）

```java
public class lazySync {

    /**
     * 单例模式-懒汉式，加锁，保证线程安全，但是呢，加了锁之后性能会下降
     *
     */

    //1.构造器私有
    private lazySync(){

    }

    //2.创建对象
    private static person person=null;
    private static final Object lock=new Object();

    //3.返回实例
    public static  person getInstance(){

        synchronized (lock){
            if(person==null){
                person=new person();
                return person;
            } else {
                return  person;
            }
        }

    }


}
```





##### 测试安全性

```java
class lazySyncMain{


    public static void main(String[] args) {
        
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                System.out.println(lazySync.getInstance().hashCode());
            }).start();
        }


    }


}
```



**结果：**

```tex
1226907195
1226907195
1226907195
1226907195
1226907195
1226907195
1226907195
1226907195
1226907195
1226907195
```







##### 利用静态内部类

```java
public class staticClass {

    /**
     * 利用静态内部类的特性来实现单例模式（推荐）
     * 优点：线程安全，懒加载（节省资源）
     * 可以说是饿汉式和懒汉式更好的一种
     * =====
     * 写法类似于饿汉式。。只是把new对象放到一个私有的静态内部类中
     *
     * =====
     * 静态内部类的特性：
     * 当classloader时静态内部类不会随着加载，当我们去调用时才会加载（懒加载）
     */

    //1.构造器私有
    private staticClass(){

    }

    //如果这段代码不放到静态内部类中，它将随着类加载而new对象（占资源）
//    private static final person person=new person();


    //2.创建对象（只不过是在静态内部类中创建对象）
    private static class personStatic{
        private static final person person=new person();
    }


    //3.返回对象

    public static person getInstance(){

        return personStatic.person;
    }





}
```

**测试：**

```java
class staticClassMain{


    public static void main(String[] args) {

        for (int i = 0; i < 5; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(staticClass.getInstance().hashCode());
                }
            }).start();

        }

    }
}
```









##### 利用枚举(不太懂)

 





#### 原型模式(多例)

**原型模式其实就是复制的意思，包含着深拷贝和浅拷贝。浅拷贝复制的引用数据类型变量只是地址，深拷贝则是内容**

**实现原型模式的实体类要实现Cloneable接口**

**注意：实现了Cloneable的类才能拷贝（克隆）**

##### 浅拷贝

```java
public class person implements Cloneable {

    private String id;
    private String name;
    private int age;
    private Firend firend; //朋友  ---模拟深拷贝和浅拷贝

    public person() {
    }

    public person(String id, String name, int age, Firend firend) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.firend = firend;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Firend getFirend() {
        return firend;
    }

    public void setFirend(Firend firend) {
        this.firend = firend;
    }

    @Override
    public String toString() {
        return "person{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", firend=" + firend +
                '}';
    }

    //浅拷贝
    @Override
    protected person clone() throws CloneNotSupportedException {
        person person = (person) super.clone(); //克隆对象
        return person;
    }



}
```



```java
public class Firend {

    private String id;
    private String name;

    public Firend() {
    }

    public Firend(String id, String name) {
        this.id = id;
        this.name = name;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Firend{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}

```



```java
public class prototypeMain {


    public static void main(String[] args) throws CloneNotSupportedException {

        person person = new person("001","张三",18,new Firend("999","小明同学"));

        person p1 = person.clone();

        System.out.println("person是否=p1:"+(person==p1)); //false ，说明克隆出来的对象不是同一个对象

        System.out.println("person:"+person);
        System.out.println("p1:"+p1);

        //说明我们的firend是浅拷贝
        System.out.println("person.getFirend().hashCode()==p1.getFirend().hashCode():"+(person.getFirend().hashCode()==p1.getFirend().hashCode()));


    }

```



**结果：**

```tex
person是否=p1:false
person:person{id='001', name='张三', age=18, firend=Firend{id='999', name='小明同学'}}
p1:person{id='001', name='张三', age=18, firend=Firend{id='999', name='小明同学'}}
person.getFirend().hashCode()==p1.getFirend().hashCode():true
```





##### 深拷贝

**深拷贝有多种方法，这里我们只使用一种，也是最简单的一种**

**person和Firend都要实现Cloneable和重写clone方法。。。。。。。。。。。。。才能实现深拷贝**

```java
public class person implements Cloneable {

    private String id;
    private String name;
    private int age;
    private Firend firend; //朋友  ---模拟深拷贝和浅拷贝

    public person() {
    }

    public person(String id, String name, int age, Firend firend) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.firend = firend;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Firend getFirend() {
        return firend;
    }

    public void setFirend(Firend firend) {
        this.firend = firend;
    }

    @Override
    public String toString() {
        return "person{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", firend=" + firend +
                '}';
    }

    //深拷贝
    @Override
    protected person clone() throws CloneNotSupportedException {
        person person = (person) super.clone(); //克隆对象

        //Firend也要实现Cloneable和clone方法
        //深拷贝：把引用数据类型变量单独的克隆并复制到person.firend
        person.firend=firend.clone();
        return person;
    }



}
```

```java
public class Firend implements Cloneable {

    private String id;
    private String name;

    public Firend() {
    }

    public Firend(String id, String name) {
        this.id = id;
        this.name = name;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    protected Firend clone() throws CloneNotSupportedException {
        Firend firend = (Firend) super.clone();
        return firend;
    }

    @Override
    public String toString() {
        return "Firend{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}

```



```java
public class prototypeMain {


    public static void main(String[] args) throws CloneNotSupportedException {

        person person = new person("001","张三",18,new Firend("999","小明同学"));

        person p1 = person.clone();

        System.out.println("person是否=p1:"+(person==p1)); //false ，说明克隆出来的对象不是同一个对象

        System.out.println("person:"+person);
        System.out.println("p1:"+p1);

        //说明我们的firend是深拷贝
        System.out.println("person.getFirend().hashCode()==p1.getFirend().hashCode():"+(person.getFirend().hashCode()==p1.getFirend().hashCode()));


    }
}

```

**结果：**

```tex
person是否=p1:false
person:person{id='001', name='张三', age=18, firend=Firend{id='999', name='小明同学'}}
p1:person{id='001', name='张三', age=18, firend=Firend{id='999', name='小明同学'}}
person.getFirend().hashCode()==p1.getFirend().hashCode():false
```









#### 工厂模式

**工厂模式分为简单工厂和工厂方法,工厂模式和抽象工厂模式不是一个概念**

**总结：工厂模式只是生产同一类产品（比如手机工厂生产手机对象，电脑工厂生产电脑对象）**







##### 简单工厂模式(推荐使用)

**总结：主要依靠if elseif 在一个大的工厂类中根据不同的传参来new不同的对象，这个大的工厂必须要是一个品类，比如手机工厂和电脑工厂要分开，后面的抽象工厂模式才能通过一个品牌分类，这里简单工厂和工厂方法只能一个种类的工厂（但是可以写多个工厂类）**

**实体类要提供一个类型的接口，工厂模式返回对象需要多态，所以要有接口或者父类**

```java
public interface phone {
    //使用接口来实现多态写法

    String getPhoneName();

}
```

```java
public class iphone implements phone {

    /**
     * 苹果手机
     */
    private String id;
    private String name;
    private double price;

    public iphone() {
    }

    public iphone(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public void setId(String id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "iphone{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }

    @Override
    public String getPhoneName() {
        return this.name;
    }
}


```



```java
public class xiaomiPhone implements phone {

    /**
     * 小米手机
     */
    private String id;
    private String name;
    private double price;

    public xiaomiPhone() {
    }

    public xiaomiPhone(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public void setId(String id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "xiaomiPhone{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }

    @Override
    public String getPhoneName() {
        return this.name;
    }
}

```





**简单工厂（核心代码实现）**

```java
public class phoneFactory {

    /**
     * 简单工厂模式
     * 缺点：不满足设计模式原则-开闭原则（对扩展性开，对修改关闭）
     */

    //简单工厂根据传参去判断new哪个对象,因为这里要使用多态写法，所以下面要定义一个接口，哪怕空接口都行，extends也行
    public static phone getPhone(String name){

        if(name==null||name.equals(""))
            return null;
        if(name.equals("小米"))
            return new xiaomiPhone("mi-001","小米11",4999.0);
        else if(name.equals("苹果"))
            return new iphone("iphone-888","iPhone12 Pro max",10999.0);
        return null;
    }




}
```

**测试**

```java
class phoneFactoryMain{

    public static void main(String[] args) {
        //通过工厂就可以获得对象
        phone p1 = phoneFactory.getPhone("");
        phone p2 = phoneFactory.getPhone("小米");
        phone p3 = phoneFactory.getPhone("苹果");
        System.out.println(p1);
        System.out.println(p2.getPhoneName());
        System.out.println(p3.getPhoneName());
    }

}
```

```tex
null
小米11
iPhone12 Pro max
```



##### 简单工厂模式弊端

**试想，当我们需要添加一款手机的时候，我们需要在工厂类的if else 里面添加new新手机的代码，这样就违反了设计模式的开闭原则了，这时候我们如果很介意这个开闭原则的话，可以使用工厂方法模式**





##### 工厂方法模式（拆分简单工厂）

**总结：工厂方法也就是为了解决简单工厂不满足设计模式原则中的开闭原则，工厂方法不采用简单工厂那种在一个大工厂（单一类型，比如手机工厂、电脑工厂要分开不同的工厂）中通过if elseif 根据不同的传参来new不同的对象了，而是把这个大的工厂拆分。拆分成一个工厂接口，**



**简单工厂是一个大的综合一个类型的工厂，而工厂方法模式是各自品牌类型的东西单独建立一个工厂，这样更符合我们日常生活的情况，比如手机里面有小米手机、苹果手机，这样工厂又会拆分成小米手机工厂、苹果手机工厂**

```java
public abstract class factory {

    /**
     * 这个工厂是生产手机，所以我们要提供一个抽象方法来生产手机
     */

    //抽象类的获取手机方法，一定不能指定获取什么手机，要用多态。
    abstract phone getPhoneByFactory();



}
```

**苹果手机工厂**

```java
public class iPhoneFactory extends factory {

    /**
     * 苹果手机工厂
     */


    @Override
    phone getPhoneByFactory() {
        return new iphone("iphone-888","iPhone12 Pro max",10999.0);
    }



}

```

**小米手机工厂**

```java
public class xiaomiFactory extends factory{

    /**
     * 小米手机工厂
     */
    @Override
    phone getPhoneByFactory() {
        return new xiaomiPhone("mi-001","小米11",4999.0);
    }
}
```



**苹果手机实体类**

```java
public class iphone implements phone {

    /**
     * 苹果手机
     */
    private String id;
    private String name;
    private double price;

    public iphone() {
    }

    public iphone(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public void setId(String id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "iphone{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}
```



**小米手机实体类**

```java
public class xiaomiPhone implements phone{

    //小米手机
    private String id;
    private String name;
    private double price;

    public xiaomiPhone() {
    }

    public xiaomiPhone(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public void setId(String id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "xiaomiPhone{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }

}

```

**手机接口：**

```java
public interface phone {
}
```



**利用各自的手机工厂创建手机对象**

```java
public class factoryMethodMain {

    public static void main(String[] args) {

        factory xiaomiFactory = new xiaomiFactory();
        factory iPhoneFactory = new iPhoneFactory();

        System.out.println(xiaomiFactory.getPhoneByFactory());
        System.out.println(iPhoneFactory.getPhoneByFactory());


    }

}
```







#### 抽象工厂模式（很大的工厂）

**总结：抽象工厂模式是生产一个品牌的任何产品（比如某一个抽象工厂，小米工厂去实现它，那么小米工厂就是抽象工厂的子类，也就是说小米工厂可以小米品牌下的所有商品，也就是说这个小米工厂可以生产小米手机、小米笔记本、小米电脑、小米耳机等等。。。），而工厂模式不可以，工厂模式只能生产同一类产品，比如就生产笔记本，但是。如果抽象工厂定义的生产产品只有一类品牌产品，那么这个抽象工厂模式就和工厂模式没有区别了。（当且仅当抽象工厂只有一个抽象方法时）**



**需求：小米公司和华为公司要生产手机和电脑。**

**定义抽象工厂：**

```java
public abstract class abstractFactory {

    /**
     * 抽象工厂模式的抽象工厂：生产手机和电脑
     * 如果是工厂方法模式的抽象工厂：只能生产手机或者只生产电脑
     */

    public abstract phone getPhone();  //生产手机


    public abstract computer getComputer();  //生产电脑



}
```



**定义华为品牌的工厂（生产多个类型产品）**

```java
public class huaweiFactory extends abstractFactory {


    @Override
    public phone getPhone() {
        return new huaweiPhone("h-05","华为Mate40 pro",5388.0);
    }

    @Override
    public computer getComputer() {
        return new huaweiComputer("h-33","华为笔记本电脑",5800.0);
    }


}
```

**定义小米品牌的工厂（生产多个类型产品）**

```java
public class xiaomiFactory extends abstractFactory {


    @Override
    public phone getPhone() {
        return new xiaomiPhone("x-01","小米11",3999.0);
    }

    @Override
    public computer getComputer() {
        return new xiaomiComputer("x-22","小米笔记本电脑",5213.0);
    }


}
```

**手机接口：**

```java
public interface phone {


}

```

**电脑接口：**

```java
public interface computer {
    
}

```

**实体类：**

```java
public class xiaomiPhone implements phone {

    /**
     * 小米手机
     */
    private String id;
    private String name;
    private double price;

    public xiaomiPhone() {
    }

    public xiaomiPhone(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
```



```java
public class huaweiComputer implements computer {

    private String id;
    private String name;
    private double price;


    public huaweiComputer() {
    }

    public huaweiComputer(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
```



**测试类：**

```java
public class abstractFactoryMain {

    /**
     * 抽象工厂模式
     */

    public static void main(String[] args) {

        //创建小米的大工厂
        xiaomiFactory xiaomiFactory = new xiaomiFactory();
        //创建华为的大工厂
        huaweiFactory huaweiFactory = new huaweiFactory();
        //从小米的大工厂中生产小米手机和小米笔记本
        phone xiaomiPhone = xiaomiFactory.getPhone();
        computer xiaomComputer = xiaomiFactory.getComputer();
        System.out.println(xiaomiPhone);
        System.out.println(xiaomComputer);
        //从华为的大工厂中生产华为手机和华为笔记本
        System.out.println("=======================");
        phone huaweiPhone = huaweiFactory.getPhone();
        computer huaweiComputer = huaweiFactory.getComputer();
        System.out.println(huaweiPhone);
        System.out.println(huaweiComputer);


    }


}

```







#### 建造者模式

**用于创建复杂对象，比如构造方法多变的情况下使用建造者模式很好用**

**建造者所需角色：抽象建造者、具体建造者、指挥者（非必要,可有可无：用于拼装建造的顺序）、测试**

**比如有一个项目：我们要建造一台电脑，里面有cpu、显卡、内存条、键盘等等。。（使用建造者模式实现这个项目）**

**抽象建造者：**

```java
public abstract class builder {

    //抽象建造者提供建造对象所需要的方法

    /*
    实现链式编程，addCPU等方法要返回一个builder对象
     */
    public abstract builder addCPU(String cpu); //添加CPU

    public abstract builder addXianka(String xianka); //添加显卡

    public abstract builder addShubiao(String shubiao); //添加鼠标

    public abstract builder addKeyboard(String keyboard);//添加键盘

    public abstract builder addMemory(String memory); //添加内存条

    public abstract computer builderComputer(); //返回构建的computer对象

}
```

**具体建造者：**

```java
public class computerBuilder extends builder {

    private computer computer=new computer();

    //给电脑默认属性
    public computerBuilder(){
        computer.setCpu("i3");
        computer.setKeyboard("联想键盘");
        computer.setMemory("金士顿8g内存");
        computer.setShubiao("华硕鼠标");
        computer.setXianka("GTX1060");
    }


    @Override
    public builder addCPU(String cpu) {
        computer.setCpu(cpu);
        return this; //返回当前调用者的对象，实现链式编程
    }

    @Override
    public builder addXianka(String xianka) {
        computer.setXianka(xianka);
        return this;
    }

    @Override
    public builder addShubiao(String shubiao) {
        computer.setShubiao(shubiao);
        return this;
    }

    @Override
    public builder addKeyboard(String keyboard) {
        computer.setKeyboard(keyboard);
        return this;
    }

    @Override
    public builder addMemory(String memory) {
        computer.setMemory(memory);
        return this;
    }

    @Override
    public computer builderComputer() {
        return computer;
    }


}

```

```java
public class computer {
    private String cpu;
    private String xianka;
    private String shubiao;
    private String keyboard;
    private String memory; //内存

    public computer() {
    }

    public computer(String cpu, String xianka, String shubiao, String keyboard, String memory) {
        this.cpu = cpu;
        this.xianka = xianka;
        this.shubiao = shubiao;
        this.keyboard = keyboard;
        this.memory = memory;
    }

    public String getCpu() {
        return cpu;
    }

    public void setCpu(String cpu) {
        this.cpu = cpu;
    }

    public String getXianka() {
        return xianka;
    }

    public void setXianka(String xianka) {
        this.xianka = xianka;
    }

    public String getShubiao() {
        return shubiao;
    }

    public void setShubiao(String shubiao) {
        this.shubiao = shubiao;
    }

    public String getKeyboard() {
        return keyboard;
    }

    public void setKeyboard(String keyboard) {
        this.keyboard = keyboard;
    }

    public String getMemory() {
        return memory;
    }

    public void setMemory(String memory) {
        this.memory = memory;
    }

    @Override
    public String toString() {
        return "computer{" +
                "cpu='" + cpu + '\'' +
                ", xianka='" + xianka + '\'' +
                ", shubiao='" + shubiao + '\'' +
                ", keyboard='" + keyboard + '\'' +
                ", memory='" + memory + '\'' +
                '}';
    }
}

```



**director（可写可不写）：也可以直接操作具体建造者，下面会有用director和不用director的写法：**

```java
public class director {
    /**
     * 指挥者。用来拼装具体建造者实现的方法
     */

    private builder builder;

//    私有化构造器，也可以不这么写
    private director(){

    }

    public director(builder builder){
        this.builder=builder;
    }



    //拼装方法
    public computer builder(){

        builder.addCPU("director_cpu");
        builder.addXianka("director_xianka");

        return builder.builderComputer();
    }




}

```



**Main测试：**

```java
public class builderMain {


    public static void main(String[] args) {

        System.out.println("不采用director类"); //不采用director类
        builder builder=new computerBuilder();

        computer computer = builder.builderComputer();
        System.out.println(computer);

        computer computer1 = builder.addCPU("i5").addXianka("RTX2080").builderComputer();
        System.out.println(computer1);

        computer computer2 = builder.addCPU("i7").addXianka("RTX2090").addMemory("金士顿32g").addShubiao("华为鼠标").builderComputer();
        System.out.println(computer2);

        System.out.println("=============");
        System.out.println("采用director类"); //采用director类
        director director = new director(new computerBuilder());
        computer builder1 = director.builder();
        System.out.println(builder1);



    }

}

```











### 结构型



#### 代理模式



##### 静态代理

**被代理的类**

```java
public class logger implements Mylogger {

    /**
     * 静态代理：
     *  当我们有个接口的实现类，其中需要对实现类的方法进行增强而不修改源代码，我们可以用代理模式
     *  需求：我们想不改变源代码的情况下对下面 System.out.println("写入日志。");的前面和后面加入一个输出语句
     *
     *  实现：我们只需要创建一个logger的代理类，并实现Mylogger接口，重写方法即可
     */
    @Override
    public void writerLog() {

        System.out.println("写入日志。");

    }



}
```



**接口：**

```java
public interface Mylogger {

    public void writerLog();

}
```



**代理类：**

```java
public class proxyLogger implements Mylogger {

    private Mylogger mylogger;


    public proxyLogger(Mylogger logger){
        this.mylogger=logger;
    }

    @Override
    public void writerLog() { //增强的方法
        System.out.println("logger被代理了===前");
        mylogger.writerLog(); //需要增强的方法
        System.out.println("logger被代理了===后");
    }
}
```



**测试：**

```java
public class staticProxyMain {


    public static void main(String[] args) {

        proxyLogger proxyLogger = new proxyLogger(new logger());

        proxyLogger.writerLog();

    }


}

```



**结果：**

```tex
logger被代理了===前
写入日志。
logger被代理了===后
```







##### 动态代理（JDK动态代理）



**要实现如上的结果，我们这次不用静态代理，而是用JDK动态代理**



**其实所谓的动态代理，也就是不用写代理类了，而是又JDK底层通过反射在内存中帮你创建好了一个代理类**



```java
public class logger implements loggers {
    /**
     * JDK动态代理：
     * 我们无需创建代理类，而是有JDK底层的反射帮我们在内存中自动创建
     */

    @Override
    public void writerlogger() {

        System.out.println("写入日志。。");


    }
}
```



```java
public interface loggers {


    void writerlogger();




}
```



**动态代理：**

```java
public class loggerProxyMain {

      private static logger logger=new logger();

    public static void main(String[] args) {

        /**
         * public static Object newProxyInstance(ClassLoader loader,
         *                                           Class<?>[] interfaces,
         *                                           InvocationHandler h)
         */
     loggers loggers= (com.design.proxy.dongtaiProxy.loggers) Proxy.newProxyInstance(logger.class.getClassLoader(), logger.class.getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        /*
                        第一个参数：被代理的类的对象
                        第二个参数：方法参数args
                         */
                        System.out.println("代理开始====");
                        //如果用代理实例调用方法，将会在这里执行
                        Object invoke = method.invoke(logger, args);//不返回这个对象的话，代理实例为null
                        System.out.println("代理结束");
                        return invoke; //返回代理实例，并返回到loggers对象中

                    }
                });

     //****注意：进入invoke方法是有前提的，就是我们要使用代理实例对象loggers，如果不使用它就不会进入到invoke方法中
        //输入logger对象也行，也可以让他进入invoke方法
//        System.out.println(loggers);

        /*
        当我们去调用代理的方法时，method.invoke就会监听到，并执行被代理的方法
         */
        loggers.writerlogger();



    }



}

```

















#### 装饰器模式

**我们可以把装饰器分为主菜和配菜，配菜就是装饰器，而主菜就是被装饰的**



**公式：创建一个抽象类（作为被装饰者的父类），被装饰者类去继承它，然后创建装饰者抽象类并继承被装饰者的父类，再有装饰者子类去继承装饰者抽象类。**



**案例：**

**被装饰者的抽象类，在这里是最高的类**

```java
public abstract class breakFast {


    public abstract int cost(); //价格



}
```

**被装饰者具体类**

```java
public class rice extends breakFast {
    /**
     * 主食：米饭
     */

    @Override
    public int cost() {
        return 5;
    }

 
}
```



```java
public class noodles extends breakFast {
    /**
     *主食： 面条
     */

    @Override
    public int cost() {
        return 10;
    }


}
```

**装饰者抽象类：继承与被装饰者的父类**

```java
public abstract class decorations extends breakFast{



    public abstract int cost(); //装饰者的钱

 
}
```

 

**装饰者类**

```java
public class coke extends decorations {


    private breakFast breakFast;
    private final int COKE_COST=2; //coke的价格

    public coke(breakFast breakFast){
        this.breakFast=breakFast;
    }



    @Override
    public int cost() {
        return breakFast.cost()+COKE_COST;
    }
}
```



```java
public class milk extends decorations {


    private breakFast breakFast;
    private final int MILK_COST=3; //milk的价格

    public milk(breakFast breakFast){
        this.breakFast=breakFast;
    }


    @Override
    public int cost() {
        return breakFast.cost()+MILK_COST;
    }


}
```

**测试：**

```java
public class main {


    public static void main(String[] args) {
        rice rice = new rice(); //创建被装饰者对象
        milk milk = new milk(rice);//把被装饰者对象放到装饰者的构造方法上去

        System.out.println("rice:"+rice.cost());
        System.out.println("milk+rice:"+milk.cost());

        milk milk1 = new milk(milk);

        System.out.println("milk*2+rice"+milk1.cost());



    }
 
}
```











#### 适配器模式

**适配器模式：就是把原本互不兼容的两样东西或者格式，转换成可以兼容**

**比如说：家用电压是220v，但是我们手机充电不需要这么大的电压，我们这时候需要电源适配器（充电头）,去适配这个电压，把220v适配成（比如：5v，11v等等）**

**再比如说：有一个方法只能接收int类型数据，但是我们接收到了一个String类型数据，int和String本身就是两个不相同的类型，所以说这个方法是接收不到String类型的，我们可以使用适配器，把String类型转换（适配）成int类型，这样这个方法就可以接收到这个String类型数据了。**



##### 类适配器

**公式：适配器类要继承src（原来的类），并实现dest（目标）接口**



**首先创建适配器类，并运用上面的公式：**

```java
public class classAdapter extends srcClass implements destInterface{

    /**
     * 类适配器是有公式的
     * 公式：类适配器要继承源类，实现于目标接口
     * public class classAdapter extends srcClass implements destInterface
     *
     * srcClass：在这里是指家用电压220V
     * destInterface ： 在这里是指我们想适配成多少v的电压，比如手机常用的电压 5v
     */

    @Override
    public int vivoBattery_5V() {
        int src = super.getSrc();
        int dest=src/44;
        return dest;
    }




}
```



**源（src）类：**

```java
public class srcClass {

    private final int v=220; //家用电压


    //源类生成220v
    public int getSrc(){
        return v;
    }

}
```



**目标接口：**

```java
public interface destInterface {


    int vivoBattery_5V(); //vivo手机想要多少伏的电压



}
```

**测试：**

```java
public class Test {


    public static void main(String[] args) {

        classAdapter classAdapter = new classAdapter();

        int res = classAdapter.vivoBattery_5V();
        System.out.println("当前电压："+res+"v");


    }

}
```









##### 对象适配器



**为什么会有对象适配器：我们的对象适配器都是在类适配器上优化的，类适配器的缺点是extends（继承），因为Java是单继承的，而我们的对象适配器不过就是把这个extends srcClass 优化成一个对象，通过构造器进行传入，然后通过这个对象进行调用srcClass的方法，而类适配器就是通过super关键字去调用方法。。**



```java
public class adapter  implements destInterface {

    private srcClass srcClass;  //把之前的类适配器的extends 源类，改成一个对象进行调用

    public adapter(srcClass srcClass){ //再通过构造器传入srcClass对象
        this.srcClass=srcClass;
    }

    @Override
    public int get10V() {
        int src = srcClass.getV();
        int dest=src/22;
        return dest;
    }
}
```



```java
public interface destInterface {


    int get10V();



}
```



```java
public class srcClass {

    private final int v=220;

    public int getV(){

        return v;
    }
  
}

```



```java
public class Test {


    public static void main(String[] args) {

        adapter adapter = new adapter(new srcClass());
        int v = adapter.get10V();
        System.out.println(v);

    }

}

```

 

##### 接口适配器

**为何有接口适配器：我们上面说的两种适配器方式都有一个缺点：就是我们上面两种适配器都需要实现目标接口，当目标接口有多个适配方法时，我们都要对它进行实现，而我们不需要全部实现，只需要实现其中一个即可，我们的接口适配器就是为了解决这个问题的。**



**没有给接口方法默认实现时：**

```java
public interface dest {


    int adapter_5v();

    int adapter_10v();

    int adapter_20v();





}
```

```java
public class adapter implements dest {


    @Override
    public int adapter_5v() {
        return 0;
    }

    @Override
    public int adapter_10v() {
        return 0;
    }

    @Override
    public int adapter_20v() {
        return 0;
    }
}
```

**就需要全部进行实现，很复杂**



```java
public class srcClass {

    private final int v=220; //家用电压


    //源类生成220v
    public int getSrc(){
        return v;
    }
 

}
```



```java
public interface dest {

    /**
     * 当接口适配的方法很多，但是我们不需要全部的进行实现，我们可以随便给他一个默认值，用default修饰
     * 当我们需要具体实现某个方法时，只需要重写对应的方法即可
     */

    default int adapter_5v(){

        return 0;
    }

     default int adapter_10v(){

        return 0;
     }

    default int adapter_20v(){
        return 0;
    }


}
```



```java
public class adapter implements dest {

    private srcClass srcClass;

    public adapter(srcClass srcClass){
        this.srcClass=srcClass;
    }

    @Override
    public int adapter_10v() {
        int src = srcClass.getSrc();
        int res=src/22;
        return res;
    }



}
```



```java
class main{


    public static void main(String[] args) {

        adapter adapter = new adapter(new srcClass());
        int i = adapter.adapter_10v();
        System.out.println(i);


    }

}
```













#### 享元模式

**享元模式：“享”是共享 ，“元”是对象 ，享元模式也就是一种利用共享技术对实例对象进行共享，提高实例对象的复用性，和节省资源的开销，减少对象的创建，应用在大量的“”相似“”对象，但是这些对象只有一些不同，我们可以用这个模式。**



**享元模式如何在定制相似的对象（对对象的数据进行修改）：我们可以通过享元模式的简单工厂从HashMap/HashTable中get到，然后再用set方法对它进行定制。**



**何为相似的对象： 说白了也就是同一个类创建出来的对象就是相似的对象**





**案例：用户的网站界面属性（网站类型（type）、颜色（color），动态消息（message））**



```java
public abstract class website {

    /**
     * 总的网站抽象类
     */

    public abstract void setColor(String color);

    public abstract void setMessages(List<String> messages);

    public abstract void setRandomCode(String randomCode);


}
```



```java
public class sina extends website {

    /**
     * 新浪网站
     */
    private final String type="新浪";
    private String color;
    private List<String> messages;
    private String randomCode; //随机码

    public sina() {
    }



    public String getType() {
        return type;
    }

    public String getColor() {
        return color;
    }


    public List<String> getMessages() {
        return messages;
    }


    public String getRandomCode() {
        return randomCode;
    }


    @Override
    public String toString() {
        return "sina{" +
                "type='" + type + '\'' +
                ", color='" + color + '\'' +
                ", messages=" + messages +
                ", randomCode='" + randomCode + '\'' +
                '}';
    }


    @Override
    public void setColor(String color) {
        this.color=color;
    }

    @Override
    public void setMessages(List<String> messages) {
        this.messages=messages;
    }

    @Override
    public void setRandomCode(String randomCode) {
        this.randomCode=randomCode;
    }
}

```



```java
public class zhihu extends website {

    /**
     * 知乎
     */
    private final String type="知乎";
    private String color;
    private List<String> messages;
    private String randomCode;

    public zhihu() {
    }



    public String getType() {
        return type;
    }

    public String getColor() {
        return color;
    }



    public List<String> getMessages() {
        return messages;
    }



    public String getRandomCode() {
        return randomCode;
    }



    @Override
    public String toString() {
        return "zhihu{" +
                "type='" + type + '\'' +
                ", color='" + color + '\'' +
                ", messages=" + messages +
                ", randomCode='" + randomCode + '\'' +
                '}';
    }
    @Override
    public void setColor(String color) {
        this.color=color;
    }

    @Override
    public void setMessages(List<String> messages) {
        this.messages=messages;
    }

    @Override
    public void setRandomCode(String randomCode) {
        this.randomCode=randomCode;
    }
}

```



**享元工厂(相当于简单工厂的升级版)：**

**这边存在线程安全问题，因为getWebsiteByFactory方法不是原子操作，所以我们多线程环境下一定要加锁**

```java
public class simpleFactory {

    /**
     * 享元模式是需要简单工厂模式的
     */
    private Map<String,website> map=new HashMap<>(); //享元模式精髓
    private String[] colors={"red","blue","yellow","green","orange"};

    public  website getWebsiteByFactory(String type){
        if(type==null||type.equals("")) {
            return null;
        }else {
            website website = map.get(type);
            if(website==null){
                //用简单工厂模式来书写
                if(type.equals("新浪")){
                    sina sina = new sina();
                    Random random = new Random();
                    int index = random.nextInt(5);
                    sina.setColor(colors[index]);
                    int i = random.nextInt(1000);
                    ArrayList<String> list = new ArrayList<>();
                    list.add("hello"+i);
                    sina.setMessages(list);
                    sina.setRandomCode(String.valueOf(random.nextInt(9999)));
                    map.put(type,sina);
                    System.out.println("创建新浪网站对象成功");
                    return sina;

                }else if(type.equals("知乎")){
                    zhihu zhihu = new zhihu();
                    Random random = new Random();
                    zhihu.setColor(colors[random.nextInt(5)]);
                    ArrayList<String> list = new ArrayList<>();
                    list.add("hello"+random.nextInt(1000));
                    zhihu.setMessages(list);
                    zhihu.setRandomCode(String.valueOf(random.nextInt(9999)));
                    map.put(type,zhihu);
                    System.out.println("创建知乎网站成功");
                    return zhihu;
                }else {
                    return null;
                }

            }else {
                System.out.println("在享元模式的简单工厂中已经存在这个网站，所以直接获取到，没有重新创建对象");
                return website;
            }
 
        }
 
    }

}
```



**测试：**

```java
public class main {


    private static final String[] websites={"新浪","知乎"};

    public static void main(String[] args) {

        simpleFactory simpleFactory = new simpleFactory(); //创建享元模式的简单工厂

        website s1 = simpleFactory.getWebsiteByFactory("新浪");
        System.out.println(s1); //会创建对象
        /**
         * 享元模式重要作用之一：定制对象
         */
        s1.setColor("利用享元模式定制对象");
        website s2 = simpleFactory.getWebsiteByFactory("新浪");
        System.out.println(s2); //不会创建对象，而是从map里面拿
        




        Random random = new Random();


//        for (int i = 0; i < 20; i++) {
//            new Thread(()->{
//                website website = simpleFactory.getWebsiteByFactory(websites[random.nextInt(2)]);
//                System.out.println(website);
//
//            }).start();
//        }




    }

}
```



**输出结果：**

```tex
创建新浪网站对象成功
sina{type='新浪', color='red', messages=[hello713], randomCode='2522'}
在享元模式的简单工厂中已经存在这个网站，所以直接获取到，没有重新创建对象
sina{type='新浪', color='利用享元模式定制对象', messages=[hello713], randomCode='2522'}
```







##### 多线程环境下的问题

**只需在getWebsiteByFactory方法上加锁即可**

```java
public class simpleFactory {

    /**
     * 享元模式是需要简单工厂模式的
     */
    private Map<String,website> map=new HashMap<>(); //享元模式精髓
    private String[] colors={"red","blue","yellow","green","orange"};

    public synchronized website getWebsiteByFactory(String type){
```

 

 





#### 组合模式（树型结构）



**组合模式的项目应用：实现多级菜单 、文件夹的管理**



**组合模式我们把它看成一种“树”型结构，固然后面我们在打印这些内容的时候是需要用递归的**



**因为组合模式是按照树形结构来组合的，所以也有树的特点（比如：叶子节点）**

 

**案例：利用组合模式来实现三级菜单。**



**定义组合模式的菜单组件接口，并定义添加、删除、展示结点方法：**

```java
public interface component {

    /**
     * 组件接口：所有菜单都要实现这个component（组件）接口
     * 组合模式所需方法：添加结点、删除结点、打印菜单
     */

    //需要给这个添加结点一个默认实现，因为我们的叶子结点不需要添加结点和删除结点这两个方法
    default void addNode(component component){
        throw new UnsupportedOperationException();//默认抛出不支持操作异常
    }

    default void deleteNode(component component){
        throw new UnsupportedOperationException();
    }

    //打印===都需要实现
    void showMenu();

 
}
```



**菜单组件实现类（非叶子结点==第一、二级菜单）：**

**一级菜单：**

```java
public class firstMenu implements component {
    /**
     * 一级菜单
     */
    private String name; //菜单名称
    private List<component> components;

    public firstMenu(String name){
        this.name=name;
        this.components=new ArrayList<>();
    }

    @Override
    public void addNode(component component) {
        components.add(component);
    }

    @Override
    public void deleteNode(component component) {
        components.remove(component);
    }

    @Override
    public void showMenu() {
        System.out.println("---"+this.getName()); //打印当前调用者的名称
        if(components.size()>0){
            for (component component : components) {
                component.showMenu(); //递归的调用这个showMenu方法
            }
        }

    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<component> getComponents() {
        return components;
    }

    public void setComponents(List<component> components) {
        this.components = components;
    }
}

```



**二级菜单：**

```java
public class secondMenu implements component {
    /**
     * 二级菜单
     */
    private String name;
    private List<component> components;

    public secondMenu(String name){
        this.name=name;
        this.components=new ArrayList<>();
    }



    @Override
    public void addNode(component component) {
        components.add(component);
    }

    @Override
    public void deleteNode(component component) {
        components.remove(component);
    }

    @Override
    public void showMenu() {
        System.out.println("------"+this.getName());

        if(components.size()>0){
            for (component component : components) {
                component.showMenu();;
            }
        }


    }
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<component> getComponents() {
        return components;
    }

    public void setComponents(List<component> components) {
        this.components = components;
    }
}

```



**三级菜单：（叶子节点）,没有添加、删除结点方法**

```java
public class lastMenu implements component {

    private String name;

    /**
     * 最后一个菜单（三级菜单）：不能添加和删除，因为它是树的叶子结点
     */

    public lastMenu(String name){
        this.name=name;
    }


    @Override
    public void showMenu() {
        System.out.println("---------"+this.name);
    }
}

```



**组合模式客户端代码：**

```java
public class client {

    /**
     * 组合模式客户端
     * @param args
     */

    public static void main(String[] args) {

        //创建一级菜单
        component f1=new firstMenu("用户列表");
        //创建二级菜单
        component s1=new secondMenu("用户信息");
        component s2=new secondMenu("用户权限");
        component s3=new secondMenu("用户好友");
        //创建三级菜单
        component l1=new lastMenu("信息修改");
        component l2=new lastMenu("信息删除");

        component l3=new lastMenu("修改权限");

        component l4=new lastMenu("添加好友");
        component l5=new lastMenu("删除好友");
        component l6=new lastMenu("修改好友信息");


        //进行组合
        f1.addNode(s1);
        f1.addNode(s2);
        f1.addNode(s3);

        s1.addNode(l1);
        s1.addNode(l2);

        s2.addNode(l3);

        s3.addNode(l4);
        s3.addNode(l5);
        s3.addNode(l6);

        //展示菜单
        f1.showMenu();

    }


}

```



**输出结果：**

```tex
---用户列表
------用户信息
---------信息修改
---------信息删除
------用户权限
---------修改权限
------用户好友
---------添加好友
---------删除好友
---------修改好友信息
```









#### 外观模式

**为什么会有外观模式？外观模式能解决什么？其实就是把其他对象调用方法封装到外观类上，我们只需要通过调用外观类的方法就可以调用其他类的方法**



**总结：外观模式（门面模式）说白了就是把多个不同对象（类）的方法归纳到一个门面类（外观类）中，这样省去频繁创建对象，核心就是门面类**

**使用两种方法实现案例“买水果问题”。。**

##### **传统方式（买水果问题）：**

**苹果实体类：**

```java
public class apple {

    /**
     * 苹果实体类
     */

    private String id;
    private String name;

    public apple(String name){
        this.name=name;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "apple{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}

```



```java
public class banana {

    /**
     * 香蕉实体类
     */

    private String id;
    private String name;

    public banana(String name){
        this.name=name;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "banana{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}

```



```java
public class orange {
    /**
     * 橙子实体类
     */

    private String id;
    private String name;

    public orange(String name){
        this.name=name;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "orange{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}

```

**创建各自的商店：**

```java
public class appleStore {

    /**
     * 苹果专卖店
     */


    //买苹果
    public apple buyApple(){
        apple apple = new apple("苹果");
        apple.setId(UUID.randomUUID().toString().replaceAll("-",""));
        return apple;
    }

}

```



```java
public class bananaStore {

    //香蕉实体店

    public banana buyBanana(){

        banana banana = new banana("香蕉");
        banana.setId(UUID.randomUUID().toString().replaceAll("-",""));
        return banana;

    }
}
```



```java
public class orangeStore {

    //橙子实体店

    public orange buyOrange(){
        orange orange=new orange("橙子");
        orange.setId(UUID.randomUUID().toString().replaceAll("-",""));
        return orange;
    }


}

```

**顾客买水果：**

```java
public class client {


    public static void main(String[] args) {

        //买苹果
        //1.创建苹果店对象
        appleStore appleStore = new appleStore();
        //2.购买
        apple apple = appleStore.buyApple();
        //买香蕉
        bananaStore bananaStore = new bananaStore();
        banana banana = bananaStore.buyBanana();
        //买橙子
        orangeStore orangeStore = new orangeStore();
        orange orange = orangeStore.buyOrange();

        System.out.println(apple);
        System.out.println(banana);
        System.out.println(orange);
    }


}
```

**====================**

**总结：可以看出来使用传统方法买水果比较复杂，顾客需要创建各自的商店，再去购买。。。**



##### 外观模式（买水果问题）：

**上面实体类不变，只需要添加一个门面类（外观类）即可**



**核心（外观类）：**

```java
public class myFacade {

    /**
     * 创建门面类（外观类）
     */
    //这几个变量也可以写成单例，这里暂且不写
    private appleStore appleStore;
    private bananaStore bananaStore;
    private orangeStore orangeStore;

    public myFacade(){
        //初始化对象
        this.appleStore=new appleStore();
        this.bananaStore=new bananaStore();
        this.orangeStore=new orangeStore();
    }

    /**
     * 精髓。。。
     * 使用外观模式的外观类来归纳方法，把多个类的方法归纳到一个外观类中
     * @return
     */
    //创建门面方法（外观方法），购买水果
    //买苹果
    public apple buyAppleByfacade(){
        apple apple = appleStore.buyApple();
        apple.setId(UUID.randomUUID().toString().replaceAll("-",""));
        return apple;
    }
    //买香蕉
    public banana buyBananaByfacade(){
        banana banana = bananaStore.buyBanana();
        banana.setId(UUID.randomUUID().toString().replaceAll("-",""));
        return banana;
    }
    //买橙子
    public orange buyOrangeByfacade(){
        orange orange = orangeStore.buyOrange();
        orange.setId(UUID.randomUUID().toString().replaceAll("-",""));
        return orange;
    }
 
}
```



```java
public class client {


    public static void main(String[] args) {

        //创建已经归纳好方法的外观类
        myFacade myFacade = new myFacade();
        //购买水果
        apple apple = myFacade.buyAppleByfacade();
        banana banana = myFacade.buyBananaByfacade();
        orange orange = myFacade.buyOrangeByfacade();

        System.out.println(apple);
        System.out.println(banana);
        System.out.println(orange);


    }
}
```



**这时候我们不需要创建商店了，只需要创建外观类调用买水果方法即可。**



**外观模式优点：假如水果有1000种，那么我们就要创建1000个对象，而使用外观模式，只需要创建一个外观类的对象即可，只需要1个对象。这样对比起来就相当明显了。。。。。。。。。。。。。。。。。。**



#### 桥接模式

 **使用桥接类和需要被桥接的接口进行桥接**

**案例：假设我们需要一个接口，用来连接mysql或者Oracle**



```java
public interface connection {


    public void connectDatasource();


}
```



```java
public class connection_mysql implements connection {

    /*
     连接mysql数据源
     */
    @Override
    public void connectDatasource() {
        System.out.println("mysql数据源连接成功！！！");
    }
}
```



```java
public class connection_oracle implements connection {

    /*
    连接Oracle数据源
     */
    @Override
    public void connectDatasource() {
        System.out.println("oracle数据源连接成功！！！");
    }
}
```



**传统方式测试：**

```java
public class client_chuantong {


    public static void main(String[] args) {

        //传统方法调用接口方法
        connection connection=new connection_mysql();
        connection.connectDatasource();

        connection connection1=new connection_oracle();

        connection.connectDatasource();


    }

}
```



**桥的抽象类：**

```java
public abstract class bridge {
    /**
     * 桥的抽象类
     */
    
    public abstract void connection();


}
```



**桥的实现类**

```java
public class bridgeImpl extends bridge {

    /**
     * 核心
     * 使用了桥接模式之后，我们只需要创建桥接
     */
    private connection connection;

    //给对象注入
    public bridgeImpl(connection connection){
        this.connection=connection;
    }

    //桥接方法（桥接接口方法）
    @Override
    public void connection() {

        connection.connectDatasource();

    }
}
```



**桥接测试：**

```java
public class client_bridge {


    public static void main(String[] args) {
        //使用桥接实现类调用被桥接的接口方法
        bridge bridge = new bridgeImpl(new connection_oracle());
        bridge.connection();
    }

}
```



 



### 行为型



#### 观察者模式



**既然有观察者，那么也就有被观察者。一个被观察者被多个观察者观察。比如说天气预报，天气预报就是被观察者，订阅了天气预报的人就是观察者，天气预报需要通知观察者。**



**被观察者接口：**

```java
public interface observerable {
    /**
     * 被观察者
     */

    //添加观察者
    public void addObserver(observer observer);
    //移除观察者
    public void removeObserver(observer observer);
    //通知所有观察者
    public void notifyAllObserver();



}
```



**观察者接口：**

```java
public interface observer {

    /**
     * 观察者
     */

    //接收通知的方法
    public void getTianqi(tianqi tianqi);

}

```



**被观察者（在这是天气预报）：**

```java
public class tianqi implements observerable {

    /**
     * 被观察者 ：天气预报
     */
    private String date;//日期
    private String wendu;//摄氏度
    private List<observer> observers; //管理所有观察者，以便通知观察者们（其实也就是调用观察者的接收通知方法）

    public tianqi(){
        this.observers=new ArrayList<>();
        this.date="2021/5/9";
        this.wendu="37°C";
    }

    public tianqi(String date, String wendu) {
        this.date=date;
        this.wendu=wendu;
    }

    public String getDate() {
        return date;
    }

    public void setDate(String date) {
        this.date = date;
    }

    public String getWendu() {
        return wendu;
    }

    public void setWendu(String wendu) {
        this.wendu = wendu;
    }

    @Override
    public void addObserver(observer observer) {
        observers.add(observer);
    }

    @Override
    public void removeObserver(observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyAllObserver() {
        if(observers.size()>0){
            for (int i = 0; i < observers.size(); i++) {
                observers.get(i).getTianqi(new tianqi(this.date,this.wendu));
            }
        }else{
            System.out.println("没有人订阅过天气预报。。。");
        }
    }


    @Override
    public String toString() {
        return "天气信息{" +
                "date='" + date + '\'' +
                ", wendu='" + wendu + '\'' +
                '}';
    }
}
```

**观察者：**

```java
public class person implements observer {

    /**
     * 人：观察者
     */
    private String name; //观察者名字

    public person(String name){
        this.name=name;
    }

    @Override
    public void getTianqi(tianqi tianqi) {
        System.out.println("当前用户名为："+name+"，"+tianqi);


    }
}

```



**客户端（测试）：**

```java
public class client {


    public static void main(String[] args) {

        tianqi tianqi = new tianqi();

        tianqi.addObserver(new person("小明"));
        tianqi.addObserver(new person("小华"));

        tianqi.notifyAllObserver();

        System.out.println("=======修改后");

        tianqi.addObserver(new person("小刘"));
        tianqi.setDate("2021/5/10");
        tianqi.setWendu("15°C");
        tianqi.notifyAllObserver();

    }

}

```

 

#### 备忘录模式

**备忘录模式：可以用来备份数据和恢复**

**备忘录需要有三个类：需要备份的类、originator（生成需要备份的类的对象），存储数据和恢复数据的类**


 

#### 责任链模式






 

#### 策略模式





#### 模板模式





## RabbitMQ

**在目前主流的消息队列中有（ActiveMQ,RocketMQ,RabbitMQ,kafka）**

**RabbitMQ在上面的各种消息队列中对于消息的保护是十分到位的（不会丢失消息），相对于kafka，虽然kafka性能十分强悍，在大数据中处理海量数据游刃有余，但是kafka容易丢失消息，而RabbitMQ虽然性能不及kafka，但是也不会很差，对于消息要求完整性很高的系统中用RabbitMQ十分好。**

### 安装RabbitMQ环境

**总教程：https://www.cnblogs.com/saryli/p/9729591.html**

**1.安装erlang**

**（1.）下载erlang**

**官网地址：https://www.erlang.org/**

**下载教程：https://www.cnblogs.com/minily/p/7398445.html**

**（2.）配置erlang环境**

**配置教程：https://blog.csdn.net/g6256613/article/details/80191402**

**需要配置环境变量**

![image-20210304210138086](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210304210138086.png)


![image-20210304210211142](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210304210211142.png)

 

**（3.）检查是否安装成功**

**打开cmd，输入erl，有输出说明成功**



**（4.）下载rabbitMQ**

**下载地址：https://www.cnblogs.com/saryli/p/9729591.html**



**。。。。。。。。。。。。省略，在总教程都有。**



**（5.）最后访问http://localhost:15672，如果访问成功，说明rabbitMQ安装成功**



### RabbitMQ的5种模型（重点**）



#### 导入依赖

```xml
		 <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.7.3</version>
        </dependency>
```

#### 基本消息模型(hello world)

**生产者**

```java
public class provider {
    /**
     * 最基本的消息队列模型
     *
     * 消息生产者
     * @param args
     */

    public static void main(String[] args) throws IOException, TimeoutException {
        //1.先new一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //定义指定rabbitmq配置的工厂
        factory.setUsername("ems");
        factory.setPassword("123456");
        factory.setVirtualHost("/ems"); //虚拟主机
        factory.setHost("127.0.0.1");  //rabbitMQ的主机名（ip）
        //2.通过连接工厂创建一个connection
        Connection connection = factory.newConnection();
        //3.通过connection对象create一个channel通道，以后我们的操作就是channel
        Channel channel = connection.createChannel();

        //4.声明队列,如果没有这个队列则会自动生成
        /**
         *   queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments)
         *   参数1:队列名字
         *   参数2：队列是否持久化
         *   参数3：是否排斥（也就是一个队列是否只能由一个消费者消费）
         *   参数4：自动删除，当所有消费者消费完之后是否把队列删除
         *   参数5：额外参数
         */
        channel.queueDeclare("hello",true,false,false,null);

        //5.发布消息
        /**
         * 参数1：交换机名称，空字符串代表使用默认交换机。。。。
         * 参数2：路由键（在没有指定交换机的情况下（不包括空字符串），路由键是发送消息队列的名字
         * 参数3：额外参数===通常用MessageProperties.PERSISTENT_TEXT_PLAIN，意思是发送的消息在没有消费完也能持久化
         * *****参数4（最重要）：发送的消息内容（要转换成byte类型）
         */
        channel.basicPublish("","hello", MessageProperties.PERSISTENT_TEXT_PLAIN,"第一个RabbitMQ程序！！！".getBytes());

        channel.close();
        connection.close();

    }
}


```

**消费者**

```java
public class comsumer {
    /**
     * 消息消费者
     */

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        //这里的配置参数一定要和生产者一模一样，不然会报错
        channel.queueDeclare("hello",true,false,false,null);

        //进行消费
        /**
         * 参数1：队列名字
         * 参数2：是否自动确认消息
         * 参数3：通常用DefaultConsumer匿名内部类，实现handleDelivery接收消息
         */
        channel.basicConsume("hello",true,new DefaultConsumer(channel){
            /**
             *参数3：接收的消息
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("=======消费者取出消息===>"+new String(body));

            }
        });


        /**
         * 消费者端最好不要关闭channel和connection，不然可能读取不到消息
         */
//        channel.close();
//        connection.close();

    }

}


```

 

#### work queue模型

**为什么会引入这么一个消息队列模型？？？？**

**我们可以想象一下，如果按照第一个模型，点对点的，生产者发消息经过消息队列再到消费者，此时消费者只有1个，如果我们生产者发送60条消息，假设每条消息要1秒钟才能执行完，那么hello world模型就要60秒才能消费完所有消息，如果我们用workqueue模型呢，我们假如再引入一个消费者，也就是1个生产者发送60条信息到2个消费者，默认负载均衡，每个队列处理30条，而且还是异步处理，那么我们只需要30秒就处理好了，效率大大的提高**





##### 未实现能者多劳机制



```java
public class provider {
    /**
     * 生产者
     * ====workQueue模型
     */
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        connectionFactory.setVirtualHost("/ems");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        //生产者声明了队列，消费者也都要声明
        channel.queueDeclare("workqueue",true,false,false,null);
        //basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body)
        for (int i = 0; i < 10; i++) {
            channel.basicPublish("","workqueue",null,("hello=="+i+"").getBytes());
        }

        channel.close();
        connection.close();


    }

}

```

```java
public class comsumer1 {
    /**
     * 消费者1
     * @param args
     */

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        connectionFactory.setHost("127.0.0.1");

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();
        //queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments)
        channel.queueDeclare("workqueue",true,false,false,null);
        //basicConsume(String queue, boolean autoAck, Consumer callback)
        channel.basicConsume("workqueue",true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("===comsumer1===>"+new String(body));
            }
        });

    }


}

```

```java
public class comsumer2 {
    /**
     * 消费者2
     */
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        channel.queueDeclare("workqueue",true,false,false,null);

        channel.basicConsume("workqueue",true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("===comsumer2===>"+new String(body));
            }
        });


    }

}

```



**输出结果：（默认是类似负载均衡的轮询算法）**

```
===comsumer1===>hello==0
===comsumer1===>hello==2
===comsumer1===>hello==4
===comsumer1===>hello==6
===comsumer1===>hello==8
```











##### 实现了能者多劳机制

**要实现能者多劳，只需要在消费者修改几处代码即可**

**1. channel.basicQos(1);**

**2. channel.basicConsume("workqueue",false,new DefaultConsumer(channel)**

**3. channel.basicAck(envelope.getDeliveryTag(),false); //手动确认**

```java
public class comsumer1 {
    /**
     * 消费者1 能者多劳
     * @param args
     */

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        connectionFactory.setHost("127.0.0.1");

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        //每次收到一条消息
        channel.basicQos(1);

        //queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments)
        channel.queueDeclare("workqueue",true,false,false,null);
        //basicConsume(String queue, boolean autoAck, Consumer callback)
        channel.basicConsume("workqueue",false,new DefaultConsumer(channel){ //第二个参数修改为false，取消自动avk
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("===comsumer1===>"+new String(body));
                channel.basicAck(envelope.getDeliveryTag(),false); //手动确认
            }
        });

    }


}

```

```java
public class comsumer2 {
    /**
     * 消费者2 能者多劳
     */
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        //每次只能收一条消息
        channel.basicQos(1);

        channel.queueDeclare("workqueue",true,false,false,null);

        channel.basicConsume("workqueue",false,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("===comsumer2===>"+new String(body));
                channel.basicAck(envelope.getDeliveryTag(),false); //手动确认消息
            }
        });


    }

}

```



**输出结果:**

```
===comsumer1===>hello==0
===comsumer1===>hello==6
===comsumer1===>hello==8
```







#### fanout模型（广播模型）性能最好

**特点：凡是和这个fanout交换机绑定的临时队列，都能收到消息**

```java
public class provider {
    /**
     * fanout模型（广播模型）
     *
     */
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        connectionFactory.setHost("127.0.0.1");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        //生产者声明交换机==>exchangeDeclare(String exchange, String type, boolean durable, boolean autoDelete, Map<String, Object> arguments)
        /**
         * exchangeDeclare:
         * 参数一：交换机名字
         * 参数二：交换机类型：
         * 有这几种类型：""   , "fanout" , "direct" ,  "topic"
         * 参数三：交换机是否持久化。（重启rabbitmq服务如果交换机没有删除就是持久化）
         * 参数四：是否自动删除
         * 参数五：额外参数
         */
        channel.exchangeDeclare("hello_exchange_fanout","fanout",true,false,null);

        //这里不用声明消息队列，只需要声明交换机即可，消费者需要声明消息队列（临时队列）
        //basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body)
        channel.basicPublish("hello_exchange_fanout","",null,"exchange_fanout".getBytes());
        channel.close();
        connection.close();

    }


}

```

```java
public class comsumer1 {
    /**
     * fanout模型（广播模型）
     *
     */
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        //声明交换机
        /**
         * exchangeDeclare:
         * 参数一：交换机名字
         * 参数二：交换机类型：
         * 有这几种类型：""   , "fanout" , "direct" ,  "topic"
         * 参数三：交换机是否持久化。（重启rabbitmq服务如果交换机没有删除就是持久化）
         * 参数四：是否自动删除
         * 参数五：额外参数
         */
        channel.exchangeDeclare("hello_exchange_fanout","fanout",true,false,null);

        //创建一个临时队列
        String queueName = channel.queueDeclare().getQueue();

        //把交换机和临时队列绑定在一起
        //queueBind(String queue, String exchange, String routingKey)
        channel.queueBind(queueName,"hello_exchange_fanout","");

        //然后就可以通信了
        //basicConsume(String queue, boolean autoAck, Map<String, Object> arguments, Consumer callback)
        channel.basicConsume(queueName,true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println(new String(body));
            }
        });




    }



}

```

```java
public class comsumer2 {

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare("hello_exchange_fanout","fanout",true,false,null);

        String queueName = channel.queueDeclare().getQueue();

        channel.queueBind(queueName,"hello_exchange_fanout","");

        channel.basicConsume(queueName,true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println(new String(body));
            }
        });


    }

}

```





























#### direct模型(直连)（默认） 

**特点：根据路由键直接匹配**

**fanout、direct、topic 交换机类型都是可以把同一条消息路由到多个消费者身上的。而hello world、work queue不行。work queue和hello world模型同一条消息只能路由到某一个消费者身上**

```java
public class provider {
    /**
     * direct模式（直连交换机）
     * @param args
     */
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        channel.exchangeDeclare("direct_exchange","direct",true,false,null);

        /**
         * 参数2：路由键，如果消费者有符合的则可以接收消息
         */
        channel.basicPublish("direct_exchange","user_log", MessageProperties.PERSISTENT_TEXT_PLAIN,

                "hello,direct".getBytes());

        channel.close();
        connection.close();



    }

}

```



```java
public class comsumer1 {


    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        channel.exchangeDeclare("direct_exchange","direct",true,false,null);

        String queueName = channel.queueDeclare().getQueue();

        //可以绑定多个路由,只要符合一个就可以接收到消息
        channel.queueBind(queueName,"direct_exchange","user_log");
//        channel.queueBind(queueName,"direct_exchange","user_money");

        channel.basicConsume(queueName,true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("comsumer1===>"+new String(body));

            }
        });


    }

}

```



```java
public class comsumer2 {

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        channel.exchangeDeclare("direct_exchange","direct",true,false,null);

        String queueName = channel.queueDeclare().getQueue();

        //可以绑定多个路由,只要符合一个就可以接收到消息
//        channel.queueBind(queueName,"direct_exchange","user_log");
        channel.queueBind(queueName,"direct_exchange","user_money");

        channel.basicConsume(queueName,true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("comsumer2===>"+new String(body));

            }
        });
    }
}

```

 

#### topic模型(通配符)

**特点：通配符(#号和*号)，也可以不使用通配符。**

```java
public class provider {
    /**
     * topic模式
     * topic和direct相比，基本差不多，只不过topic可以使用通配符进行匹配
     * 在topic模式下，生产者发送的路由键是user.log.test，消费者可以用user.#或者#.log.test或者*.*.test 。。。等等来匹配
     * #:代表一个或多个单词的占位符
     * *:代表一个单词的占位符，如上面，user.*是匹配不了user.log.test的。。。。。
     * 交换机性能：fanout>direct>topic
     */

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare("topic_exchange","topic",true,false,null);
        for (int i = 0; i < 10; i++) {
            String msg="topic_hello_"+i;
            channel.basicPublish("topic_exchange","log.order.money", MessageProperties.PERSISTENT_TEXT_PLAIN,msg.getBytes());

        }

        channel.close();
        connection.close();

    }
}

```

```java
public class consumer1 {


    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        String queueName = channel.queueDeclare().getQueue();

        channel.exchangeDeclare("topic_exchange","topic",true,false,null);

        channel.queueBind(queueName,"topic_exchange","log.order.money");

        channel.basicConsume(queueName,true,new DefaultConsumer(channel){

            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumer1==>"+new String(body));
            }
        });


    }


}

```

```java
public class consumer2 {


    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setVirtualHost("/ems");
        connectionFactory.setHost("127.0.0.1");
        connectionFactory.setUsername("ems");
        connectionFactory.setPassword("123456");

        Connection connection = connectionFactory.newConnection();

        Channel channel = connection.createChannel();

        String queue = channel.queueDeclare().getQueue();

        channel.exchangeDeclare("topic_exchange","topic",true,false,null);

        /**
         * log.#===>#代表后面可以有一个或多个。
         * log,* ==>代表后面只能有一个，也就是类似log.xx 才能匹配上
         */
        channel.queueBind(queue,"topic_exchange","log.#");

        channel.basicConsume(queue,true,new DefaultConsumer(channel){

            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumer2====>"+new String(body));
            }
        });

    }


}

```



## SpringBoot+RabbitMQ

### 导入启动器

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
            <version>2.3.9.RELEASE</version>
        </dependency>
```

### application.yml

```yaml
spring:
  rabbitmq:
    username: ems
    password: 123456
    virtual-host: /ems
    host: localhost

```

### 自定义RabbitTemplate

**SpringBoot默认使用CachingConnectionFactory连接工厂**

```java
@Configuration
public class rabbitTemplateConfig {

    //注入SpringBoot默认的CachingConnectonFactory
    @Bean
    public RabbitTemplate rabbitTemplate(@Qualifier("rabbitConnectionFactory") CachingConnectionFactory cachingConnectionFactory){
        RabbitTemplate rabbitTemplate = new RabbitTemplate(cachingConnectionFactory);
        /**
         * 当mandatory标志位设置为true时
         * 如果exchange根据自身类型和消息routingKey无法找到一个合适的queue存储消息
         * 那么broker会调用basic.return方法将消息返还给生产者
         * 当mandatory设置为false时，出现上述情况broker会直接将消息丢弃
         */
        rabbitTemplate.setMandatory(true);
        //使用单独的发送连接，避免生产者由于各种原因阻塞而导致消费者同样阻塞
        rabbitTemplate.setUsePublisherConnection(true);

        return rabbitTemplate;
    }

}

```









### RabbitTemplate实现发送消息

#### 最简单的使用HelloWorld

**经过SpringBoot整合的RabbitMQ，发送消息只要一条语句**

**对比如下：**

**原生RabbitMQ：(11行）**

```java
 public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUsername("ems");
        factory.setPassword("123456");
        factory.setVirtualHost("/ems"); //虚拟主机
        factory.setHost("127.0.0.1");  //rabbitMQ的主机名（ip）
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare("hello",true,false,false,null);
        channel.basicPublish("","hello", MessageProperties.PERSISTENT_TEXT_PLAIN,"第一个RabbitMQ程序！！！".getBytes());
        channel.close();
        connection.close();
    }
```

**SpringBoot整合RabbitMQ：（1行）**

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class provider {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void send(){
        //一条代码即可发送消息
        /**
         * 参数1：交换机名称
         * 参数2：路由键
         * 参数3：消息内容（不需要转换成byte数组）
         */
        rabbitTemplate.convertAndSend("","boot_hello","boot_helloWorld");
    }
}

```

```java
@Component   //所有RabbitMQ的消费者都需要“”加上“”Spring的组件注解，RabbitMQ消费者监听方法不用运行都可以被自动生效。。。。
public class consumer {


    //RabbitMQ消费者监听方法
    @RabbitListener(queuesToDeclare = {@Queue(name = "boot_hello",durable = "true",exclusive = "false"
    ,autoDelete = "false")})
    public void receive(String msg){
        System.out.println(msg);
    }
}
```



#### workqueue

```java
@SpringBootTest
@RunWith(SpringRunner.class) //加载上下文
public class workqueueTest {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void send(){

//        System.out.println(rabbitTemplate);
        for (int i = 0; i < 10; i++) {
            rabbitTemplate.convertAndSend("","boot_work","workqueue===>"+i);
        }
    }
 

}

```



```java
@Component
public class consumer1 {

    @RabbitListener(queuesToDeclare = @Queue(name = "boot_work",durable = "true"))
    public void receive1(String msg1){
        System.out.println("consumer1===>"+msg1);
    }

}

@Component
class consumer2{

    @RabbitListener(queuesToDeclare = @Queue(name = "boot_work",durable = "true"))
    public void receive2(String msg2){
        System.out.println("consumer2===>"+msg2);
    }

}

```



#### fanout模式

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class fanoutTest {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void test(){
        rabbitTemplate.convertAndSend("boot_fanout","","hello");


    }
 
}

```



```java
@Component
public class consumer3 {

    @RabbitListener(bindings = {
            @QueueBinding(value = @Queue,exchange = @Exchange(value = "boot_fanout",type = "fanout"),key = "")
    })
    public void receive(String msg){
        System.out.println("consumer1===>"+msg);

    }


}
@Component
class consumer4{

    @RabbitListener(bindings = @QueueBinding(value = @Queue,exchange = @Exchange(value = "boot_fanout",type = "fanout"),key = ""))
    public void receive(String msg){
        System.out.println("consumer2===>"+msg);
    }
}

```

#### direct模式

```java

@SpringBootTest
@RunWith(SpringRunner.class)
public class directTest {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void test(){
        rabbitTemplate.convertAndSend("direct_boot","user.log","direct");

    }


}

```

````java
@Component
public class directConsumer1 {

    @RabbitListener(bindings = @QueueBinding(exchange = @Exchange(name = "direct_boot",type = "direct")
            ,value = @Queue,key = "user")
    )
    public void receive(String msg){
        System.out.println("consumer1===>"+msg);
    }



}
@Component
class directConsumer2{

    @RabbitListener(bindings = @QueueBinding(exchange = @Exchange(name = "direct_boot",type = "direct")
    ,value = @Queue,key = "user.log"
    ))
    public void receive(String msg){
        System.out.println("consumer2==>"+msg);

    }


}

````



#### topic模式

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class topicTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void test(){
        rabbitTemplate.convertAndSend("topic_boot","user.hello.log","hello");
    }

}
```



```java
@Component
public class topicConsumer1 {

    @RabbitListener(bindings = @QueueBinding(exchange = @Exchange(value = "topic_boot",type = "topic")
    ,value = @Queue,key = "user.#"
    ))
    public void receive(String msg){
        System.out.println("consumer1==>"+msg);
    }



}
@Component
class topicConsumer2{


    @RabbitListener(bindings = @QueueBinding(exchange = @Exchange(value = "topic_boot",type = "topic")
    ,value = @Queue,key = "user.*"
    ))
    public void receive(String msg){
        System.out.println("consumer2==>"+msg);
    }


}
```







## RabbitMQ高级特性

### 消息队列的过期时间ttl

**如果我们设置了消息队列的过期时间，假设我们设置了5000ms，5000ms过去了，如果这个队列还有未被消费的消息，那么这些消息将会被自动丢弃（无法找回）。。。。**

#### 队列里的消息的过期时间（有点坑）

**消费者的消息的过期时间**

**设置消息队列的argument为x-message-ttl 为xxx值，比如value="5000"，就是5秒过去了，消息队列未被消费的消息将会直接丢弃**



**坑：@argument注解设置参数一定要指定类型为Number子类，比如java.lang.Integer,不然会报错**

**比如：arguments = {@Argument(name = "x-message-ttl",value = "5000",type = "java.lang.Integer")}**





```java
spring:
  rabbitmq:
    username: ems
    password: 123456
    virtual-host: /ems
    host: localhost
    listener:
      direct:
        acknowledge-mode: manual #手动确认
      simple:
        acknowledge-mode: manual #手动确认
```



```java
@Test
    public void test1(){
        MessageProperties messageProperties = new MessageProperties();
        String msg = "hello_ttl";
        Message message = new Message(msg.getBytes(),messageProperties);
        rabbitTemplate.convertAndSend("ttl_queue","ttl_a",message);

    }
```



```java
/**
     *  ==小坑：
     * 使用RabbitListener实现队列的过期时间ttl必须要指定argument的“type”为Number类的子类，比如java.lang.Integer
     * =======切记，ttl和消息队列长度都要用Number的子类，使用默认的会报错======
     * 因为argument默认是java.lang.String类型，必须修改。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。
     * 。。。
     */

    //@Queue和@Exchange指定value就会使这个队列和交换机设置为不过期的，没有value就是暂时的
    @RabbitListener(bindings = @QueueBinding(value = @Queue(value = "ttl_temp",durable = "true"
    ,arguments = {@Argument(name = "x-message-ttl",value = "5000",type = "java.lang.Integer")}//一定要指定类型
    )
    ,exchange = @Exchange(value = "ttl_queue",type = "direct"),key = {"ttl_a"}
    ))
    public void receive(String msg,Message message,Channel channel){
        System.out.println("msg==="+msg);
        System.out.println("message==="+message);
        System.out.println("channel==="+channel);

//        try {
//            channel.basicAck(message.getMessageProperties().getDeliveryTag(),false); //手动确认
//        } catch (IOException e) {
//            e.printStackTrace();
//        }
    }
```









#### 指定消息的过期时间

**生产者消息的过期时间**

**核心代码：messageProperties.setExpiration("5000");**

```java
@Test
    public void test2(){
        MessageProperties messageProperties = new MessageProperties();

        messageProperties.setExpiration("5000"); //设置指定消息的过期时间

        String str="ttl_test2";
        Message message = new Message(str.getBytes(),messageProperties);
        rabbitTemplate.convertAndSend("","ttl_declare",message);

    }
```



```java
@RabbitListener(queuesToDeclare = @Queue(name = "ttl_declare"))
    public void  receive1(String msg,Message message,Channel channel) throws IOException {
        System.out.println(msg);
//        channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
    }
```

  

### 死信队列

**消息放进死信队列的条件：**

**1：消息过期了，如果有死信队列则放入死信队列，如果没有死信队列则直接丢弃无法找回。**

**2：某个消息队列长度已经达到最大值，此时在把消息发送到这个队列中，如果有死信队列则放入死信队列，没有则丢弃**

**3：消息被拒绝(basic.reject / basic.nack)**

**================创建死信队列步骤**

**1：创建一个普通队列**



```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class deadLetter {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void test(){


        Message message = new Message("deadLetter".getBytes(),new MessageProperties());
        rabbitTemplate.convertAndSend("","nomal_dead",message);



    }

}

```



```java
@Component
public class nomalQueue {


    /**
     * 这里我们只演示一种消息放入死信队列的情况（当消息过期后）
     * 在某个队列设置了x-dead-letter-exchange和x-dead-letter-routing-key后，如果出现丢弃消息就会
     * 通过x-dead-letter-exchange和x-dead-letter-routing-key找到指定的队列，这个队列就会默认是死信队列
     * 其实死信队列也是正常的队列。。。。配置全都一样
     */
    @RabbitListener(queuesToDeclare = @Queue(value = "nomal_dead",arguments = {
            @Argument(name = "x-message-ttl",value = "5000",type = "java.lang.Integer"),
            @Argument(name = "x-dead-letter-exchange",value = "deadletter_exchange1"),
            @Argument(name = "x-dead-letter-routing-key",value = "deadletter_key1")
    }
    ))
    public void receive(String msg, Message message, Channel channel){
        System.out.println("msg1="+msg);

    }

}


```

```java
@Component
public class deadLetterQueue {


    /**
     * 这里的交换机和路由key都要和配置的死信交换机、死信路由key一样。
     */
    @RabbitListener(bindings = @QueueBinding(value = @Queue("deadLetterQueue")
    ,exchange = @Exchange(value = "deadletter_exchange1",type = "direct")
    ,key = "deadletter_key1"
    ))
    public void receive_deadLetter(String msg){
        System.out.println(msg);
    }



}

```



 



### 固定长度的消息队列

**核心代码：arguments = @Argument(name = "x-max-length",value = "6",type = "java.lang.Integer")**

```java
  @Test
    public void test(){
            Message message = new Message(("max").getBytes(),new MessageProperties());
            rabbitTemplate.convertAndSend("","maxLength_queue",message);
    }
```



```java
 @RabbitListener(queuesToDeclare = @Queue(value = "maxLength_queue",durable = "true"
    ,arguments = @Argument(name = "x-max-length",value = "6",type = "java.lang.Integer")
    ))
    public void receive(String msg, Message message, Channel channel) throws IOException {
        System.out.println(msg);

    }
```







### 延时队列

**应用场景：下了订单过了30分钟未支付，然后就自动取消订单**

**rabbitmq本身是没有延迟队列的，我们可以通过ttl过期时间和死信队列（DLX）来实现**







## Elasticsearch7.6.1笔记

### elasticsearch概念

**elasticsearch是一个实时的分布式全文检索引擎，elasticsearch是由Lucene作为底层构建的，elasticsearch采用的不是一般的正排索引（类似于mysql索引），而是用倒排索引，好处是模糊搜索速度极快。。。**

**elasticsearch的操作都是使用JSON格式发送请求的**

### elasticsearch的底层索引

**我们知道mysql的like可以作为模糊搜索，但是速度是很慢的，因为mysql的like模糊搜索不走索引，因为底层是正排索引，所谓的正排索引，也就是利用完整的关键字去搜索。。。。而elasticsearch的倒排索引则就是利用不完整的关键字去搜索。原因是elasticsearch利用了“分词器”去对每个document分词（每个字段都建立了一个倒排索引，除了documentid），利用分出来的每个词去匹配各个document**



**比如：在索引名为hello下，有三个document**



**documentid       age       name      **

​      **1**                      **18**             **张三**

  	**2**          			**20**             **李四**	 

​	   **3**						**18**            **李四**



**此时建立倒排索引：**



**第一个倒排索引：**

**age** 

**18**        **1   ,    3**     

**20**         **2**



**第二个倒排索引：**

**name**

**张三**        **1**

**李四**       **2 ,  3**



### elasticsearch和关系型数据库（MySQL）

**我们暂且可以把es和mysql作出如下比较**

**mysql数据库（database） ==========   elasticsearch的索引（index）**

**mysql的表（table）==============   elasticsearch的type（类型）======后面会被废除**

**mysql的记录       ===========         elasticsearch的文档（document）**

**mysql的字段   =============      elasticsearch的字段（Field）**



### elasticsearch的一些注意点***

#### 跨域问题

**打开elasticsearch的config配置文件elasticsearch.yml**

**并在最下面添加如下：**

```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

#### 占用内存过多导致卡顿问题

**因为elasticsearch是一个非常耗资源的，从elasticsearch的配置jvm配置文件就可以看到，elasticsearch默认启动就需要分配给jvm1个g的内存。我们可以对它进行修改**

**打开elasticsearch的jvm配置文件jvm.options**

**找到：**

```text
-Xms1g    //最小内存
-Xms1g    //最大内存
```

**修改成如下即可：**

```text
-Xms256m
-Xms512m
```



#### elasticsearch和kibana版本问题

**如果在启动就报错，或者其他原因，我们要去看一看es和kibana的版本是否一致，比如es用的是7.6  ，那么kibana也要是7.6**







### ik分词器

#### ik分词器的使用

**ik分词器是一种中文分词器，但是比如有一些词（例如人名）它是不会分词的，所以我们可以对它进行扩展。**

**要使用ik分词器，就必须下载ik分词器插件，放到elasticsearch的插件目录中，并以ik为目录名**

**ik分词器一共有两种分词方式：ik_smart   ,   ik_max_word**

**ik_smart   :    最少切分（尽可能少切分单词）**

**ik_max_word :  最多切分  （尽可能多切分单词）  **



=============================



**ik_smart :**

```text
GET _analyze     //  _analyze 固定写法
{
  "text": ["中国共产党"],
  "analyzer": "ik_smart"
  
}
```

**结果：**

```json
{
  "tokens" : [
    {
      "token" : "中国共产党",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 0
    }
  ]
}

```



**ik_max_word : **

```text
GET _analyze
{
  "text": ["中国共产党"],
  "analyzer": "ik_max_word"
  
}
```

**结果：**

```json
{
  "tokens" : [
    {
      "token" : "中国共产党",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "中国",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "国共",
      "start_offset" : 1,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "共产党",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "共产",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "党",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 5
    }
  ]
}

```





#### ik分词器分词的扩展

```text
GET _analyze
{
  "text": ["我是游政杰，very nice"],
  "analyzer": "ik_max_word"
}
```

```json
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "游",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "CN_CHAR",
      "position" : 2
    },
    {
      "token" : "政",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "杰",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 4
    },
    {
      "token" : "very",
      "start_offset" : 6,
      "end_offset" : 10,
      "type" : "ENGLISH",
      "position" : 5
    },
    {
      "token" : "nice",
      "start_offset" : 11,
      "end_offset" : 15,
      "type" : "ENGLISH",
      "position" : 6
    }
  ]
}

```

**人名没有分正确。我们可以新建一个配置文件，去添加我们需要分的词**

**1.我们先去ik插件目录中找到IKAnalyzer.cfg.xml文件**

```xml
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict"></entry>     //如果有自己新建的dic扩展，就可以加到<entry>xxx.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

**2.创建my.dic，把自己需要分词的添加进去**

比如我们想添加多“游政杰”这个分词，就可以在my.dic输入进去

**3.重启所有服务即可**

```text
GET _analyze
{
  "text": ["我是游政杰，very nice"],
  "analyzer": "ik_max_word"
  
  
  
}
```

```json
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "游政杰",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "very",
      "start_offset" : 6,
      "end_offset" : 10,
      "type" : "ENGLISH",
      "position" : 3
    },
    {
      "token" : "nice",
      "start_offset" : 11,
      "end_offset" : 15,
      "type" : "ENGLISH",
      "position" : 4
    }
  ]
}

```


### elasticsearch的操作（REST风格）

**下面的操作使用Kibana作为可视化工具去操作es ,也可以使用postman去操作**

**method	       url地址	                                                     描述**
**PUT	localhost:9100/索引名称/类型名称/文档id 	创建文档（指定id）**
**POST	localhost:9100/索引名称/类型名称	        创建文档（随机id）**
**POST	localhost:9100/索引名称/文档类型/文档id/_update	修改文档
DELETE	localhost:9100/索引名称/文档类型/文档id	删除文档
GET	localhost:9100/索引名称/文档类型/文档id	查询文档通过文档id
POST	localhost:9100/索引名称/文档类型/_search	查询所有文档**



**可以看到，elasticsearch和原生的RESTful风格有点不同，区别是PUT和POST，原生RestFul风格的PUT是用来修改数据的，POST是用来添加数据的，而这里相反**

**PUT和POST的区别：**

**PUT具有幂等性，POST不具有幂等性，也就是说利用PUT无论提交多少次，返回结果都不会发生改变，这就是具有幂等性，而POST我们可以把他理解为uuid生成id，每一次的id都不同，所以POST不具有幂等性**





#### 创建索引

**模板：PUT /索引名**

**例1：**

**创建一个索引名为hello01，类型为_doc，documentid（记录id）为001的记录，PUT一定要指定一个documentid，如果是POST的话可以不写，POST是随机给documentid的，因为post是不具有幂等性的**

```text
PUT /hello03
{
  //请求体，为空就是没有任何数据
}
```

**返回结果**

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "hello03"
}
```





#### 删除索引

```text
DELETE hello01
{
  
}
```


#### 往索引插入数据（document）

```text
PUT /hello03/_doc/1
{
  "name": "yzj",
  "age" : 18
  
}
```



**结果:**

```json
{
  "_index" : "hello03",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```



**然后我们查看一下hello03的索引信息：**

```text
{
"state": "open",
"settings": {
"index": {
"creation_date": "1618408917052",
"number_of_shards": "1",
"number_of_replicas": "1",
"uuid": "OEVNL7cCQgG74KMPG5LjLA",
"version": {
"created": "7060199"
},
"provided_name": "hello03"
}
},
"mappings": {
"_doc": {
"properties": {
"name": {
"type": "text",
"fields": {
"keyword": {
"ignore_above": 256,
"type": "keyword"    //name的底层默认用了keyword（不可分词）
}
}
},
"age": {
"type": "long"  //age用了long
}
}
}
},
"aliases": [ ],
"primary_terms": {
"0": 1
},
"in_sync_allocations": {
"0": [
"17d4jyS9RgGEVid4rIANQA"
]
}
}
```

**我们可以看到，如果我们没有指定字段类型，就会使用es默认提供的**

**例如上面的name，默认用了keyword，不可分词**

**所以我们很有必要在创建时就指定类型**







#### 删除索引中指定的数据（根据id）

```text
DELETE hello01/_doc/004
{
  
}

```







#### 修改索引中指定的数据

```text
POST hello02/_update/001
{
  "doc": {
     "d2":"Java"
    
  }
 
}
```



#### 删除索引中指定的数据

```text
DELETE hello02/_doc/001
{
  
  
}
```

 



#### 创建映射字段

```text
PUT /hello05
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "say":{
        "type": "text",
        "analyzer": "ik_max_word"
      }
    }
  }
}
```

**查看一下hello05索引信息：**

```text
{
"state": "open",
"settings": {
"index": {
"creation_date": "1618410744334",
"number_of_shards": "1",
"number_of_replicas": "1",
"uuid": "isCuH2wTQ8S3Yw2MSspvGA",
"version": {
"created": "7060199"
},
"provided_name": "hello05"
}
},
"mappings": {
"_doc": {
"properties": {
"name": {
"analyzer": "ik_max_word",     //说明指定字段类型成功了
"type": "text"
},
"say": {
"analyzer": "ik_max_word",
"type": "text"
}
}
}
},
"aliases": [ ],
"primary_terms": {
"0": 1
},
"in_sync_allocations": {
"0": [
"lh6O9N8KQNKtLqD3PSU-Fg"
]
}
}
```



##### 指定索引映射字段只能使用一次***

**我们再重新往hello05索引添加mapping映射：**

```text
PUT /hello05
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "say":{
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "age":{
        "type": "integer"
      }
    }
  }
}
```



**然后，报错了！！！！！！**

```json
{
  "error" : {
    "root_cause" : [
      {
        "type" : "resource_already_exists_exception",
        "reason" : "index [hello05/isCuH2wTQ8S3Yw2MSspvGA] already exists",
        "index_uuid" : "isCuH2wTQ8S3Yw2MSspvGA",
        "index" : "hello05"
      }
    ],
    "type" : "resource_already_exists_exception",
    "reason" : "index [hello05/isCuH2wTQ8S3Yw2MSspvGA] already exists",
    "index_uuid" : "isCuH2wTQ8S3Yw2MSspvGA",
    "index" : "hello05"
  },
  "status" : 400
}
```

**注意：==============**

**原因是：在我们创建了索引映射属性后，es底层就会给我们创建倒排索引（不可以再次进行修改），但是可以添加新的字段，或者重新创建一个新索引，用reindex把旧索引的信息放到新索引里面去。**

**所以：我们在创建索引mapping属性的时候要再三考虑**

**不然，剩下没有指定的字段就只能使用es默认提供的了**





##### 使用"_mapping"，往索引添加字段

**我们上面说过，mapping映射字段不能修改，但是没有说不能添加，添加的方式有一些不同。**

```text
PUT hello05/_mapping
{
   
    
    "properties": {
      
      "ls":{
        "type": "keyword"
      }
      
    }
    
   
  
}
```



##### 使用_reindex实现数据迁移



**使用场景：当mapping设置完之后发现有几个字段需要“修改”，此时我们可以先创建一个新的索引，然后定义好字段，然后把旧索引的数据全部导入进新索引**



```text
POST _reindex
{
  
  "source": {
    "index": "hello05",
    "type": "_doc"
  }, 
  
  "dest": {
    "index": "hello06"
  }
  
  
}
```



```json
#! Deprecation: [types removal] Specifying types in reindex requests is deprecated.
{
  "took" : 36,
  "timed_out" : false,
  "total" : 5,
  "updated" : 0,
  "created" : 5,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}

```

 


#### 获取索引信息

```text
GET hello05
{
    
}
```



#### 获取指定索引中所有的记录（_search）

```text
GET hello05/_search
{
  "query": {
    
    "match_all": {}
    
  }
}
```

 

#### 获取索引指定的数据



```text
GET hello05/_doc/1
{ 
}
```



#### 获取指定索引全部数据(match_all:{})

```text
GET hello05/_search
{ 
}
```

**和上面的是一样的**

```text
GET hello05/_search
{
  "query": {
    
    "match_all": {}
    
  }
  
  
}
```


#### match查询(只允许单个查询条件)

**match查询是可以把查询条件进行分词的。**

```text
GET hello05/_search
{
   "query": {
     
     "match": {
        "name": "李"   //查询条件
        
     }
     
   }
}
```

````json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.9395274,
    "hits" : [
      {
        "_index" : "hello05",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.9395274,
        "_source" : {
          "name" : "李四",
          "age" : 3
        }
      },
      {
        "_index" : "hello05",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 0.79423964,
        "_source" : {
          "name" : "李小龙",
          "age" : 45
        }
      }
    ]
  }
}
````

##### 如果我们再加多一个查询条件

```text
GET hello05/_search
{
   "query": {
     
     "match": {
        "name": "李"
        , "age": 45
     }
     
   }
  
}

```

**就会报错，原因是match只允许一个查询条件，多条件可以用query   bool  must 来实现**

```json
{
  "error" : {
    "root_cause" : [
      {
        "type" : "parsing_exception",
        "reason" : "[match] query doesn't support multiple fields, found [name] and [age]",
        "line" : 6,
        "col" : 18
      }
    ],
    "type" : "parsing_exception",
    "reason" : "[match] query doesn't support multiple fields, found [name] and [age]",
    "line" : 6,
    "col" : 18
  },
  "status" : 400
}

```

 



#### 精准查询(term)和模糊查询(match)区别

**match:**

```text
GET hello05/_search
{
  "query": {
     
     "match": {
       "name": "李龙"
     }
    
  }
  
  
}
 
```

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 2.0519087,
    "hits" : [
      {
        "_index" : "hello05",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 2.0519087,
        "_source" : {
          "name" : "李小龙",
          "age" : 45
        }
      },
      {
        "_index" : "hello05",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.9395274,
        "_source" : {
          "name" : "李四",
          "age" : 3
        }
      }
    ]
  }
}
```

**==================**

**term :**

```text
GET hello05/_search
{
  "query": {
     
     "term": {
       "name": "李龙"
     }
    
  }
  
  
}
 
```



```text
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```



**区别是：**

**1：match的查询条件是会经过分词器分词的，然后再去和倒排索引去对比（对比term效率较低）**

**2：term的查询条件是不会分词的，是直接拿去和倒排索引去对比的，效率较高**

**3:同样term也是只能支持一个查询条件的**





####  multi_match实现类似于百度搜索

**match和multi_match的区别在于match只允许传入的数据在一个字段上搜索，而multi_match可以在多个字段中搜索**

**例如：我们要实现输入李小龙，然后在title字段和content字段中搜索，就要用到multi_match，普通的match不可以**

**模拟京东搜索商品**

```text
PUT /goods
{
  "mappings": {
    
    "properties": {
      
      "title":{
        "analyzer": "standard",
        "type" : "text"
      },
      "content":{
        "analyzer": "standard",
        "type": "text"
      }
      
    }
    
  }
  
  
  
}
```



```text
GET goods/_search
{
  
  "query": {
    //下面输入华为，会进行分词，然后在title和content两个字段中搜索
    "multi_match": {
      "query": "华为",
      "fields": ["title","content"]
    }
    
  }
   
  
}
```

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.1568705,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.1568705,
        "_source" : {
          "title" : "华为Mate30",
          "content" : "华为Mate30 8+128G，麒麟990Soc",
          "price" : "3998"
        }
      },
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0173018,
        "_source" : {
          "title" : "华为P40",
          "content" : "华为P40 8+256G，麒麟990Soc，贼牛逼",
          "price" : "4999"
        }
      }
    ]
  }
}
```

 

#### 短语(精准)搜索(match_phrase)

```text
GET goods/_search
{
  "query": {
    
    "match_phrase": {
      "content": "华为P40手机"
    }
    
  }
   
}
 
```

**结果查不到数据，原因是match_phrase是短语搜索，也就是精确搜索**

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}
```
 
#### 指定查询显示字段(_source)

**elasticsearch默认的显示字段规则类似于MYSQL的select * from xxx      ，我们可以自定义成类似于select id,name from xxx**

```text
GET goods/_search
{
  
  "query": {
    
    "multi_match": {
      "query": "华为",
      "fields": ["title","content"]
    }
       
  }
   , "_source" :  ["title","content"]  //指定只显示title和content
  
}
```

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.1568705,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.1568705,
        "_source" : {
          "title" : "华为Mate30",
          "content" : "华为Mate30 8+128G，麒麟990Soc"
        }
      },
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0173018,
        "_source" : {
          "title" : "华为P40",
          "content" : "华为P40 8+256G，麒麟990Soc，贼牛逼"
        }
      }
    ]
  }
}
```
 

#### 排序sort

**因为前面设计索引mapping失误，price没有进行设置，导致price是text类型，无法进行排序和filter range，所以我们再添加一个字段，od**

```text
POST goods/_update/1
{
  "doc": {
    
    "od":1
    
  }
}

```

**省略2 3 4**

```text
GET goods/_search
{
  
  "query": {
    
    "multi_match": {
      "query": "华为",
      "fields": ["title","content"]
    }
    
  }
  , "sort": [
    {
      "od": {
        "order": "desc"  //asc升序，desc降序
      }
    }
  ]
    
  
}
```

 

#### 分页

```text
GET goods/_search
{
   "query": {
     
     "match_all": {}
     
   }
   , "sort": [
     {
       "od": {
         "order": "desc"
       }
     }
   ]
 , "from" : 0
   , "size": 2
}
```



```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : null,
        "_source" : {
          "title" : "IQOONEO5",
          "content" : "IQOONEO5 高通骁龙870Soc ,",
          "price" : "2499",
          "od" : 4
        },
        "sort" : [
          4
        ]
      },
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : null,
        "_source" : {
          "title" : "小米11",
          "content" : "小米11 高通骁龙888Soc ,1亿像素",
          "price" : "4500",
          "od" : 3
        },
        "sort" : [
          3
        ]
      }
    ]
  }
}

```

 

#### 字段高亮（highlight）

**可以选择一个或者多个字段高亮，然后被选择的这些字段如果被条件匹配到则会默认加em标签**



```text
GET goods/_search
{
   "query": {
     
     "match": {
       "title": "华为P40"
     }
     
   },
   "highlight": {
     
     "fields": {
       
       "title": {}
       
     }
     
   }
   
}
```

**结果**

```json
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 2.7309713,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 2.7309713,
        "_source" : {
          "title" : "华为P40",
          "content" : "华为P40 8+256G，麒麟990Soc，贼牛逼",
          "price" : "4999",
          "od" : 1
        },
        "highlight" : {
          "title" : [
            "<em>华</em><em>为</em><em>P40</em>"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.5241971,
        "_source" : {
          "title" : "华为Mate30",
          "content" : "华为Mate30 8+128G，麒麟990Soc",
          "price" : "3998",
          "od" : 2
        },
        "highlight" : {
          "title" : [
            "<em>华</em><em>为</em>Mate30"
          ]
        }
      }
    ]
  }
}
```

**默认是em标签，我们可以更改他的前缀和后缀，利用前端的知识**

```text
GET goods/_search
{
   "query": {
     
     "match": {
       "title": "华为P40"
     }
     
   },
   "highlight": {
     "pre_tags": "<span style='color: red'>",
     "post_tags": "</span>" ,
     "fields": {
       
       "title": {}
       
     }
     
   }
   
}
```

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 2.7309713,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 2.7309713,
        "_source" : {
          "title" : "华为P40",
          "content" : "华为P40 8+256G，麒麟990Soc，贼牛逼",
          "price" : "4999",
          "od" : 1
        },
        "highlight" : {
          "title" : [
            "<span style='color: red'>华</span><span style='color: red'>为</span><span style='color: red'>P40</span>"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.5241971,
        "_source" : {
          "title" : "华为Mate30",
          "content" : "华为Mate30 8+128G，麒麟990Soc",
          "price" : "3998",
          "od" : 2
        },
        "highlight" : {
          "title" : [
            "<span style='color: red'>华</span><span style='color: red'>为</span>Mate30"
          ]
        }
      }
    ]
  }
}

```
 
##### 模仿百度搜索高亮

![image-20210417165852357](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210417165852357.png)



**例如百度搜索华为P40，不仅仅是title会高亮，content也会高亮，所以我们可以用multi_match+highlight实现**

```text
GET goods/_search
{
  "query": {
   
   "multi_match": {
     "query": "华为P40",
     "fields": ["title","content"]
   }
  }
  
  , "highlight": {
    "pre_tags": "<span style='color: red'>",
    "post_tags": "</span>", 
    "fields": {
      
      "title": {},
      "content": {}
    }
    
  }
  
  
  
}
```

```json
{
  "took" : 8,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 2.8157697,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 2.8157697,
        "_source" : {
          "title" : "华为P40",
          "content" : "华为P40 8+256G，麒麟990Soc，贼牛逼",
          "price" : "4999",
          "od" : 1
        },
        "highlight" : {
          "title" : [
            "<span style='color: red'>华</span><span style='color: red'>为</span><span style='color: red'>P40</span>"
          ],
          "content" : [
            "<span style='color: red'>华</span><span style='color: red'>为</span><span style='color: red'>P40</span> 8+256G，麒麟990Soc，贼牛逼"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.8023796,
        "_source" : {
          "title" : "华为Mate30",
          "content" : "华为Mate30 8+128G，麒麟990Soc",
          "price" : "3998",
          "od" : 2
        },
        "highlight" : {
          "title" : [
            "<span style='color: red'>华</span><span style='color: red'>为</span>Mate30"
          ],
          "content" : [
            "<span style='color: red'>华</span><span style='color: red'>为</span>Mate30 8+128G，麒麟990Soc"
          ]
        }
      }
    ]
  }
}
```
 
#### bool查询(用作于多条件查询)

**类似于MYSQL的and  or**

**重点：must 代表and   ，should  代表  or**

**must（and）的使用：**

**下面我们在must里面给了两个条件，如果这里是must，那就必须两个条件都要满足**

```text
GET goods/_search
{
  
  "query": {
     
      "bool": {
        
        "must": [
          {
          "match": {
            "title": "华为"
          }
          },
          {
            "match": {
              "content": "MATE30"
            }
          }  
        ] 
        
      }
  }
}
```

**结果：**

```json
{
  "took" : 10,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 2.9512205,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 2.9512205,
        "_source" : {
          "title" : "华为Mate30",
          "content" : "华为Mate30 8+128G，麒麟990Soc",
          "price" : "3998",
          "od" : 2
        }
      }
    ]
  }
}
```



**should（or）的使用：**

**should里面同样有两个条件，但是只要满足一个就可以了**

```text
GET goods/_search
{
  
  "query": {
     
      "bool": {
        
        "should": [
          {
          "match": {
            "title": "华为"
          }
          },
          {
            "match": {
              "content": "MATE30"
            }
          }
          
          
        ] 
        
      }
  }
}
```

**结果：**

````json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 2.9512205,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 2.9512205,
        "_source" : {
          "title" : "华为Mate30",
          "content" : "华为Mate30 8+128G，麒麟990Soc",
          "price" : "3998",
          "od" : 2
        }
      },
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.5241971,
        "_source" : {
          "title" : "华为P40",
          "content" : "华为P40 8+256G，麒麟990Soc，贼牛逼",
          "price" : "4999",
          "od" : 1
        }
      }
    ]
  }
}

````

#### 过滤器，区间条件（filter  range）

**比如我们要实现，输入title=xx，我们如果想得到price>4000作为一个条件，可以用到这个。**

```text
GET goods/_search
{
  
  "query": {
     
      "bool": {
        
        "must": [
          {
          "match": {
            "title": "小米"
          }  
          }
        ],"filter": {
          "range": {
            "price": {
              "gt": 4000
            }
          }
        }
      }
  }
}
```

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 2.4135482,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 2.4135482,
        "_source" : {
          "title" : "小米11",
          "content" : "小米11 高通骁龙888Soc ,1亿像素",
          "price" : "4500",
          "od" : 3
        }
      }
    ]
  }
}
```

 

#### 查看整个es的索引信息

```text
GET _cat/indices?v
```



### elasticsearch的Java Api

#### 准备阶段

**1.导入elasticsearch高级客户端依赖和elasticsearch依赖（注意版本要和本机的es版本一致）,我们本机现在用的是7.6.1的es**

```xml
 <!--        导入java elastic 两个依赖-->
       <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
<!--           这个版本要和你本机的elasticsearch版本一致-->
            <version>7.6.1</version>
        </dependency>

        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
<!--            这个版本要和你本机的elasticsearch版本一致-->
            <version>7.6.1</version>
        </dependency>
<!--        引入fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.75</version>
        </dependency>
```

 **2.打开RestHighLevelClient的构造器：**

```java
public RestHighLevelClient(RestClientBuilder restClientBuilder) {
        this(restClientBuilder, Collections.emptyList());
    }
```

**我们发现需要传入一个RestClientBuilder，但是这个对象我们需要通过RestClient来得到，而不是RestClientBuilder**

**3.打开RestClient：**

```
 public static RestClientBuilder builder(HttpHost... hosts) {
        if (hosts == null || hosts.length == 0) {
            throw new IllegalArgumentException("hosts must not be null nor empty");
        }
        List<Node> nodes = Arrays.stream(hosts).map(Node::new).collect(Collectors.toList());
        return new RestClientBuilder(nodes);
    }

```

**我们发现RestClient的builder可以得到RestClientBuilder，然后我们点进去看HttpHost：**

```java
public HttpHost(String hostname, int port, String scheme) { //es所在主机名，es的端口号，协议（默认http）
        this.hostname = (String)Args.containsNoBlanks(hostname, "Host name");
        this.lcHostname = hostname.toLowerCase(Locale.ROOT);
        if (scheme != null) {
            this.schemeName = scheme.toLowerCase(Locale.ROOT);
        } else {
            this.schemeName = "http";
        }

        this.port = port;
        this.address = null;
    }
```



**4.然后我们就配置好了如下：**

```java
		HttpHost httpHost = new HttpHost("localhost",9200,"http");
        RestClientBuilder restClientBuilder = RestClient.builder(httpHost);
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(restClientBuilder);
        
```

**5.为了方便，我们可以把这个RestHighLevelClient交给SpringIOC容器管理，后面我们自动注入即可**

```java
@Configuration
public class esConfig {


    @Bean
    public RestHighLevelClient restHighLevelClient(){
        HttpHost httpHost = new HttpHost("localhost",9200,"http");
        RestClientBuilder builder = RestClient.builder(httpHost);
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);
        return restHighLevelClient;
    }
 
}
```

   

#### 索引操作

**java elasticsearch api操作索引都是用restHighLevelClient.indices().xxxxx()的格式**

##### 创建索引

```java
//创建索引
    @Test
    public void createIndex() throws IOException {
        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);
        //new一个创建索引请求，并传入一个创建的索引名称
        CreateIndexRequest createIndexRequest = new CreateIndexRequest("java01");
        //向es发送创建索引请求。
        CreateIndexResponse createIndexResponse = restHighLevelClient.indices().create(createIndexRequest, RequestOptions.DEFAULT);

        restHighLevelClient.close();

    }
```

 



##### 删除索引

```java
//删除索引
    @Test
    public void deleteIndex() throws IOException {

        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);

        //new一个删除索引请求，并传入需要删除的索引名称
        DeleteIndexRequest deleteIndexRequest = new DeleteIndexRequest("java01");
        //resthighLevelClient发送删除索引请求
        restHighLevelClient.indices().delete(deleteIndexRequest,RequestOptions.DEFAULT);
        restHighLevelClient.close();

    }
```



##### 检查索引是否存在

```java
//检查索引是否存在
    @Test
    public void indexExsit() throws IOException {
        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);

        GetIndexRequest getIndexRequest = new GetIndexRequest("goods");


        boolean exists = restHighLevelClient.indices().exists(getIndexRequest, RequestOptions.DEFAULT);

        System.out.println(exists);


    }
```

 





#### 文档操作





##### 创建指定id的文档

```java
//创建文档
    @Test
    public void createIndexDoc() throws IOException {

        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);

        IndexRequest indexRequest = new IndexRequest("hello");
        //指定文档id
        indexRequest.id("1");
        /**
         *  public IndexRequest source(Map<String, ?> source, XContentType contentType) throws ElasticsearchGenerationException {
         *         try {
         *             XContentBuilder builder = XContentFactory.contentBuilder(contentType);
         *             builder.map(source);
         *             return this.source(builder);
         *         } catch (IOException var4) {
         *             throw new ElasticsearchGenerationException("Failed to generate [" + source + "]", var4);
         *         }
         *     }
         *     source有很多种方法，哪种都可以，我现在选的是Map的方法添加key:value
         */
        Map<String,Object> source=new HashMap<>();
        source.put("a_age","50");
        source.put("a_address","广州");
        //在es里面，一切皆为JSON，我们要把Map用fastjson转换成JSON字符串，XContentType指定为JSON类型
        indexRequest.source(JSON.toJSONString(source), XContentType.JSON);

        IndexResponse response = restHighLevelClient.index(indexRequest, RequestOptions.DEFAULT);

        System.out.println("response:"+response);
        System.out.println("status:"+response.status());

    }
```



##### 删除指定id的文档



```java
  //删除文档
    @Test
    public void deleteDoc() throws IOException {

        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));

        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);


        DeleteRequest deleteRequest = new DeleteRequest("hello");

        deleteRequest.id("1");

        DeleteResponse delete = restHighLevelClient.delete(deleteRequest, RequestOptions.DEFAULT);
        System.out.println(delete.status());

    }
```



##### 修改指定id的文档

```java
//修改文档
    @Test
    public void updateDoc() throws IOException {

        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);

        /**
         * 通过下面的方法去调用
         *     public UpdateRequest(String index, String id) {
         *         super(index);
         *         this.refreshPolicy = RefreshPolicy.NONE;
         *         this.waitForActiveShards = ActiveShardCount.DEFAULT;
         *         this.scriptedUpsert = false;
         *         this.docAsUpsert = false;
         *         this.detectNoop = true;
         *         this.id = id;
         *     }
         */
        UpdateRequest updateRequest = new UpdateRequest("hello","1");

        Map<String,Object> source=new HashMap<>();
        source.put("a_address","河源");
        updateRequest.doc(JSON.toJSONString(source),XContentType.JSON);
        UpdateResponse response = restHighLevelClient.update(updateRequest, RequestOptions.DEFAULT);
        System.out.println(response.status());
    }
```





##### 获取指定id的文档

```java
 //获取文档
    @Test
    public void getDoc() throws IOException {

        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);


        GetRequest getRequest = new GetRequest("hello");
        getRequest.id("1");

        GetResponse response = restHighLevelClient.get(getRequest, RequestOptions.DEFAULT);

        String sourceAsString = response.getSourceAsString();
        System.out.println(sourceAsString);

    }
```



##### 搜索(匹配全文match_all)

```java
//搜索(匹配全文match_all)
    @Test
    public void search_matchAll() throws IOException {

        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);

        /**
         *  public SearchRequest(String... indices) {
         *         this(indices, new SearchSourceBuilder());
         *     }
         */
        SearchRequest searchRequest = new SearchRequest("hello");

        //相当于文本
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        MatchAllQueryBuilder matchAllQueryBuilder = QueryBuilders.matchAllQuery();
        searchSourceBuilder.query(matchAllQueryBuilder); //相当于search的query

        searchRequest.source(searchSourceBuilder);




        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);

        SearchHit[] hits = search.getHits().getHits();

 
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
        }
    }
```



##### 搜索(模糊查询match)

```java
//模糊搜索match
    @Test
    public void search_match() throws IOException {

        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));

        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);

        SearchRequest searchRequest = new SearchRequest();

        //查询文本
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("a_address", "广州");
        searchSourceBuilder.query(matchQueryBuilder);

        searchRequest.source(searchSourceBuilder);

        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);

        SearchHit[] hits = search.getHits().getHits();
 
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
        }

    }
```



##### 搜索(多字段搜索multi_match)

```java
 //搜索(多字段搜索multi_match)
    @Test
    public void  search_term() throws IOException {
        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        searchSourceBuilder.query(QueryBuilders.multiMatchQuery("华为","title","content"));

        searchRequest.source(searchSourceBuilder);


        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);

        SearchHit[] hits = search.getHits().getHits();
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
        }

    }
```





##### 搜索(筛选字段fetchSource)

**fetchsource方法相当于_source**

```java
//fetchsource实现筛选字段(_source)
    @Test
    public void search_source() throws IOException {

        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));

        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        searchSourceBuilder.query(QueryBuilders.matchAllQuery());
        /**
         * public SearchSourceBuilder fetchSource(@Nullable String[] includes, @Nullable String[] excludes) {
         *         FetchSourceContext fetchSourceContext = this.fetchSourceContext != null ? this.fetchSourceContext : FetchSourceContext.FETCH_SOURCE;
         *         this.fetchSourceContext = new FetchSourceContext(fetchSourceContext.fetchSource(), includes, excludes);
         *         return this;
         *     }
         *
         */
        String[] includes={"title"}; //包含
        String[] excludes={}; //排除
        searchSourceBuilder.fetchSource(includes,excludes);

        searchRequest.source(searchSourceBuilder);

        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);

        SearchHit[] hits = search.getHits().getHits();
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
        }


    }
```



##### 分页、排序、字段高亮

**我们要把下面的es命令行代码转换成Java代码**

```text
GET goods/_search
{
  
  "query": {
    
      "match": {
        "title": "华为"
      }
      
    
  },"sort": [
    {
      "od": {
        "order": "desc"
      }
    }
  ]
  
  ,"from": 0,
  "size": 1,
  "highlight": {
    "pre_tags": "<span style='color:red'>",
    "post_tags": "</span>", 
    "fields": {
      
      "title": {}
    }
    
  } 
}
```



**Java 实现**

```java
//分页，排序，字段高亮
    @Test
    public void page_sort_HighLight() throws IOException {

        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("title", "华为");


        searchSourceBuilder.query(matchQueryBuilder);

        //分页====
        searchSourceBuilder.from(0);
        searchSourceBuilder.size(1);
        //=======

        //排序
        searchSourceBuilder.sort("od", SortOrder.DESC);


        //字段高亮
        //=========高亮开始==
        HighlightBuilder highlightBuilder = new HighlightBuilder();

        //构建高亮的前缀后缀标签pre_tag和post_tag
        highlightBuilder.preTags("<span style='color:blue'>");
        highlightBuilder.postTags("</span>");

        //highlightBuilder.field()方法我们用一个String类型的
        /**
         * public HighlightBuilder field(String name) {
         *         return this.field(new HighlightBuilder.Field(name));
         *     }
         */
        highlightBuilder.field("title");
        //如果还需要更多字段高亮，则多写一遍field方法
//        highlightBuilder.field(); //第二个字段高亮
//        highlightBuilder.field(); //第三个字段高亮 。。。。。以此类推

        searchSourceBuilder.highlighter(highlightBuilder);

        //====================高亮结束



        searchRequest.source(searchSourceBuilder);

        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);

        SearchHit[] hits = search.getHits().getHits(); //hits里面封装了命中的所有数据
        for (SearchHit hit : hits) {
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            System.out.println("highlightMap:"+highlightFields);
            //通过title这个key去获取fragments
            //fragment里面是高亮之后的字段内容（很重要，可以用来覆盖原来没高亮的字段内容） <span style='color:blue'>华</span><span style='color:blue'>为</span>Mate30
            System.out.println("fragments:"+Arrays.toString(highlightFields.get("title").getFragments()));
        }


        restHighLevelClient.close();


    }
```





##### 布尔搜索(bool)

**实现类似如下es代码：**

```text
GET goods/_search
{
  "query": {
    
    "bool": {
      
      "should": [
        {
         
         "term": {
           "title": {
             "value": "华"
           }
         }
          
        },
        {
          
          "term": {
            "title": {
              "value": "米"
            }
          }
          
        }
      ]
      
    }
    
  }

}
```



**Java实现：**



```java
 //布尔搜索(bool)
    @Test
    public void search_bool() throws IOException {

        RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(builder);

        SearchRequest searchRequest = new SearchRequest("goods");

        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        //通过searchSourceBuilder对象构建bool查询对象
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();

        //这里should只能写一个，如should里面有多个条件，可以写多个should
        /**
         *
         *  "should": [
         *         {
         *
         *          "term": {
         *            "title": {
         *              "value": "华"
         *            }
         *          }
         *
         *         },
         *         {
         *
         *           "term": {
         *             "title": {
         *               "value": "米"
         *             }
         *           }
         */
        //例如上面should有两个条件，我们就要写两个should
        boolQueryBuilder.should(QueryBuilders.termQuery("title","华"));
        boolQueryBuilder.should(QueryBuilders.termQuery("title","米"));
        searchSourceBuilder.query(boolQueryBuilder);
        

        searchRequest.source(searchSourceBuilder);


        SearchResponse search = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);

        SearchHit[] hits = search.getHits().getHits();
        for (SearchHit hit : hits) {
            System.out.println(hit.getSourceAsString());
        }


        restHighLevelClient.close();

    }
```





#### es实战(京东商品搜索)



##### 从京东上爬取数据

**1:导入依赖：**

```xml
	<!--        jsoup-->
         <dependency>
            <groupId>org.jsoup</groupId>
            <artifactId>jsoup</artifactId>
            <version>1.12.1</version>
        </dependency>
```



**2.创建实体类：**

```java
public class goods{

    private String img; //商品图片
    private String price; //商品价格
    private String title; //商品标题

    public goods() {
    }

    public goods(String img, String price, String title) {
        this.img = img;
        this.price = price;
        this.title = title;
    }

    public String getImg() {
        return img;
    }

    public void setImg(String img) {
        this.img = img;
    }

    public String getPrice() {
        return price;
    }

    public void setPrice(String price) {
        this.price = price;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return "goods{" +
                "img='" + img + '\'' +
                ", price='" + price + '\'' +
                ", title='" + title + '\'' +
                '}';
    }
}

```



**3.利用jsoup解析爬取京东商城搜索(核心)，编写工具类：**

```java
@Component
public class jsoupUtils {


    private static RestHighLevelClient restHighLevelClient;

    @Autowired
    public  void setRestHighLevelClient(RestHighLevelClient restHighLevelClient) {
        jsoupUtils.restHighLevelClient = restHighLevelClient;
    }

    /**
     *封装了京东搜索功能，把搜索的数据添加进es中
     */
    public static void searchData_JD(String keyword) {

        BulkRequest bulkRequest = new BulkRequest();

        try {
            URL url = null;
            try {
                url = new URL("https://search.jd.com/Search?keyword=" + keyword);
            } catch (MalformedURLException e) {
                e.printStackTrace();
            }

            Document document = null;//jsoup解析URL
            try {
                document = Jsoup.parse(url, 30000);
            } catch (IOException e) {
                e.printStackTrace();
            }

            Element e1 = document.getElementById("J_goodsList");

            Elements e_lis = e1.getElementsByTag("li");

            for (Element e_li : e_lis) {

                //这边可能获取到多个价格，因为有些有套餐价格，我们可以获取第一个价格
                Elements e_price = e_li.getElementsByClass("p-price");
                String text = e_price.get(0).text();
                //这里获取的价格可能有多个，正常价和京东PLUS会员专享价，所以我们要进行切分
                String realPirce = "￥";
                int x = 1; //默认第一个就是￥的符号，也从1开始遍历，如果还有￥符号就break即可
                for (int i = 1; i < text.length(); i++) {

                    if (text.charAt(i) == '￥') {
                        break;
                    } else {
                        realPirce += text.charAt(i);
                    }

                }
                //商品图片
                Elements e_img = e_li.getElementsByClass("p-img");
                Elements img = e_img.get(0).getElementsByTag("img");
                //因为京东的商品图片不是封装到src里面的，而是封装到懒加载属性==data-lazy-img
                String src = img.get(0).attr("data-lazy-img");
                System.out.println("http:" + src);


                //价格
                System.out.println(realPirce);
                //商品标题
                Elements e_title = e_li.getElementsByClass("p-name");
                String title = e_title.get(0).getElementsByTag("em").text();
                System.out.println(title);

                IndexRequest indexRequest = new IndexRequest("jd_goods");

                //添加信息
                Map<String,Object> good=new HashMap<>();
                good.put("img","http:" + src);
                good.put("price",realPirce);
                good.put("title",title);
                IndexRequest source = indexRequest.source(JSON.toJSONString(good), XContentType.JSON);

                bulkRequest.add(source);


            }
            //批量操作，减少访问es服务器的次数
              restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);

        }catch (Exception e){
            System.out.println(e.getMessage());
        }



    }
}


```





**4.使用工具类：**

```java
public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);

        jsoupUtils.searchData_JD("vivo"); 

    }
```

 

**有了数据我们就可以用来展示到页面上了。。。。。**




## Spring(部分)

**Spring由两大部分组成，IOC和AOP。**

### SpringIOC

Spring IOC是通过**工厂模式**实现的

#### Bean

当我们在xml文件**创建每一个bean**的时候，当**容器被初始化**的时候，Spring就会通过这个**bean的所属类**通过这个类的**无參构造方法**创建这个类的对象

##### Bean注入方式

> 1：Set注入，也就是用property（最常用）

* 首先要该属性的**public的Set/get方法**才能注入成功

```xml
<bean id="Setzhuru" class="zhuru.zhuru"  scope="singleton">
       <property name="id" value="11111"></property>
       <property name="name" value="Set注入成功"></property>
</bean>
```

> 2：构造方法注入（constructor -arg）

* 首先要**有构造方法才能注入**

```java
public zhuru(int id,String name){
        this.id=id;
        this.name=name;
}
```

```xml
<bean id="gouzao" class="zhuru.zhuru">
          <constructor-arg value="22222"></constructor-arg>
         <constructor-arg value="构造方法注入成功"></constructor-arg>
</bean>
```

**两个constructor -arg代表构造方法为两个参数，如果在写多一个,constructor -arg就会报错，因为没有创建三个参数的构造方法**


##### p命名空间

**p命名空间 property的缩写：首先我们要在idea的xml文件上用xmlns:p="http://www.springframework.org/schema/p"写在命名空间上，才能使用p命名空间**

例如：

```xml
<bean id="Pname" class="zhuru.zhuru" p:id="33333" p:name="P命名空间注入了"  >
</bean>
```

##### 字面量和ref

* 字面量
  * 不能用ref去引用的，可以直接用value去赋值（注入）的属性，比如说：int 、Integer、String、boolean、double、float。。等等。。。
* 非字面量
  * 比如：**数据类型是自己创建的类**的属性等等
  * 只能用ref去引用，也就是一个引用的数据类型，不可以直接用
  * value去赋值的一个属性，就是非字面量。。。

例如：

```java
public class zhuru {
    //注入
    private int id;
    private String name;
    private zhuru_ref ref;//******这个就是非字面量
｝
public class zhuru_ref {


    private int age;
    private String address;
｝
```

```xml
<bean id="noref" class="zhuru.zhuru">
        <property name="id" value="55555"></property>
        <property name="name" value="id和name可以直接用value赋值，他们是字面量"></property>
        <property name="ref" >
            <ref bean="isref"></ref>
        </property>

</bean>
<bean id="isref" class="zhuru.zhuru_ref">
      <property name="age" value="18"></property>
      <property name="address" value="字面量"></property>
</bean>
```

##### 内部bean的使用场景

```xml
<bean id="noref" class="zhuru.zhuru">
        <property name="id" value="55555"></property>
        <property name="name" value="id和name可以直接用value赋值，他们是字面量"></property>
        <property name="ref" >
            <bean id="isref" class="zhuru.zhuru_ref">
         这里就是内部bean*************
                <property name="age" value="18"></property>
                <property name="address" value="内部bean"></property>
            </bean>
             
        </property>

</bean>
```


##### 为集合属性赋值

> 命名空间

```text
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"
>
```


> 需要赋值的类

```java
public class listzhuru1 {
    private List<Integer> numbList=new ArrayList<>();//纯数据类型的赋值
    private List<listzhuru2> list2=new ArrayList<>();//纯类的对象赋值
｝

public class listzhuru2 {
    private int id;
    private String name;
｝
```


> 使用list标签

```xml
<bean id="list1" class="listzhuru.listzhuru1">
         <property name="numbList">
             <list>
                 <value>111</value>
                 <value>222</value>
                 <value>333</value>
             </list>
         </property>

    </bean>
```


> 使用util命名空间

```xml
<bean id="list1" class="listzhuru.listzhuru1">
         <property name="numbList"  ref="utilList1">
 
         </property>
        


    </bean>
<!--      用util命名空间，再使用util:list标签为集合属性赋值-->
       <util:list id="utilList1">
           <value>555</value>

       </util:list>
```


##### 为Map数据结构赋值

```xml
1. 方法一，就是直接通过在property标签里面加入<Map>标签注入：
如下
<bean id="map" class="Mapzhuru.Mapzhuru">
        <property name="map">
            <map>
                <entry>
                    <key>
                        <value>张三</value>
                    </key>
                    <value>100</value>
                </entry>


                <entry>
                    <key>
                        <value>李四</value>

                    </key>

                    <value>200</value>
                </entry>
                
            </map>
方法二：使用util集合类型的bean为map赋值：

<bean id="utilMap" class="Mapzhuru.Mapzhuru" scope="singleton">
        <property name="map">
            <ref bean="maps"></ref>
        </property>

    </bean>

    <util:map id="maps">
        <entry>
            <key>
                <value>utilMap赋值成功</value>
            </key>
            <value>99999</value>
        </entry>

    </util:map>
```


##### 集合类型的bean

> 命令空间

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"
>
```
> 使用util集合类型的bean为map赋值

```xml
<bean id="utilMap" class="Mapzhuru.Mapzhuru" scope="singleton">
        <property name="map">
            <ref bean="maps"></ref>
        </property>

    </bean>

    <util:map id="maps">
        <entry>
            <key>
                <value>utilMap赋值成功</value>
            </key>
            <value>99999</value>
        </entry>

    </util:map>
```


#### FactoryBean

**FactoryBean其实就是用了工厂模式，他把创建对象的过程隐藏起来，放在getObject里面**

```java
implements FactoryBean<泛型（也就是这个工厂要生产的对象）>

@Override
    public car getObject() throws Exception {
//这一步，其实就是把创建对象的过程隐藏起来
        car car=new car();
        car.setId(111);
        car.setName("宝马");
        return car;
    }

    @Override
    public Class<?> getObjectType() {
        return car.class;
    }

```

#### bean的作用域


> 单例模式

**scope="singleton"**
此时当一个bean是singleton的，**当Spring容器初始化时候**，他就自动的通过这个bean的class的**无參构造**创建对象。

```xml
<bean id="single" class="scope.User" scope="singleton">

    </bean>
```

> 多例模式

**scope="prototype"**
当一个bean是prototype的时候，当**Spring使用时才会创建这个bean的class的对象**，也是通过**无參构造**，但是不同之处是**多例模式是用到才创建对象**，**初始化容器不会创建任何对象**。

```xml
 <bean id="proto" class="scope.User" scope="prototype">

    </bean>
```

#### bean的生命周期

* 1:首先就是创建bean的对象(无參构造创建对象)
* 2:对这个bean进行注入（set注入）
* 3:将这个bean进行初始化（init-method）
* 4:使用这个bean
* 5:销毁这个bean（destroy-method）

> 代码展示Bean的生命周期

```java
    public sm(){
        System.out.println("1.bean创建对象成功");
    }

       public void setId(int id) {
        System.out.println("2.bean注入id成功");

        this.id = id;
    }
    public void init(){
        System.out.println("3.bean已经被初始化完毕了");
    }
public String toString() {
        System.out.println("4.使用bean");
        return "";
    }
    public void destroy(){
        System.out.println("5.bean被销毁了！！！");
    }
```

**这时候‘销毁bean的方法不会生效’，因为只有在容器关闭之后才能销毁bean，这时我们要这样**

> 销毁bean,close()方法

```java
ClassPathXmlApplicationContext ac=new ClassPathXmlApplicationContext("beansm.xml");
        sm sm = ac.getBean("sm", sm.class);
        System.out.println(sm);
        ac.close();//只有调用了close方法才能销毁bean
```


#### bean的后置处理器BeanPostProcessor

**个人理解：就是当这个xml文件配置了bean的后置处理器，当getbean()去使用bean对象的时候，他就会在这些数据进行处理，不使用他就不会发生作用**

* 实现 BeanPostProcessor接口

```java
@Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        car car=(car)bean;//说明他会找到car类型
        if(car.getId()==12223){
            car.setId(66666);
        }
        return car;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
```

> 注册后置处理器

**其实就是把实现了BeanPostProcessor接口的类配置到bean里面去，这时bean会自动寻找符合条件的bean**

```xml
<bean class="bean_post.postTest"></bean>
```

#### 引用外部资源文件和使用Druid数据库连接池

创建一个bean，引入PropertyPlaceholderConfigurer类，在里面property注入location，这个location的值就填写properties配置文件全名，xxx.properties

> 引入外部资源文件

```xml
<bean id="pro" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
       <property name="location" value="db.properties"/>
   </bean>
```

> 使用Druid数据库连接池

**db.properties文件:**

```properties
driverClassName:com.mysql.jdbc.Driver

userName:root

password:18420163207

url:jdbc:mysql://localhost:3306/xscj
```

```xml
<bean id="db" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${driverClassName}"></property>
        <property name="url" value="${url}"></property>
        <property name="username" value="${userName}"></property>
        <property name="password" value="${password}"></property>
    </bean>
```

> 使用Druid数据库连接池访问xscj数据库的xs表的使用

```java
ApplicationContext ac=new ClassPathXmlApplicationContext("database.xml");
        DruidDataSource db = ac.getBean("db", DruidDataSource.class);
        DruidPooledConnection conn = db.getConnection();
        Statement stmt = conn.createStatement();
        String sql="Select * from xs";
        ResultSet res = stmt.executeQuery(sql);
        while(res.next()){//判断res里面是否有下一行，有就打印
            String s1 = res.getString(1);
            String s2 = res.getString(2);
            String s3 = res.getString(3);
            int s4 = res.getInt(4);
            Date s5 = res.getDate(5);
            int s6 = res.getInt(6);
            System.out.println(s1+"    "+s2+"    "+s3+"    "+s4+"    "+s5+"    "+s6);
        }
```


#### 自动装配和兼容性

在bean的定义后面加个autowire ，有两种类型：
其实自动装配就是自动的为‘’非字面量‘’赋值，也就是可以通过ref去引用bean的属性，只有这些才能自动装配，字面量不可以，也就是可以直接通过value赋值的不可以autowire


byName:其实就是把这个要赋值的非字面量属性的属性名和Spring所管理的bean的id相比较，相同就赋值。

byType:

#### 基于xml的自动装配和基于注解组件的扫描

自动装配autowired注解为某个属性赋值

```java
@Autowired  
  //  @Qualifier("carImpl1")   这个是指定bean的id   carImpl1是bean id
    private carInteface cars;
```

> 注解组件的扫描

先引入context的命名空间和网址

```xml
 xmlns:context="http://www.springframework.org/schema/context"

http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context.xsd
```

**例如扫描AutoWire包下的注解组件:**

```xml
<context:component-scan base-package="AutoWire">

     </context:component-scan>
```

> 常用组件注解类型

```java
@component("xxx")     普通注解组件
@Repository ("xxx")     持久层注解组件
@Service            业务逻辑层注解组件
@Controller       控制层注解组件
```

**默认扫描到的bean名是类名首字母小写的类名**

#### 扫描注解组件的包含和排除

```xml
<context:component-scan base-package="扫描的包名" use-default-filters="布尔值">
    可以选填下面的筛选语句“”之一“”，注意千万不可以把下面两句都填，可选其一

     </context:component-scan>

<context:exclude-filter type="" expression=""/>
*******重点：exclude就是‘’排除‘’，他必须要建立在所有包都被扫描过的情况下才行，不然会报错，=====也就是使用exclude之前要在use-default-filters设置为true=======



 <context:include-filter type="" expression=""/>
重点：include，也就是‘包含’，他要建立在没有扫描的情况下，不然没有作用，但是也不会exception，只是单纯的没作用，所以要让他有作用，就要把use-default-filters设置为false

```

**exclude和include不能同时用.**


#### 基于注解的自动装配

**@Autowired:**

* @Autowired注解组件就会**优先使用byType**，如果byType无法为这个加了@Autowired注解的属性赋值，他就会**切换到byName**




### Spring AOP

**AOP的很多操作都要结合IOC进行**

AOP是面向切面编程,比如我们要在一个计算器方法中增加一个日志功能:

* 如果我们是用面向对象的话，我们就直接在计算器方法中直接添加代码。
* 如果我们是用面向切面的话，我们就可以先把这个方法抽取出来，再进行更改

> 切面

其实就是**公共功能的‘类’**，“一个切面包含了一个至多个的通知”，例如上面例子的日志功能就是公共功能，我们把这个各个公共功能叫做’横切关注点‘，把这些横切关注点抽取到一个‘类’中这个类就是切面。。，切面的好处就是：‘不必修改源代码，就可以对这个方法进行增强或者更改，还有就是便于维护，业务模块更加简洁。。’


#### Aop使用场景

**如果在不修改源代码的情况下，我们要对某个已存在的方式进行增强，我们可以用AOP**

> AOP底层原理

**使用的是动态代理（Proxy）**

* 如果要增强的方法有接口（**实现了接口**）的话，就使用这个**JDK动态代理**  ====import java.lang.reflect.Proxy;
* 如果要增强的方法**没有接口**的话，就使用**CGLIB动态代理**


#### JDK动态代理

* 创建接口的‘’实现类‘’的代理对象
* 通过这个代理对象去对实现类方法的增强，或者是一些改变

> 需要被增加的实现类

```java
import java.lang.reflect.Proxy;

public interface UserDao {

    int add(int i,int j);

}
public class UserImpl implements UserDao {
    /**
     *如果要该类的add方法进行改动，而不修改这里的源代码，我们可以用动态代理
     * 又因为这个类实现了接口，我们可以用JDK动态代理
     * 反之 CGLIB动态代理
     *
     *
     */

    @Override
    public int add(int i, int j) {
        return i+j;
    }

}
```

> 代理类

```java
public static void main(String[] args) throws ClassNotFoundException {
        UserImpl impl=new UserImpl();//创建目标类的对象，以供method.invoke()方法使用
        Class<?> cla = Class.forName("AOP底层实现.UserImpl");//用反射去获取要增强的类的所有信息
        UserDao user=(UserDao) Proxy.newProxyInstance(cla.getClassLoader(), cla.getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                /**
                 * 这个类的方法是创建代理对象，对方法进行增强
                 */
                //******获取调用的方法,第一个参数是目标类的对象，第二个是args参数数组
                System.out.println("创建代理对象");
                Object result = method.invoke(impl, args);
                System.out.println("method.invoke调用了返回了结果="+result);//method.invoke就是控制 方法的执行
                return result;//返回结果
            }
        });
        int res = user.add(2, 3);
        System.out.println(res);


    }
```

#### CGLIB动态代理过程

* 创建这个类的子类的代理对象
* 通过这个子类的代理对象去对父类的方法进行增强


#### AOP术语

> 横切关注点

抽取出来的公共功能

> 切面

存储一个至多个通知的类

> 通知

横切关注点存储到切面的类，就变成了通知，和横切关注点的区别，只是在不同的位置，叫法不一样罢了

* 5种通知类型
  * 前置通知 :方法执行之前执行的代码  ===@Before
  * 后置通知：方法执行之后执行的代码 ===@After
  * 返回通知：===@AfterReturning
  * 异常通知：其实就是当方法出现异常时才会调用的方法===@AfterThrowing
  * 环绕通知

> 目标

被通知的对象，也就是通知所作用的对象，目标被切面作用

> 代理

向目标对象通知之后创建的代理对象

> 连接点

其实就是Spring允许你使用通知的地方，或者说和方法有关的前前后后（包括抛出异常）都是连接点

> 切入点

切入点就是作用的连接点的条件   ，切面作用的点，一定要有切入点，不然切面不知道作用在哪里（切入点其实就是指定“切面要作用哪个方法，也就是切入点表达式    execution(xxx)”）

> 切面类

```java
@Component
@Aspect//切面的注解，说明这个类是切面
public class myLogAspect {
    /**
     *一个切面用来存储非业务逻辑、公共功能
     */
    //切入点表达式的格式：execution([可见性]返回类型[声明类型].包名.类名.方法名(参数)[异常])

/*
@Before:将方法标注为前置通知
*/
=====
    @Before(value = "execution(public int cglib动态代理.UserImpl.add(int ,int))")//前置通知注解,括号要有切入点表达式(也就是execution(访问修饰符,返回值类型,实现类的方法全名))
    public void beforeLog(){//前置'通知'
        System.out.println("方法执行之前===前置通知");
    }

=======
/**
 @After:将方法标注为后置通知
    作用于finally语句，即不管怎么样，有没有异常，这个方法都会执行
*/
    @After(value = "execution(public int cglib动态代理.UserImpl.add(int,int))")
    public void afterLog(){//后置'通知'
        System.out.println("方法执行之后===后置通知");
    }

}
======
/**
     * @AfterReturning：标注这个方法是返回通知
     * 作用是可以获取到返回值
     * @AfterReturning(execution(xxx),returning="yyy")
     * execution(xxx):和前置通知，后置通知是”一样”的写法
     * returning="yyy":是接受返回值，并将获取到的返回值赋值到yyy这个变量中
     * 此时我们必须在形参李定义一个Object yyy,用于接收returning="yyy"的值
     */
       @AfterReturning(value = "execution(*  aspectj.*.*(..))",returning = "res")
    public void Returning(JoinPoint joinPoint,Object res){
           System.out.println("方法"+joinPoint.getSignature().getName()+"返回通知"+"result="+res);
    }
======
/**
     * @AfterThrowing:就是标注一个方法为异常通知，当这个方法出现Exception时才会执行，否则不执行
     * throwing：用来获取异常信息，这点和返回通知的returning相似，都是用来获取信息，传递到
     * 方法形参中
     */
    @AfterThrowing(value = "execution(* aspectj.UserImpl.*(..))",throwing = "ex")
    public void throwing(Exception ex){
        System.out.println("有异常了,异常类型="+ex);
    }
```

#### 获取方法信息

加入一个JoinPoint接口作为形参，通过这个JoinPoint接口的‘对象’可以获取到这个方法的信息

> JoinPoint

```java
java.lang.Object[] getArgs()：获取连接点方法运行时的参数列表；
Signature getSignature()：获取连接点的方法签名对象；
方法签名对象.getName()  可以得到方法名


java.lang.Object getTarget()：获取连接点所在的目标对象；
java.lang.Object getThis()：获取代理对象本身；
```

> JoinPoint使用

```java
@Before(value = "execution(public int aspectj.UserImpl.add(int ,int))")//前置通知注解,括号要有切入点表达式(也就是execution(访问修饰符,返回值类型,实现类的方法全名))

    public void beforeLog(JoinPoint joinPoint){//前置'通知'
        Object[] args = joinPoint.getArgs();
        Signature signature = joinPoint.getSignature();
//        System.out.println(signature.getName());//通过方法签名得到方法名字
        System.out.println(Arrays.toString(args));
        System.out.println(signature.getName()+"方法执行之前===前置通知");
    }
```

#### 切入点表达式的改进


```text
@Before(value = "execution(* aspectj.UserImpl.*(..))")
改进如下
1.我们可以把public int 也就是访问修饰符和返回值类型===改成   *  号，
这代表着不管是什么访问修饰符和返回值类型都能被作用到
2.我们可以把方法名改成*号，和方法参数改成(..)，括号中间两个点。
3.包名也可以改成*号，代表所有的意思
=================================
公共切入点：简化操作
公共切入点需要的注解：@Pointcut   
总结：@Pointcut的操作和@Before、@After一样
==***************************
定义一个******公共切入点*******：
 /**
     * 创建一个空方法，将这个方法加上====     @Pointcut注解   =====，其余写法和@Before、@After一样
     * 意思是将这个方法'=====变成一个公共的切入点====='，方便去调用
     *此时这个方法test()就变成了一个（切入点 ===  execution(* autoProxyTest.UserImpl.*(..)) ）
     */
    @Pointcut("execution(* autoProxyTest.UserImpl.*(..))")
    public void test(){
    }
====此时这个test()就相当于execution(xxx),test()就是公共切入点，故调用操作如下：
//    @Before("execution(* autoProxyTest.UserImpl.*(..))")
    @Before("test()")//要加test()
    public void before(JoinPoint joinPoint){
        System.out.println("前置通知"+joinPoint.getSignature().getName()+"方法");
    }
====注意：不是test，而是test()
 @AfterReturning(value = "test()",returning = "res")
    public int returnRes(int res){
        System.out.println("返回通知，返回res="+res);
        return res;
    }
```

#### @Order注解

**这个@Order是对bean执行顺序进行优先级设置，而不是bean的加载顺序，加载顺序是不能改变的。@Order(数字)，数字越小，优先级越高，默认数字是Int的最大值**


## SpringBoot


### 创建SpringBoot项目报错的问题

![image-20210127162355056](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210127162355056.png)


**遇到这个问题我们可以在Custom输入：https://start.springboot.io，这个是阿里云的SpringBoot镜像**

![image-20210127164110200](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210127164110200.png)


 
### 生成SpringBoot项目

![image-20210109152424990](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109152424990.png)
![image-20210109152519889](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109152519889.png)

再点击Next，勾选自己需要的模块，这样SpringBoot项目就构建好了。

------



### SpringBoot的Hello World

1.**在resources目录下的templates放页面**

![image-20210109154046821](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109154046821.png)


2.**必须把Java代码放在springBoot主程序同级的目录下（也就是当前的boot目录），不然springboot检测不到**

![image-20210109154314721](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109154314721.png)


3.controller层方法。

![image-20210109155102243](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109155102243.png)



***然后去访问这个路径就OK啦。***

![image-20210109155246006](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109155246006.png)


------



#### 运行时的异常。-datasource

![image-20210109153300025](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109153300025.png)


很显然可以看出这是关于数据源的异常，因为我们在构建SpringBoot项目时勾选了Datasource模块，SpringBoot的AutoConfiguration自动去配置数据源，而我们没有对数据源进行配置，所以就会报错。

**解决办法：application.properties配置如下**

```properties
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/ssmrl?serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=18420163207 
```

------



### SpringBoot运行原理

#### **POM.XML**

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```

SpringBoot有父依赖。

**我们点进去spring-boot-starter-parent看看。**

![image-20210109160042561](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109160042561.png)


**springBoot里面自带了很多依赖，这些依赖都在spring-boot-dependencies里面。**

我们截取了一段代码。

![image-20210109160422815](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109160422815.png)


**说明SpringBoot自带了很多依赖，和控制了这些依赖的version（版本）==》spring-boot-dependencies是版本控制中心**



**springBoot的启动器**

![image-20210109161653710](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109161653710.png)



我们点进去一个看看。

![image-20210109162019249](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109162019249.png)



![image-20210109162243797](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109162243797.png)



**结论：可以看出来，很多我们在WEB开发需要用的，SpringBoot都给我们封装好了，变成了一个个starter（启动器），简化了开发。也就是说启动器里面就是我们要用的依赖**。

------



#### SpringBoot的主程序

![image-20210109163611654](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109163611654.png)


![image-20210109163828860](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109163828860.png)



**上面短短的几句代码就可以把SpringBoot项目运行起来。说明里面的原理是很复杂的。**

##### SpringBoot主程序注解

**点开@SpringBootApplication：**

![image-20210109164119351](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109164119351.png)



SpringBoot底层运用了大量的Spring底层注解。

**@SpringBootConfiguration**：说明这个类是SpringBoot的配置类

**@EnableAutoConfiguration：开启自动配置功能。SpringBoot最核心的功能就是自动配置。大大的简化了开发，所以这个注解是非常重要的**

**@ComponentScan：**Spring的注解，也就是去扫描这些类，并添加到SpringIOC容器中。



进去**@SpringBootConfiguration注解里面：**

![image-20210109165129494](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109165129494.png)


![image-20210109165231322](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109165231322.png)


因为@SpringBootConfiguration里面有@Configuration注解，@Configuration里面又有@Component注解。

**说明@SpringBootConfiguration是以一个Spring组件添加进来的。**



点进去**@EnableAutoConfiguration**注解：

![image-20210109165552449](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109165552449.png)



**我们可以看到两个注解：@AutoConfigurationPackage和@Import({AutoConfigurationImportSelector.class})**

**@AutoConfigurationPackage：自动配置包**

在@AutoConfigurationPackage里面有如下代码：

```java
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};
}
```

**Registrar.class：作用是将springBoot主程序类所在的包和所在包的子包，也就是目前的“boot”目录下所有类进行扫描，并加载到SpringIOC容器中，所以也就是为什么在boot外面的Java代码会没有作用，正因为springBoot在自动配置包注解中，默认只会扫描主程序类所在的包和所在包的子包的类**





**@Import({AutoConfigurationImportSelector.class})：导入自动配置导入选择器类**

在**AutoConfigurationImportSelector**类中，有如下代码：

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
```



**作用是得到候选配置**

**点进去 SpringFactoriesLoader.loadFactoryNames（）方法：**

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
        ClassLoader classLoaderToUse = classLoader;
        if (classLoader == null) {
            classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
        }

        String factoryTypeName = factoryType.getName();
        return (List)loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
    }
```



**我们在点进(List)loadSpringFactories(classLoaderToUse)的loadSpringFactories方法中**

```java
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
        Map<String, List<String>> result = (Map)cache.get(classLoader);
        if (result != null) {
            return result;
        } else {
            HashMap result = new HashMap();

            try {
                Enumeration urls = classLoader.getResources("META-INF/spring.factories");

                while(urls.hasMoreElements()) {
                    URL url = (URL)urls.nextElement();
                    UrlResource resource = new UrlResource(url);
                    Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                    Iterator var6 = properties.entrySet().iterator();

                    while(var6.hasNext()) {
                        Entry<?, ?> entry = (Entry)var6.next();
                        String factoryTypeName = ((String)entry.getKey()).trim();
                        String[] factoryImplementationNames = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                        String[] var10 = factoryImplementationNames;
                        int var11 = factoryImplementationNames.length;

                        for(int var12 = 0; var12 < var11; ++var12) {
                            String factoryImplementationName = var10[var12];
                            ((List)result.computeIfAbsent(factoryTypeName, (key) -> {
                                return new ArrayList();
                            })).add(factoryImplementationName.trim());
                        }
                    }
                }

                result.replaceAll((factoryType, implementations) -> {
                    return (List)implementations.stream().distinct().collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList));
                });
                cache.put(classLoader, result);
                return result;
            } catch (IOException var14) {
                throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var14);
            }
        }
    }
```



**我们可以看到它频繁的出现META-INF/spring.factories。我们去搜索它**



![image-20210109171956534](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109171956534.png)


![image-20210109172121879](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210109172121879.png)



**在这里我们可以看到非常多的配置信息。这就是SpringBoot自动配置的根源所在，在SpringBoot运行的时候，自动配置类会在类路径下的**META-INF/spring.factories里面去找到对应的值，只有导入了这些对应的值，自动配置才能生效



##### SpringBoot主程序的Run方法：

**我们点进去run()，找到SpringApplication的构造器。**

```java
 public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
         .............
         .............  #上面省略
        this.webApplicationType = WebApplicationType.deduceFromClasspath();
  this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
        this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
        this.mainApplicationClass = this.deduceMainApplicationClass();
    }
```



**SpringApplication**

**结论：SpringApplication这个类会做如下的事：**

1. **先去推断这个项目是不是WEB项目**
2. **从SpringFactories实例中查找出所有初始化器，并设置到initializers属性中**
3. **从SpringFactories实例中查找出所有应用监听器，并设置到listeners属性中**
4. **推断SpringBoot主程序类，并设置到mainApplicationClass中**

****

### yml配置注入

**SpringBoot自带了application.properties，但是呢，SpringBoot更加推荐用yml或者yaml，不过本质上其实是差不多的，只是语法有些许不同罢了，yml和yaml会更加简洁**

```yaml
#yml语法：key:空格 值
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/ssmrl?serverTimezone=UTC
    username: root
    password: 18420163207
```

```properties
#properties语法：key=值
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```





**问：我们如何用SpringBoot自带的配置文件对一个对象进行封装，以便我们用@Autowired对这个类进行注入**

![image-20210110161513385](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210110161513385.png)



**@ConfigurationProperties(prefix="前缀")：把这个类的对象交给SpringBoot配置文件，我们可以在配置文件中用  前缀.属性名去赋值，这样SpringBoot就会把这些属性值封装成一个‘’对象‘’，放在IOC容器里，这样我们通过自动装配就可以获得这个对象 **



**如图上所示，报了一个错误====Not registered via @EnableConfigurationProperties, marked as Spring component, or scanned via @ConfigurationPropertiesScan **

**意思是：我们少了一个@EnableConfigurationProperties注解，我们必须开启这个注解，@ConfigurationProperties才会生效**



**解决方法：**

**1.**

```xml
        <!-- 1.在Pom.xml上导入这个依赖-->
		<dependency>   
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

**2.**

```java
@EnableConfigurationProperties(emp.class) //必须要加上这个注解，并指定类
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```



进入application.yml：

![image-20210110162742344](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210110162742344.png)



**======说明我们已经通过@ConfigurationProperties绑定到这个类了**

去绑定一下：

```yaml
myemp:
  id: 999
  name: springBoot
```



```java
@SpringBootTest
class DemoApplicationTests {

    @Autowired
    private emp emp;

    @Test
    void contextLoads() {

        System.out.println(emp);

    }

}
```

![image-20210110163028079](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210110163028079.png)


**====然后我们就绑定成功了！！！**

------

### 多环境切换

**在实际的开发中，我们可能会需要有多种环境，比如开发环境、测试环境、真实环境，我们如何做到这一点呢？**



**在resources目录下创建application-dev.properties**

此时这个环境名就叫做：**dev**

**springBoot默认会读取application.properties而不是application-dev.properties，所以我们要切换环境只能如下操作：**



**application.properties**

```properties
server.port=8080   #设置该环境服务器的端口号
spring.profiles.active=dev  #环境切换成dev
```

**application-dev.properties**

```properties
server.port=8081
```

**当我们去启动SpringBoot项目，下面有一段日志写着：**



![image-20210110171903820](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210110171903820.png)





**在application.properties我们配置的端口号是8080，在application-dev.properties我们配置的端口号是8081，但是启动时我们发现初始化端口号是8081，说明我们已经顺利的切换了环境。**

------

### SpringBoot自动装配原理（不懂）



```java
@EnableAutoConfiguration  //开启自动配置功能
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
 ....   
}

```



**我们进去@EnableAutoConfiguration:**

```java
@Import({AutoConfigurationImportSelector.class}) //将指定的类导入到容器中
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}

```



**我们再点进去AutoConfigurationImportSelector**

找到如下：

```java
 protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
```

进入loadFactoryNames（）

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
        ClassLoader classLoaderToUse = classLoader;
        if (classLoader == null) {
            classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
        }

        String factoryTypeName = factoryType.getName();
        return (List)loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
    }
```

再进入(List)loadSpringFactories(classLoaderToUse)

```java
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
        Map<String, List<String>> result = (Map)cache.get(classLoader);
        if (result != null) {
            return result;
        } else {
            HashMap result = new HashMap();

            try {
                //1.获取所有META-INF/spring.factories
                Enumeration urls = classLoader.getResources("META-INF/spring.factories");

                while(urls.hasMoreElements()) {
                    URL url = (URL)urls.nextElement();
                   
                    UrlResource resource = new UrlResource(url);
                    //2.把所有META-INF/spring.factories封装成properties
                    Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                    Iterator var6 = properties.entrySet().iterator();

                    while(var6.hasNext()) {
                        Entry<?, ?> entry = (Entry)var6.next();
                        String factoryTypeName = ((String)entry.getKey()).trim();
                        String[] factoryImplementationNames = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                        String[] var10 = factoryImplementationNames;
                        int var11 = factoryImplementationNames.length;

                        for(int var12 = 0; var12 < var11; ++var12) {
                            String factoryImplementationName = var10[var12];
                            ((List)result.computeIfAbsent(factoryTypeName, (key) -> {
                                return new ArrayList();
                            })).add(factoryImplementationName.trim());
                        }
                    }
                }

                result.replaceAll((factoryType, implementations) -> {
                    return (List)implementations.stream().distinct().collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList));
                });
                cache.put(classLoader, result);
                return result;
            } catch (IOException var14) {
                throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var14);
            }
        }
    }
```



**结论：**

1. **当springBoot启动时，会去搜索所有/META-INF/spring.factories**
2. ![image-20210115165407600](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210115165407600.png)


**并把所有EnableAutoConfiguration的值导入到容器中，然后自动配置才会生效**

**xxxAutoConfiguration会和xxxProperties绑定在一起，xxxAutoConfiguration需要的值会在xxxProperties里面取，xxxProperties的默认值可以通过application.properties来设置**

 

------



### 静态资源处理



**SpringBoot项目自带的静态资源目录**

![image-20210111225839932](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210111225839932.png)





**注意：templates目录只用来存放html页面。**



#### 欢迎页

**我们先打开WebMvcAutoConfiguration，会看到如下代码**

```java
  @Bean
        public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext, FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
            WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
            welcomePageHandlerMapping.setInterceptors(this.getInterceptors(mvcConversionService, mvcResourceUrlProvider));
            welcomePageHandlerMapping.setCorsConfigurations(this.getCorsConfigurations());
            return welcomePageHandlerMapping;
        }
```

**在 WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());代码中有一句：this.getWelcomePage()，我们进入这个方法看看，看它是如何得到欢迎页的**

 

**进入后代码如下：**

```java
 private Optional<Resource> getWelcomePage() {
            String[] locations = WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations());
            return Arrays.stream(locations).map(this::getIndexHtml).filter(this::isReadable).findFirst();
        }
   .......
       .....
private Resource getIndexHtml(String location) {
            return this.resourceLoader.getResource(location + "index.html");
        }  //欢迎页就是location下面的index.html而已
```



**再进入this.resourceProperties.getStaticLocations()方法**

```java
public String[] getStaticLocations() {
            return this.staticLocations;
        }

```

**点进this.staticLocations之后我们会发现有如下代码：**

```java
  public static class Resources {
        private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
        private String[] staticLocations;
        private boolean addMappings;
        private boolean customized;
        private final WebProperties.Resources.Chain chain;
        private final WebProperties.Resources.Cache cache;

        public Resources() {
            this.staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
            this.addMappings = true;
            this.customized = false;
            this.chain = new WebProperties.Resources.Chain();
            this.cache = new WebProperties.Resources.Cache();
        }
```



**这就是SpringBoot对静态资源的处理**

**这段代码 private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};说明了SpringBoot只去认这些路径下面的静态资源，其他路径的静态资源是无效的**

**回到上一步，我们进入WelcomePageHandlerMapping这个类，会有如下代码：**



```java
WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders, ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
        if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
            logger.info("Adding welcome page: " + welcomePage.get());
            this.setRootViewName("forward:index.html");
        } else if (this.welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
            logger.info("Adding welcome page template: index");
            this.setRootViewName("index");
        }

    }
```



**上面我们说了，所有在"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"路径下的静态资源都会被SpringBoot扫描，上面的代码可以看出SpringBoot会扫描这些路径下的index.html，作为欢迎页**

![image-20210111235539517](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210111235539517.png)


#### 静态资源处理的两种方式

**在WebMvcAutoConfiguration里面有一段代码，里面写着怎么处理静态资源**

```java
 public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
                CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
                if (!registry.hasMappingForPattern("/webjars/**")) { 
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl).setUseLastModified(this.resourceProperties.getCache().isUseLastModified()));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl).setUseLastModified(this.resourceProperties.getCache().isUseLastModified()));
                }

            }
        }
```

1. ```java
    方式一：   this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl).setUseLastModified(this.resourceProperties.getCache().isUseLastModified()));   //用webjars方式
   ```

2. ```java
      this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl).setUseLastModified(this.resourceProperties.getCache().isUseLastModified()));
                   }
   ```

   在（2）里面有一个**this.resourceProperties.getStaticLocations()，进入我们可以看到**

```java
 public static class Resources {
        private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
        private String[] staticLocations;
        private boolean addMappings;
        private boolean customized;
        private final WebProperties.Resources.Chain chain;
        private final WebProperties.Resources.Cache cache;

        public Resources() {
            this.staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
            this.addMappings = true;
            this.customized = false;
            this.chain = new WebProperties.Resources.Chain();
            this.cache = new WebProperties.Resources.Cache();
        }
```

```java
  方式二：  private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};  //只要资源文件在这里面的目录，就可以被扫描到
```

------

### SpringBoot+JDBC

**1.导入JDBC和mysql的依赖**

在Pom.xml导入以下依赖

```xml
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

```



**2.在application的配置文件配置数据源（DataSource）**

```properties
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/ssmrl?serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=18420163207
```



**3.测试数据源是否生效**

**在SpringBoot单元测试进行**

```java
@SpringBootTest
class DemoApplicationTests {
    @Autowired    //自动注入数据源，因为我们已经配置过了（application.properties），它会在IOC容器去找到数据源
    private  DataSource dataSource;

    @Test
    void contextLoads() {
        System.out.println(dataSource);
    }

}
```

**运行结果如下：**

![image-20210113161631607](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210113161631607.png)



**结论：我们从输出结果可以看到，SpringBoot默认的数据源是HikariDataSource，这个数据源是目前最快的数据源，比阿里巴巴的Druid还要快，但是DruidDatasource有监控功能，后面再说**







#### SpringBoot原生JDBC



**1.在SpringBoot单元测试类中：**



```java
@SpringBootTest
class DemoApplicationTests {
    @Autowired
    private  DataSource dataSource;

    @Test
    void contextLoads() throws SQLException {

        Connection connection = dataSource.getConnection();
        String sql="select empid,empName from emp";
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        ResultSet resultSet = preparedStatement.executeQuery();
        while (resultSet.next()){
            System.out.println(resultSet.getInt(1)+"    "+resultSet.getString(2));
        }

    }

}
```



**输出结果如下：**


![image-20210113162723306](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210113162723306.png)




#### SpringBoot+JdbcTemplate

**JdbcTemplate是Spring对JDBC的封装，目的是让Jdbc更加容易使用**

![image-20210113164334271](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210113164334271.png)


**说明我们的JdbcTemplate可以正常使用了**

**里面的sql语句可以写预处理，也就是用“  ?  ”占位 **

**查询单个数据：**

![image-20210113165628283](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210113165628283.png)



**查询多个数据：**

![image-20210113165815772](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210113165815772.png)


------



### SpringBoot+Druid

**因为我们在DataSourceAutoConfiguration类中看到如下代码：**

```java
@Import({Hikari.class, Tomcat.class, Dbcp2.class, OracleUcp.class, Generic.class, DataSourceJmxConfiguration.class})
    protected static class PooledDataSourceConfiguration {
        protected PooledDataSourceConfiguration() {
        }
    }
```

**Hikari.class  导入了Hikari的数据源，SpringBoot默认自带了HikariDataSource，如何切换数据源呢？**

我们只需要在配置文件加上一句代码：



```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
```



#### 添加Druid数据源的监控功能



```java
public abstract class ResourceServlet extends HttpServlet {
    //Druid监控的帐号密码
    private static final Log LOG = LogFactory.getLog(ResourceServlet.class);
    public static final String SESSION_USER_KEY = "druid-user";
    public static final String PARAM_NAME_USERNAME = "loginUsername"; //配置帐号
    public static final String PARAM_NAME_PASSWORD = "loginPassword"; //配置密码
```



**因为我们在配置文件指定了spring.datasource.type=com.alibaba.druid.pool.DruidDataSource，会自动使用SpringBoot的默认配置，我们不想这样可以自己写一个Druid配置类，然后在附加上去**

```java
@Configuration
public class myDruid {

    //指定前缀为：spring.datasource
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean  //添加到IOC容器中去
    public DataSource druidDatasource(){
        DruidDataSource druidDataSource=new DruidDataSource();
        return druidDataSource;
    }

    //添加Druid监控，这个ServletRegisterBean相当于web.xml的<servlet></servlet>
    @Bean
    public ServletRegistrationBean servletRegistrationBean(){
        //指定servlet的类
        ServletRegistrationBean servletRegistrationBean=new ServletRegistrationBean(new StatViewServlet());
        //配置映射路径
        servletRegistrationBean.addUrlMappings("/druid/*");
        //配置帐号密码
        servletRegistrationBean.addInitParameter("loginUsername","admin");
        servletRegistrationBean.addInitParameter("loginPassword","123456");
        return servletRegistrationBean;
    }
}
```



**这样就完成啦**

![image-20210113172829360](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210113172829360.png)



**输入刚刚配置的帐号密码就可以进入**

![image-20210113172854913](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210113172854913.png)



### SpringBoot+Mybatis


![image-20210116152421022](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210116152421022.png)


![image-20210116152434429](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210116152434429.png)

![image-20210116152457152](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210116152457152.png)

![image-20210116152513332](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210116152513332.png)





**springBoot+Mybatis整合比较简单**

**@Mapper：作用在dao接口上，作用就是把这个接口转换成可以注入的实现类。（注意，此时只是转换成了实现类，并没有在IOC容器中，所以我们还要加上组件注解，如@Repository，不然会不起作用）**

**也可以在springBoot启动类加上：@MapperScan：扫描一个包上的Mapper接口，并将自动转换成实现类，这时候我们在加上一个组件注解，然后@AutoWired就可以了。**



**具体的SpringBoot+Mybatis原理还未掌握。。。**





------



### **SpringBoot+Mybtis绑定异常解决**

**绑定异常有很多种，当我们Mapper和Mapper.xml都绑定起来了，但是还报错。**

![image-20210128114800015](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210128114800015.png)





**解决方法：在POM.xml加上Mybatis的文件过滤：**

```xml
         <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
```











###  SpringBoot异步任务

**使用场景：比如我们发送验证码，从前端发送请求到后端是要等待的（网络通信是有延迟的），比如我们发送成功一个验证码需要3s，如果不是异步操作，我们就处于“单线程“的状态，会在3s内都无法在页面进行任何操作，这样用户体验感会很差，如果我们是异步任务，就相当于“多线程”，会单独的开启一个线程去发送验证码，另外一个线程以供用户操作页面，这样用户的体验感会更好。**

 

**@Async：标注这个类或者方法是异步的，标注之后就相当于另外开启了一个线程去处理这个@Async方法**

**@EnableAsync：开启异步任务功能。（必须要有）**



```java
@SpringBootApplication
@EnableAsync  //开启异步任务功能。
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

 



**Async的方法要放在另外的类上，不然会导致异步功能失效**

 **错误示范：**


![image-20210117230956632](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210117230956632.png)



**发现上面的写法，@Async不起作用。**



**正确写法：**

![image-20210117231249701](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210117231249701.png)


![image-20210117231338074](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210117231338074.png)



**这样就生效了，所以结论是，@Async异步任务需要单独放在其他类上。。。。。（很重要）**





### SpringBoot定时任务

**主要是运用cron表达式，进行定时的操作**

```java
public class MySend {
    //每两秒执行一次。
    @Scheduled(cron = "0/2 * * * * ? ")
    public void send(){
         System.out.println("hello");
    }

}
```



```java
@SpringBootApplication
@EnableScheduling //开启定时任务，定时任务是cron表达式
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

 

 

### SpringBoot邮件任务

**SpringBoot把原生java发送邮件的代码进行进一步的封装，使得原来十分繁琐的过程变得极其好用。**

**1.在Pom.xml导入对应的邮件启动器**

````xml
<!--        springBoot邮件启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
````



**2.开启qq邮箱的SMTP服务**

![image-20210118174756651](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210118174756651.png)





**3.获取SMTP授权码**

![image-20210118174701564](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210118174701564.png)



**4.配置application-dev.properties**

```properties
#发送邮箱配置
spring.mail.host=smtp.qq.com
spring.mail.username=1550324080@qq.com #邮箱号
spring.mail.password=zgwjpsscpsidicce   #授权码（不是密码）
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
```



**发送简单邮件**

**5.用测试类去测试一下**

```java
@SpringBootTest
class DemoApplicationTests {
    @Autowired
    JavaMailSenderImpl javaMailSender; //邮件发送必须要的类
    @Test
    void contextLoads(){
        //发送简单邮件用SimpleMailMessage类
        SimpleMailMessage simpleMailMessage=new SimpleMailMessage();
        simpleMailMessage.setFrom("1550324080@qq.com"); //从哪个邮箱发送
        simpleMailMessage.setTo("1550324080@qq.com");  //发送到哪个邮箱
        simpleMailMessage.setSentDate(new Date()); //日期
        simpleMailMessage.setText("hello");  //内容文本
        simpleMailMessage.setSubject("主题"); //题目
        javaMailSender.send(simpleMailMessage); //开启发送邮件。。需要时间去发送，这里我们可以用异步方法，提升体验感
    }
}
```



**发现报错了。。。**

![image-20210118180513820](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210118180513820.png)





**成功了！！！！**

![image-20210118181901117](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210118181901117.png)





**这里有个要注意的点，就是刚刚为什么会报错。**

**当我们自动装配 JavaMailSenderImpl   javaMailSender;会显示无法找到他对应的bean。。。导致刚刚报错**

**因为我们邮箱的配置必须要在application.properties里面配置，而不能在application-dev.properties配置**


![image-20210118182343270](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210118182343270.png)



**发送复杂邮件**



```java
@SpringBootTest
class DemoApplicationTests {

    @Autowired
    JavaMailSenderImpl javaMailSender;

    @Test
    void contextLoads() throws MessagingException, FileNotFoundException {
        MimeMessage mimeMessage = javaMailSender.createMimeMessage(); //创建复杂邮件
        //*****需要上传附件，就要在MimeMessageHelper的构造器加上true，不然会报错****
        MimeMessageHelper mimeMessageHelper=new MimeMessageHelper(mimeMessage,true); //创建MimeMessageHelper去封装MimeMessage
        mimeMessageHelper.setFrom("1550324080@qq.com");
        mimeMessageHelper.setTo("1550324080@qq.com");
        mimeMessageHelper.setSubject("主题");
        mimeMessageHelper.setText("<h3>h3字体</h3><br/><h5>h5字体</h5>");
//        FileSystemResource fileSystemResource = new FileSystemResource(new File("C:\\Users\\youzhengjie666\\Pictures\\1.PNG"));

       //上传附件
        mimeMessageHelper.addAttachment("tupian1",new File("C:\\Users\\youzhengjie666\\Pictures\\1.PNG"));

        javaMailSender.send(mimeMessage);


    }

}
```







### SpringBoot+Shiro安全框架

**1.先导入Shiro-spring的依赖**

```xml
<!--        shiro-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.2.2</version>
        </dependency>
```

**2.创建一个类，去配置shiro**

```java
package com.boot.shiro;

import com.boot.realm.myRealm;
import org.apache.shiro.authc.credential.HashedCredentialsMatcher;
import org.apache.shiro.realm.Realm;
import org.apache.shiro.spring.LifecycleBeanPostProcessor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.filter.DelegatingFilterProxy;

import javax.servlet.Servlet;
import java.util.ArrayList;
import java.util.LinkedHashMap;

@Configuration
public class myShiro {

    //配置加密
    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher(){
        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
        hashedCredentialsMatcher.setHashAlgorithmName("MD5");
        return hashedCredentialsMatcher;
    }


    @Bean
    public Realm realm(@Qualifier("hashedCredentialsMatcher") HashedCredentialsMatcher hashedCredentialsMatcher){
        myRealm myRealm=new myRealm();
        myRealm.setCredentialsMatcher(hashedCredentialsMatcher);
        return myRealm;
    }


    @Bean
    public DefaultWebSecurityManager defaultWebSecurityManager(@Qualifier("realm") Realm realm){
        DefaultWebSecurityManager defaultWebSecurityManager=new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(realm);
        return defaultWebSecurityManager;
    }


    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor(){
        LifecycleBeanPostProcessor lifecycleBeanPostProcessor = new LifecycleBeanPostProcessor();
        return lifecycleBeanPostProcessor;
    }

    @Bean("shiroFilter")
    public ShiroFilterFactoryBean setshiroFilter(@Qualifier("defaultWebSecurityManager") DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean=new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(defaultWebSecurityManager);
        shiroFilterFactoryBean.setLoginUrl("/tologin");
        shiroFilterFactoryBean.setSuccessUrl("/tolist");
        shiroFilterFactoryBean.setUnauthorizedUrl("/tounauth");

        LinkedHashMap<String, String> filters = new LinkedHashMap<>();
        filters.put("/tologin","anon");
        filters.put("/login","anon");
        filters.put("/toadmin","roles[admin]");
        filters.put("/touser","roles[user]");
        filters.put("/**","authc");


        shiroFilterFactoryBean.setFilterChainDefinitionMap(filters);

        return shiroFilterFactoryBean;
    }


//    @Bean
//    public FilterRegistrationBean shiroFilter(){
//        FilterRegistrationBean filterRegistrationBean=new FilterRegistrationBean();
//        filterRegistrationBean.setFilter(new DelegatingFilterProxy());
//        ArrayList<Object> list = new ArrayList<>();
//        list.add("/**");
//        filterRegistrationBean.setUrlPatterns(list);
//
//        return filterRegistrationBean;
//    }

 


}

```



**3.创建一个类，配置realm**

```java
package com.boot.realm;

import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;

import java.util.HashSet;
import java.util.Set;

public class myRealm extends AuthorizingRealm {


    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        //模仿从后台查询数据库
        String us="123";
        String pd="202cb962ac59075b964b07152d234b70"; //原密码是：123  ，这是MD5加密而成的

        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(us,pd,this.getName());

        return info;
    }

       @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        //获取principal===>也就是我们传入的帐号
        String principal = (String) principalCollection.getPrimaryPrincipal();

        Set<String> roles=new HashSet<>();
        roles.add("admin");
        SimpleAuthorizationInfo simpleAuthorizationInfo=new SimpleAuthorizationInfo(roles);

        return simpleAuthorizationInfo;
    }


}

```



**4.配置controller层**

```java
@Controller
public class testController {

    @RequestMapping(path = "/tologin")
    public String tologin(){
        return "login";
    }


    @RequestMapping(path = "/login")
    public String login(String username,String password){

        Subject subject = SecurityUtils.getSubject();

        if(!subject.isAuthenticated()){
            UsernamePasswordToken usernamePasswordToken=new UsernamePasswordToken(username,password);

            try {
                subject.login(usernamePasswordToken);
            }catch (Exception e){
                return "redirect:tologin";
            }

        }

        return "forward:tolist";
    }



    @RequestMapping(path = "/tolist")
    public String tolist(){
        return "list";
    }
    @RequestMapping(path = "/tounauth")
    public String tounauth(){
        return "unauth";
    }

    @RequestMapping(path = "/toadmin")
    public String toadmin(){
        return "role_admin";
    }

    @RequestMapping(path = "/touser")
    public String touser(){
        return "role_user";
    }

}
```



**5.需要的html文件**

![image-20210122143848238](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210122143848238.png)





### 扩展MVC

**扩展SpringMVC只需要去实现WebMvcConfigurer类就可以了，并且这个类是配置类**

**如果加了@EnableWebMVC注解，就相当于全面接管SpringMVC，也就是会把SpringBoot默认配置全部失效**





```java
@Configuration
public class myMvcConfig implements WebMvcConfigurer {

    //添加视图跳转
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {

        //这个代码相当于==>
        //@RequestMapping("/test")
        //public String xxx{
        // return "role_admin";
        //}
        registry.addViewController("/test").setViewName("role_admin");

    }

 
}
```













### 页面国际化

![image-20210127145329346](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210127145329346.png)


**创建xxx.properties作为默认国际化配置文件，xxx_zh_CN.properties作为中文配置，xxx_en_US.properties作为英文配置，这个Resource Bundle 'login'是会自动生成的**



**进入login.properties**

![image-20210127145551970](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210127145551970.png)



**左上角有个“+”号,就是用来添加国家化消息的**



**在application-dev.properties指定国家化消息的basename，因为我们自定义了xxx.properties的xxx是login，所以login就是我们的总的国家化配置文件，只需要绑定这个login就行了**

![image-20210127145922680](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210127145922680.png)


```properties
spring.messages.basename=i18n.login //i18n就是我们用来存放国家化配置文件的包名
```



**login.html**

```html
<form th:action="@{/login}" th:method="post">
    <span th:text="#{username}"></span>：<input type="text" name="username">
    <br/>
    <span th:text="#{password}"></span>:<input type="password" name="password">
    <br/>
    <input type="submit" th:value="#{submit}">
</form>


<a th:href="@{/tologin(locale='zh_CN')}">中文</a>
&nbsp;&nbsp;&nbsp;&nbsp;
<a th:href="@{/tologin(locale='en_US')}">English</a>
```



**注意：thymeleaf的#{}就是用来取国家化消息的**



**此时我们想进行中英文切换。我们需要做如下配置：**

**1.在login.html中传locale**

```html
<a th:href="@{/tologin(locale='zh_CN')}">中文</a>
&nbsp;&nbsp;&nbsp;&nbsp;
<a th:href="@{/tologin(locale='en_US')}">English</a>
```



**2.配置自定义的localeResolver**

```java
public class myLocale implements LocaleResolver {
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        Locale aDefault = Locale.getDefault();//获取默认的Locale
        String locale = request.getParameter("locale"); //获取login.html传来的语言
        if(!StringUtils.isEmpty(locale)){
            //zh_CN 要变成====>Locale("zh","CN")，只有把"_"分割成zh和CN
            String language=locale.charAt(0)+""+locale.charAt(1);
            String country=locale.charAt(3)+""+locale.charAt(4);
            return new Locale(language, country);
        }

        return aDefault;
    }

    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {

    }
}

```



**3.将这个自定义LocaleResolver交给Spring管理**

**注意：这个自定义地区解析器的Bean的id必须是“  localeResolver ”，不然会找不到这个Bean**

```java
@Configuration
public class myLocaleConfig {

    @Bean("localeResolver") 
    public myLocale localeResolver(){

        return new myLocale();
    }


}
```


**然后就大功告成了**


## Spring Cloud Netflix



### 搭建提供者、消费者模块

**1：创建一个空Maven项目，删除`src`目录**


![image-20210220134218449](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210220134218449.png)



**2：父POM**

```xml
 <!--    修改打包方式为pom-->
    <packaging>pom</packaging>

    <properties>
        <!--        springcloud的版本-->
        <spring.cloud-version>Hoxton.RELEASE</spring.cloud-version>
<!--        springBoot的版本-->
        <spring.boot-version>2.2.4.RELEASE</spring.boot-version>

<!--        防止idea发疯，编译一直都是java5-->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <maven.compiler.source>10</maven.compiler.source>
        <maven.compiler.target>10</maven.compiler.target>
        <encoding>UTF-8</encoding>



    </properties>

    <!--    dependencyManagement是用定义依赖的，并没有直接引入依赖，作用是控制父子模块的依赖版本一致性问题,
    子模块通过不加version，version会自动的去父pom找到定义好的依赖版本，这样兼容性就会大大提升
    -->

    <dependencyManagement>
        <!--  springcloud总依赖-->
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud-version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--springBoot总依赖-->
            <dependency>

                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring.boot-version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

<!--            引入Mybatis-->
<!--            注意：一定要导入mybatis的springBoot启动器，不要直接导入mybatis依赖，不然会没效果-->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>1.3.1</version>
            </dependency>

            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.47</version>
            </dependency>

<!--            druid数据库连接池-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.0.9</version>
            </dependency>

        </dependencies>


    </dependencyManagement>


    <build>
<!--      mybatis的静态资源过滤，也就是过滤Mapper.xml，我们也可以这样想，xml文件是放在resources目录下的，放在java
目录下不会被扫描，所以我们要过滤java目录中的xml
-->
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```



**3.创建一个子模块，作微服务的提供者，端口号为8001**

![image-20210222004113174](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210222004113174.png)



**里面代码如下。。。**

**deptController**

```java
@RestController
public class deptController {

    private deptService deptService;

    @Autowired
    @Qualifier("deptServiceImpl")
    public void setDeptService(com.boot.service.deptService deptService) {
        this.deptService = deptService;
    }
    @GetMapping(path = "/queryAllDept")
    public List<dept> queryAllDept(){


        return deptService.queryAllDept();
    }



}
```

**dao层。deptMapper**

```java
@Mapper //把这个Mapper接口变成可以注入的Bean,*****一定要。
@Repository  //变成组件  ***一定要
public interface deptMapper {

    List<dept> queryAllDept();
 
}
```



**Mapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">


<mapper namespace="com.boot.dao.deptMapper">

    <select id="queryAllDept" resultType="com.pojo.dept">
        select deptid,deptName from dept;
    </select>
 
</mapper>
```



**service层省略，和普通springBoot项目构建是一样的**

**application.yml(8001)**



```yaml
server:
  port: 8001

spring:
  application:
#    微服务名
    name: provider_dept8001/8002
  datasource:
    url: jdbc:mysql://localhost:3306/ssmrl?serverTimezone=UTC
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: 18420163207
    type: com.alibaba.druid.pool.DruidDataSource
```



**创建一个子模块(springcloud-02-api)，专门放实体类**

![image-20210222011216901](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210222011216901.png)


```java
public class dept implements Serializable {

    private String deptid;
    private String deptName;

    public dept() {
    }

    public dept(String deptid, String deptName) {
        this.deptid = deptid;
        this.deptName = deptName;
    }

    public String getDeptid() {
        return deptid;
    }

    public void setDeptid(String deptid) {
        this.deptid = deptid;
    }

    public String getDeptName() {
        return deptName;
    }

    public void setDeptName(String deptName) {
        this.deptName = deptName;
    }

    @Override
    public String toString() {
        return "dept{" +
                "deptid='" + deptid + '\'' +
                ", deptName='" + deptName + '\'' +
                '}';
    }
}

```





**创建另外一个子模块，springcloud-02-provider-dept8002**

**把springcloud-02-provider-dept8001代码全部复制上去，修改配置文件application.yml(8002)**

```yaml
server:
  port: 8002

spring:
  application:
#    微服务名
    name: provider_dept8001/8002
  datasource:
    url: jdbc:mysql://localhost:3306/ssmrl?serverTimezone=UTC
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: 18420163207
    type: com.alibaba.druid.pool.DruidDataSource
```



**为什么我们要复制多一份微服务提供者代码？**

**因为考虑到后面我们要使用负载均衡ribbon或者openFeign（不过底层也是ribbon）**



**4.创建子模块springcloud-02-comsumer-dept80**

**controller层：**

```java
@RestController
public class deptController80 {

    @Autowired
    private RestTemplate restTemplate;
    private final String URL_DEPT="http://localhost:8001/";


    @GetMapping("/comsumer/queryAllDept")
    public List<dept> queryAllDept80(){
        List<dept> res = restTemplate.getForObject(URL_DEPT + "queryAllDept", List.class);

        return res;
    }

}
```

**因为默认的RestTemplate没有放入IOC容器中（也就是没有Bean），我们需要手动的放入IOC容器**

**config层**

```java
@Configuration
public class restTemplateConfig {

    @Bean
    public RestTemplate restTemplate(){

        return new RestTemplate();
    }

}

```



**application.yml(80)**

```yaml
server:
  port: 80

#
spring:
  application:
    name: comsumer_dept80
```



**为什么微服务消费者层的端口是80，因为80端口可以省略不写，就比如我们打开百度，也是不需要写端口号的，因为微服务消费者层是给用户去访问的**



**消费者层只需要Controller去远程调用提供者的Controller方法即可，所以消费者层---不能---有dao，service层**



**这样提供者和消费者就搭建好了，接下来我们可以引入SpringCloud组件了。。。。。。。**



------

### 引入注册中心SpringCloud Eureka



**配置如下：**

![image-20210222203000271](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210222203000271.png)



**创建子模块springcloud-02-eureka7001(eureka注册中心服务端)**

**pom.xml**

```xml
 <dependencies>

        <!--      web和actuator是必备的，******除了gateway网关不能加入web包  -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--        actuator用来监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>


<!--        eureka Server-->
          <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>





    </dependencies>
```

**eureka模块主启动类**

```java
@SpringBootApplication
@EnableEurekaServer //开启Eureka服务器
public class SpringBootApplication7001 {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootApplication7001.class,args);
    }

}
```

**application.yml(7001)**

```yaml
server:
  port: 7001


#  配置eureka
eureka:
  instance:
    hostname: eureka-server7001.com
  client:
    register-with-eureka: false
    fetchRegistry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:7001/eureka/
```



**给提供者springcloud-02-provider-dept8001和8002修改如下**

```java
@SpringBootApplication
@EnableEurekaClient //eureka客户端
public class SpringBootApplication8001 {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootApplication8001.class,args);


    }

}
```

```yaml
server:
  port: 8001

spring:
  application:
#    微服务名
    name: provider_dept  
  datasource:
    url: jdbc:mysql://localhost:3306/ssmrl?serverTimezone=UTC
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: 18420163207
    type: com.alibaba.druid.pool.DruidDataSource


#    eureka Client
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server7001.com:7001/eureka/
    register-with-eureka: true
    fetch-registry: false
  instance:
    instance-id: eureka-client8001
```



```yaml
server:
  port: 8002

spring:
  application:
#    微服务名
    name: provider_dept
  datasource:
    url: jdbc:mysql://localhost:3306/ssmrl?serverTimezone=UTC
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: 18420163207
    type: com.alibaba.druid.pool.DruidDataSource


#   eureka
eureka:
  client:
    fetch-registry: false
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka-server7001.com:7001/eureka/
  instance:
    prefer-ip-address: true
    instance-id: eureka-client8002
```



##### Bug：引入Eureka后报错。

![image-20210222160445890](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210222160445890.png)



**。。。取名要规范。。。**



### 搭建Eureka集群

**创建springcloud-02-eureka7002和7003两个模块**

**application.yml（7001）**

```yaml
server:
  port: 7001


#  配置eureka
eureka:
  instance:
    hostname: eureka-server7001.com
  client:
    register-with-eureka: false
    fetchRegistry: false
    service-url:
      defaultZone: http://eureka-server7002.com:7002/eureka/,http://eureka-server7003.com:7003/eureka/   #集群版就修改这个。单机认自己，集群认其他
      
      
```

**application.yml（7002）**

```yaml
server:
  port: 7002

spring:
  application:
    name: eureka7002   #这里可有可无，除了微服务提供者。

eureka:
  instance:
    hostname: eureka-server7002.com
  client:
    fetch-registry: false
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka-server7001.com:7001/eureka/,http://eureka-server7003.com:7003/eureka/
```



**application.yml（7003）**

```yaml
server:
  port: 7003

spring:
  application:
    name: eureka7003

eureka:
  instance:
    hostname: eureka-server7003.com
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://eureka-server7001.com:7001/eureka/,http://eureka-server7002.com:7002/eureka/
```



**其实Eureka集群没啥变化，也就是修改了serviceUrl的defaultZone罢了。记住一句话，单机版认自己，集群版认其他**



### 搭建提供者集群(为了负载均衡)

**在8001和8002微服务中修改defaultZone。**

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://eureka-server7001.com:7001/eureka/, http://eureka-server7002.com:7002/eureka/, http://eureka-server7003.com:7003/eureka/
```

**还有8001和8002的微服务名要一致（spring.application.name）**

```yaml
spring:
  application:
#    微服务名
    name: provider-dept  #8001和8002要一致
```

**在springcloud-02-comsumer-dept80消费者层**

```java
@Configuration
public class restTemplateConfig {


    @Bean
    @LoadBalanced //开启负载均衡
    public RestTemplate restTemplate(){

        return new RestTemplate();
    }



}
```

```java
@SpringBootApplication
@EnableEurekaClient
public class SpringBootApplication80 {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootApplication80.class,args);
    }


}

```

**用Ribbon+RestTemplate员工调用提供者的Controller方法**

```java
@RestController
public class deptController80 {

    @Autowired
    private RestTemplate restTemplate;
    private final String URL_DEPT="http://PROVIDER-DEPT/"; //实现负载均衡用提供者微服务名代替IP:Port



    @GetMapping("/comsumer/queryAllDept")
    public List<dept> queryAllDept80(){
        List<dept> res = restTemplate.getForObject(URL_DEPT + "queryAllDept", List.class);

        return res;
    }


}

```





**application.yml(80)**

```yaml
server:
  port: 80

#
spring:
  application:
    name: comsumer-dept80
eureka:
  client:
    fetch-registry: true
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka-server7001.com:7001/eureka/,http://eureka-server7002.com:7002/eureka/,http://eureka-server7003.com:7003/eureka/


```









##### Bug：ribbon+restTemplate报错

![image-20210222232329911](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210222232329911.png)





**ribbon不支持微服务名有下划线（_）,修改过来即可**

```yaml
spring:
  application:
#    微服务名
    name: provider-dept
```





#### 使用actuator功能

```yaml
#暴露端点。使用actuator功能
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

![image-20210222233505528](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210222233505528.png)



### Feign/OpenFeign



**创建子模块springcloud-02-comsumer-openFeign-dept80**



**所需的依赖**

```xml
<!--        OpenFeign-->
       <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```



**点进去看看**

![image-20210224104302484](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210224104302484.png)



**我们可以发现，Feign/openFeign的底层就是Ribbon，所以openFeign自带了负载均衡的功能，相较于Ribbon+restTemplate，openFeign无需手动的使用@LoadBalanced注解来开启负载均衡，而Ribbon需要在restTemplate的Bean上加这个注解才有负载均衡的能力**



**主启动类**

```java
@SpringBootApplication
@EnableFeignClients //开启Feign的客户端支持
public class springApplicationFeign80 {

    public static void main(String[] args) {

        SpringApplication.run(springApplicationFeign80.class,args);

    }



}
```



**编写微服务接口**

```java
@Service
@FeignClient("PROVIDER-DEPT") //标注这个微服务接口是属于“PROVIDER-DEPT”这个微服务的
public interface deptService {
    
    //下面的代码直接从微服务提供者的controller复制过来即可
    @GetMapping(path = "/queryAllDept")
    public List<dept> queryAllDept();



}

```



**然后便是使用**

```java
@RestController
public class deptController {

    @Autowired
    private deptService deptService;


    @RequestMapping("/feign/queryAllDept")
    public List<dept> queryAllDept(){

        return deptService.queryAllDept();
    }



}

```



##### Bug：OpenFeign调用失败报错405

**错误类型：**

![image-20210227163134805](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210227163134805.png)



**原因是我们没有在提供者加上@PathVariable或者@RequestParam注解**

**解决方法一：在提供者加上@RequestParam注解（每个提供者传入的参数都要加上这个注解）**

```java
@RestController
public class deptController {

    private deptService deptService;

    @Value("${server.port}")
    private String port;


    @Autowired
    @Qualifier("deptServiceImpl")
    public void setDeptService(com.boot.service.deptService deptService) {
        this.deptService = deptService;
    }

    @GetMapping(path = "/queryAllDept")
    @HystrixCommand(fallbackMethod = "queryAllDept_Hystrix",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "20000"),//注意这是毫秒。1秒=1000毫秒
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "50")

    })
    public List<dept> queryAllDept(@RequestParam("id") String id){
        List<dept> depts = deptService.queryAllDept();
        depts.add(new dept("999",port));
        if(Integer.parseInt(id)<0){
            throw new RuntimeException();
        }

        return depts;
    }

    /**
     * 服务熔断
     */
    public List<dept> queryAllDept_Hystrix(@RequestParam("id") String id){

        List<dept> depts = deptService.queryAllDept();
        depts.add(new dept("1066","Break"));
        return depts;
    }

}

```



**修改openFeign的微服务接口**

```java
@Service
@FeignClient("PROVIDER-DEPT")
public interface deptService {
    @GetMapping(path = "/queryAllDept")
    public List<dept> queryAllDept(@RequestParam("id") String id);
}

```



**没有报错了！！！**

![image-20210227165930130](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210227165930130.png)


**解决方法二：在微服务提供者加上@PathVariable注解。。**

省略！！！



### 断路器springcloud Hystrix



#### 服务降级

**在springcloud-02-comsumer-openFeign-dept80的Pom.xml**

```xml
<!--        Hystrix-->
          <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

**修改它的deptController**

```java
@RestController
public class deptController {

    @Autowired
    private deptService deptService;


    @HystrixCommand(fallbackMethod = "queryAllDept_Hystrix",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1500") //添加超时服务降级
    })
    @RequestMapping("/feign/queryAllDept")
    public List<dept> queryAllDept(String id){
        int i = Integer.parseInt(id);
        if(i<0){
            throw new RuntimeException(); //此处为了触发服务降级fallback
        }
        return deptService.queryAllDept();
    }


    //服务降级方法，这个方法直接从上面的正常方法复制过来，把名字修改一下即可，参数类型也和正常的方法要一致
    public List<dept> queryAllDept_Hystrix(String id){

        List<dept> depts = deptService.queryAllDept();
        depts.add(new dept("1000","Hystrix_fallback"));
        return depts;
    }

}

```



**主启动类**

```java
@SpringBootApplication
@EnableFeignClients
@EnableHystrix //开启Hystrix功能
public class springApplicationFeign80 {

    public static void main(String[] args) {

        SpringApplication.run(springApplicationFeign80.class,args);

    }
}
```

**commandProperties支持什么参数属性**

**我们全局搜索HystrixCommandProperties，往下翻可以看到如下**

```java
protected HystrixCommandProperties(HystrixCommandKey key, HystrixCommandProperties.Setter builder, String propertyPrefix) {
        this.key = key;
        this.circuitBreakerEnabled = getProperty(propertyPrefix, key, "circuitBreaker.enabled", builder.getCircuitBreakerEnabled(), default_circuitBreakerEnabled);
        this.circuitBreakerRequestVolumeThreshold = getProperty(propertyPrefix, key, "circuitBreaker.requestVolumeThreshold", builder.getCircuitBreakerRequestVolumeThreshold(), default_circuitBreakerRequestVolumeThreshold);
        this.circuitBreakerSleepWindowInMilliseconds = getProperty(propertyPrefix, key, "circuitBreaker.sleepWindowInMilliseconds", builder.getCircuitBreakerSleepWindowInMilliseconds(), default_circuitBreakerSleepWindowInMilliseconds);
        this.circuitBreakerErrorThresholdPercentage = getProperty(propertyPrefix, key, "circuitBreaker.errorThresholdPercentage", builder.getCircuitBreakerErrorThresholdPercentage(), default_circuitBreakerErrorThresholdPercentage);
        this.circuitBreakerForceOpen = getProperty(propertyPrefix, key, "circuitBreaker.forceOpen", builder.getCircuitBreakerForceOpen(), default_circuitBreakerForceOpen);
        this.circuitBreakerForceClosed = getProperty(propertyPrefix, key, "circuitBreaker.forceClosed", builder.getCircuitBreakerForceClosed(), default_circuitBreakerForceClosed);
        this.executionIsolationStrategy = getProperty(propertyPrefix, key, "execution.isolation.strategy", builder.getExecutionIsolationStrategy(), default_executionIsolationStrategy);
        this.executionTimeoutInMilliseconds = getProperty(propertyPrefix, key, "execution.isolation.thread.timeoutInMilliseconds", builder.getExecutionIsolationThreadTimeoutInMilliseconds(), default_executionTimeoutInMilliseconds);
        this.executionTimeoutEnabled = getProperty(propertyPrefix, key, "execution.timeout.enabled", builder.getExecutionTimeoutEnabled(), default_executionTimeoutEnabled);
        this.executionIsolationThreadInterruptOnTimeout = getProperty(propertyPrefix, key, "execution.isolation.thread.interruptOnTimeout", builder.getExecutionIsolationThreadInterruptOnTimeout(), default_executionIsolationThreadInterruptOnTimeout);
        this.executionIsolationThreadInterruptOnFutureCancel = getProperty(propertyPrefix, key, "execution.isolation.thread.interruptOnFutureCancel", builder.getExecutionIsolationThreadInterruptOnFutureCancel(), default_executionIsolationThreadInterruptOnFutureCancel);
        this.executionIsolationSemaphoreMaxConcurrentRequests = getProperty(propertyPrefix, key, "execution.isolation.semaphore.maxConcurrentRequests", builder.getExecutionIsolationSemaphoreMaxConcurrentRequests(), default_executionIsolationSemaphoreMaxConcurrentRequests);
        this.fallbackIsolationSemaphoreMaxConcurrentRequests = getProperty(propertyPrefix, key, "fallback.isolation.semaphore.maxConcurrentRequests", builder.getFallbackIsolationSemaphoreMaxConcurrentRequests(), default_fallbackIsolationSemaphoreMaxConcurrentRequests);
        this.fallbackEnabled = getProperty(propertyPrefix, key, "fallback.enabled", builder.getFallbackEnabled(), default_fallbackEnabled);
        this.metricsRollingStatisticalWindowInMilliseconds = getProperty(propertyPrefix, key, "metrics.rollingStats.timeInMilliseconds", builder.getMetricsRollingStatisticalWindowInMilliseconds(), default_metricsRollingStatisticalWindow);
        this.metricsRollingStatisticalWindowBuckets = getProperty(propertyPrefix, key, "metrics.rollingStats.numBuckets", builder.getMetricsRollingStatisticalWindowBuckets(), default_metricsRollingStatisticalWindowBuckets);
        this.metricsRollingPercentileEnabled = getProperty(propertyPrefix, key, "metrics.rollingPercentile.enabled", builder.getMetricsRollingPercentileEnabled(), default_metricsRollingPercentileEnabled);
        this.metricsRollingPercentileWindowInMilliseconds = getProperty(propertyPrefix, key, "metrics.rollingPercentile.timeInMilliseconds", builder.getMetricsRollingPercentileWindowInMilliseconds(), default_metricsRollingPercentileWindow);
        this.metricsRollingPercentileWindowBuckets = getProperty(propertyPrefix, key, "metrics.rollingPercentile.numBuckets", builder.getMetricsRollingPercentileWindowBuckets(), default_metricsRollingPercentileWindowBuckets);
        this.metricsRollingPercentileBucketSize = getProperty(propertyPrefix, key, "metrics.rollingPercentile.bucketSize", builder.getMetricsRollingPercentileBucketSize(), default_metricsRollingPercentileBucketSize);
        this.metricsHealthSnapshotIntervalInMilliseconds = getProperty(propertyPrefix, key, "metrics.healthSnapshot.intervalInMilliseconds", builder.getMetricsHealthSnapshotIntervalInMilliseconds(), default_metricsHealthSnapshotIntervalInMilliseconds);
        this.requestCacheEnabled = getProperty(propertyPrefix, key, "requestCache.enabled", builder.getRequestCacheEnabled(), default_requestCacheEnabled);
        this.requestLogEnabled = getProperty(propertyPrefix, key, "requestLog.enabled", builder.getRequestLogEnabled(), default_requestLogEnabled);
        this.executionIsolationThreadPoolKeyOverride = HystrixPropertiesChainedProperty.forString().add(propertyPrefix + ".command." + key.name() + ".threadPoolKeyOverride", (Object)null).build();
    }
```



**例如：超时降级execution.isolation.thread.timeoutInMilliseconds和服务熔断的circuitBreaker.enabled和circuitBreaker.requestVolumeThreshold等等都在上面可以找到**



##### 全局服务降级

**实现全局服务降级主要就是靠@DefaultProperties和@HystrixCommand**

**进入@DefaultProperties**

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface DefaultProperties {
    String groupKey() default "";

    String threadPoolKey() default "";

    HystrixProperty[] commandProperties() default {};

    HystrixProperty[] threadPoolProperties() default {};

    Class<? extends Throwable>[] ignoreExceptions() default {};

    HystrixException[] raiseHystrixExceptions() default {};

    String defaultFallback() default "";
}

```

**使用方法：这个注解加到消费者的controller层上，必要配置（defaultFallback）也就是默认服务降级方法名称，然后在需要使用默认服务降级的方法上加上@HystrixCommand（不加任何参数）即可**

**小坑：注意=======>默认服务降级的方法也就是defaultFallback方法“”不能有任何参数“”，不然就会报错**

**比如如下配置**

```java
@RestController
@DefaultProperties(defaultFallback = "queryAllDept_Hystrix") //默认全局服务降级，也就是配置上去后，需要服务降级的方法加上@HystrixCommand不加参数即可
public class deptController {

    @Autowired
    private deptService deptService;


//    @HystrixCommand(fallbackMethod = "queryAllDept_Hystrix",commandProperties = {
//            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1500")
//    })

    @HystrixCommand //触发全局服务降级
    @RequestMapping("/feign/queryAllDept")
    public List<dept> queryAllDept(String id){
        int i = Integer.parseInt(id);
        if(i<0){
            throw new RuntimeException();
        }
        return deptService.queryAllDept(id);
    }


//    //服务降级方法,全局服务降级方法不能有参数
    public List<dept> queryAllDept_Hystrix(){

        List<dept> depts = new ArrayList<>();
        depts.add(new dept("1000","Hystrix_fallback"));
        return depts;
    }

}

```







#### 服务熔断

**Pom.xml**

```xml
 <!--        Hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```



**在springcloud-02-provider-dept8001和8002的controller**

```java
@RestController
public class deptController {

    private deptService deptService;

    @Value("${server.port}")
    private String port;


    @Autowired
    @Qualifier("deptServiceImpl")
    public void setDeptService(com.boot.service.deptService deptService) {
        this.deptService = deptService;
    }
    @GetMapping(path = "/queryAllDept")
    @HystrixCommand(fallbackMethod = "queryAllDept_Hystrix",commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "20000"),//注意这是毫秒。1秒=1000毫秒
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "50")

    })
    public List<dept> queryAllDept(String id){
        List<dept> depts = deptService.queryAllDept();
        depts.add(new dept("999",port));
//        try {
//            Thread.sleep(1500);
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }

        if(Integer.parseInt(id)<0){
            throw new RuntimeException();
        }

        return depts;
    }

    /**
     * 服务熔断
     */
    public List<dept> queryAllDept_Hystrix(String id){

        List<dept> depts = deptService.queryAllDept();
        depts.add(new dept("1066","Break"));
        return depts;
    }

}

```



**提供者主启动类：**

```java
@SpringBootApplication
@EnableEurekaClient //eureka客户端
@EnableCircuitBreaker //开启服务熔断功能
public class SpringBootApplication8001 {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootApplication8001.class,args);


    }

}
```





#### Hystrix-Dashboard

**创建子模块springcloud-02-hystrix-Dashboard8110**

**仪表盘的访问页面：**

**假如端口号是8110：http://localhost:8110/hystrix**

**Pom.xml**

```xml
 <dependencies>

        <!--      web和actuator是必备的，******除了gateway网关不能加入web包  -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--        actuator用来监控-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

<!--     Hystrix-dashboard-->
         <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
 
    </dependencies>
```

**配置文件没有要求，就修改端口号即可**

```yaml
server:
  port: 8110
spring:
  application:
    name: hystrixDashboard8110
```



**主启动类：**

```java
@SpringBootApplication
@EnableHystrixDashboard //开启了Hystrix仪表盘功能
public class springBootApplication8110 {

    public static void main(String[] args) {
        SpringApplication.run(springBootApplication8110.class,args);
    }



}
```





##### Bug：Hystrix仪表盘连接不上

![image-20210227124135194](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210227124135194.png)


**解决方法：在每一个需要监控的模块（比如微服务提供者8001和8002）加上如下配置**

**1.微服务提供者的Pom.xml**

```xml
<!--     Hystrix-dashboard（微服务提供者）-->
         <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
```



**2.在微服务提供者8001和8002加上一个bean**

```java
@SpringBootApplication
@EnableEurekaClient //eureka客户端
@EnableCircuitBreaker //开启服务熔断功能
public class SpringBootApplication8001 {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootApplication8001.class,args);


    }
    //解决Hystrix-dashboard连接不上
    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}
```



##### Bug：Hystrix仪表盘一直是loading

**解决方法：用openFeign去调用一下即可**

 

**成功页面👇**

![image-20210227170129598](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210227170129598.png)


### 服务网关springcloud gateway

**小坑：1.注意===》gateWay不能有springBoot-Web的启动器，不然会报错。。。。。。。。**

**小坑（负载均衡）：2.注意==》需要把gateway当作提供者注册到注册中心eureka中，不然不能进行服务网关的负载均衡**





**服务网关gateway作用：把所有请求都先进入网关gateway，再由gateway进行分发请求，这样的好处就是隐藏分发到的微服务的端口号，安全性更高**





#### 配置gateway服务网关

**1.创建子模块springcloud-02-gateWay9527**

**2.打开pom.xml，添加依赖**

```xml
 		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
```



**3.application.yml**

```yaml
server:
  port: 9527

spring:
  cloud:
    gateway:
      routes: #配置路由
        - id: providerGateway8001
          uri: http://localhost:8001
          predicates: #配置断言
            - Path=/queryAllDept
```



**4.然后再访问http://localhost:9527/queryAllDept?id=1**

#### gateway服务网关负载均衡（lb）

**注意：需要把gateway当作提供者“”注册“”到注册中心eureka中，不然不能进行服务网关的负载均衡****

**1.先导入这个**

```xml
		<!--        eureka Client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```



**2.springBootApplication**

```java
@SpringBootApplication
@EnableEurekaClient
public class springBootApplication9527 {

    public static void main(String[] args) {

        SpringApplication.run(springBootApplication9527.class,args);

    }

}
```



**3.application.yml**

```yaml
server:
  port: 9527

spring:
  application:
    name: gateway9527
  cloud:
    gateway:
      routes:
        - id: providerGateway8001
          uri: lb://PROVIDER-DEPT
          predicates:
            - Path=/queryAllDept

eureka:
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka-server7001.com:7001/eureka/, http://eureka-server7002.com:7002/eureka/, http://eureka-server7003.com:7003/eureka/

```



### 分布式配置中心springcloud config

**1.创建gitee或者GitHub，并把配置文件上传到git**

![image-20210302204336025](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210302204336025.png)





**我们目的是拉取上面红框的配置文件**

#### 分布式配置中心服务器端（server）

**2.创建子模块springcloud-02-config-server9001**



**3.导入依赖**

```xml
<!--        config-server-->
      <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>

```

**4.application.yml**

```yaml
server:
  port: 9001
spring:
  cloud:
    config:
      server:
        git:
          default-label: master #文件所在git的分支，默认是master
          uri: https://gitee.com/youzhengjie/could-config #这个uri就是文件所在的网页地址。。。。
```

**5.启动类**

```java
@SpringBootApplication
@EnableConfigServer //开启分布式配置中心服务器端
public class springBootApplication9001 {

    public static void main(String[] args) {

        SpringApplication.run(springBootApplication9001.class,args);

    }

}
```





#### 分布式配置中心客户端（client）



**1.创建子模块springcloud-02-config-client8002**



**2.导入依赖（注意：web和config-client必须要导入，特别是web依赖，不导入不行）**

```xml
		 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-client</artifactId>
        </dependency>
```



**3.主启动类====不用加什么，就一个普通的springBoot启动类即可**



**4.bootstrap.yml**

```yaml
server:
  port: 8200

spring:
  cloud:
    config:
      label: master
      name: application
      uri: http://localhost:9001
```



**5.controller(去测试有没有sys.version，有的话就说明成功了）**

```java
@RestController
public class testController {

    @Value("${sys.version}")
    private String version;

    @RequestMapping("/getVersion")
    public String getVersion(){

        return version;
    }
}
```



![image-20210303204534651](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210303204534651.png)



**引入springcloud bus消息总线之前。。。。**

**我们用git去把sys.version更新成1.8**



**去访问一下config-server**

![image-20210303204818402](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210303204818402.png)


**发现config-server是可以立刻更新到的，因为config-server和gitee是直连的。**



**我们去访问一下config-client**

![image-20210303204946992](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210303204946992.png)


**发现还是1.7，只有重新启动config-client项目才能更新成1.8，这是我们要用到actuator的refresh端点，手动刷新。**





##### actuator手动刷新config-client

**修改如下：1.在controller加上@RefreshScope**

```java
@RestController
@RefreshScope //config client刷新注解
public class testController {

    @Value("${sys.version}")
    private String version;

    @RequestMapping("/getVersion")
    public String getVersion(){

        return version;
    }



}

```

**2.暴露端点**

```yaml
server:
  port: 8200

spring:
  cloud:
    config:
      label: master
      name: application
      uri: http://localhost:9001

#开启所有端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```



**3.用postman去发送post刷新请求**

**前提：1.发送POST请求到uri/actuator/refresh**

**2.前面几步工作做好，比如@RefreshScope。**




![image-20210303210634591](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210303210634591.png)





**就ok了！！！**

##### 为什么要引入springcloud bus的原因

**感想：假如有20个config-client，我们要去刷新他们，难道一个个去发送post请求/actuator/refresh????那肯定不行，这样可以是可以，但是效率太低了，所以我们就会引入消息总线springcloud bus，一次post刷新请求即可，不管有多少模块需要刷新。很方便**



### 消息总线springcloud Bus

**引入消息总线springcloud bus不需要创建新的模块。。。。只需要用config server和config client即可**

**为什么不需要创建新的模块呢？因为在下面的架构中可以说明一切**

![1202638-20180521203126866-1299643942](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/1202638-20180521203126866-1299643942.png)



#### 安装rabbitMQ环境

**总教程：https://www.cnblogs.com/saryli/p/9729591.html**

**1.安装erlang**

**（1.）下载erlang**

**官网地址：https://www.erlang.org/**

**下载教程：https://www.cnblogs.com/minily/p/7398445.html**

**（2.）配置erlang环境**

**配置教程：https://blog.csdn.net/g6256613/article/details/80191402**

**需要配置环境变量**

![image-20210304210138086](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210304210138086.png)


![image-20210304210211142](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210304210211142.png)


 
**（3.）检查是否安装成功**

**打开cmd，输入erl，有输出说明成功**



**（4.）下载rabbitMQ**

**下载地址：https://www.cnblogs.com/saryli/p/9729591.html**



**。。。。。。。。。。。。省略，在总教程都有。**



**（5.）最后访问http://localhost:15672，如果访问成功，说明rabbitMQ安装成功**



#### 配置springcloud Bus

**有了springcloud bus，我们只需要通知config server，config server就会去“”遍历“”config client，一个个发送刷新的通知，就比如一个个发送/actuator/refresh**

**1.在上面的基础上，config server9001的pom.xml添加依赖**

```xml
		<!--        bus-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```



**注意===========》2.config server 9001的application.yml**

```yaml
spring:
  cloud:
    config:
      server:
        git:
          default-label: master
          uri: https://gitee.com/youzhengjie/could-config
  rabbitmq:
    port: 5672
    username: guest
    password: guest
    host: localhost

#########注意：：：：一定要开放bus-refresh端点
management:
  endpoints:
    web:
      exposure:
        include: "bus-refresh"
```



**3.在上面的基础上，config client 8200 的pom.xml**

```xml
		<!--        bus-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```

**4.config client 8200的application.yml**

```yaml
spring:
  cloud:
    config:
      label: master
      name: application
      uri: http://localhost:9001
  rabbitmq:
    host: localhost
    username: guest
    password: guest
    port: 5672
```



**5.在postman发送刷新命令即可。http://localhost:9001/actuator/bus-refresh**

![image-20210304204839588](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/image-20210304204839588.png)



## NIO+Netty

### NIO三大组件
#### Channel and Buffer
Java NIO的核心：通道(Channel)和缓冲区(Buffer),通道是用来传输数据的,缓冲区是存储数据的。

常见的Channel有以下四种，其中FileChannel主要用于文件传输，其余三种用于网络通信。
* FileChannel
* SocketChannel
* DatagramChannel
* ServerSocketChannel

Buffer有几种，使用最多的是ByteBuffer
* ByteBuffer
  * MappedByteBuffer
  * DirectByteBuffer
  * HeapByteBuffer
* ShortBuffer
* IntBuffer
* LongBuffer
* FloatBuffer
* DoubleBuffer
* CharBuffer

**8大基本数据类型除了boolean没有Buffer，其余的7种基本类型都有**

#### Selector
未使用Selector之前，有如下几种方案
> 1.多线程技术

**实现逻辑** :每一个连接进来都开一个线程去处理Socket。

**缺点:**
* 如果同时有100000个（**大量**）连接进来,系统大概率是挡不住的,而且线程会占用内存，会导致内存不足。
* 线程需要进行上下文切换，成本高

> 2.采用线程池技术

**实现逻辑** :创建一个固定大小(系统能够承载的线程数)的线程池对象,去处理连接的请求，假如线程池大小为
100个线程数，这时候同时并发连接1000个Socket，此时只有100个Socket会得到处理，其余的会阻塞。这样很好的防止了系统线程数
过多导致线程占用内存大，不容易导致系统由于内存占用的问题而崩溃。

**相对于第一种多线程技术处理客户端Socket,第二种方案使用线程池去处理连接会更好**,但是还是不够好

**缺点:** 
* 阻塞模式下，线程仅能处理一个连接，若socket连接一直未断开，则该线程无法处理其他socket。

> 3.使用Selector选择器
![img1.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/img1.png)


selector的作用就是配合一个线程来管理多个channel,获取这些 channel 上发生的事件，这些 channel 工作在非阻塞模式下，当一个channel中没有执行任务时，可以去执行其他channel中的任务

**注意：fileChannel因为是阻塞式的，所以无法使用selector**

**使用场景：适合连接数多，但流量较少的场景**

**流程：** 假如当前Selector绑定的Channels没有任何一个Channel触发了感兴趣的事件，
则selector的select()方法会阻塞线程，直到channel触发了事件。这些事件发生后，select方法就会返回这些事件交给thread来处理。

#### IO and NIO 区别

**区别：**
* IO是面向**流**的，NIO是面向缓冲区（**块**）的
* Java IO的各种流是阻塞的，而Java NIO是非阻塞的
* Java NIO的选择器允许一个单独的线程来监视多个输入通道

> 普通io读取文件

```java
@Test
    public void test01(){

        try {
            FileInputStream fileInputStream = new FileInputStream("data.txt");

            long start = System.currentTimeMillis();

            byte bytes[]=new byte[1024];

            int n=-1;
            while ((n=fileInputStream.read(bytes,0,1024))!=-1){

                String s = new String(bytes,0,n,"utf-8");

                System.out.println(s);
            }
            long end = System.currentTimeMillis();
            System.out.println("普通io共耗时："+(end-start)+"ms");
        } catch (Exception e) {
            e.printStackTrace();
        }
        
    }
```

> 缓冲流IO读取文件

```java

@Test
    public void test02(){

        try {
            BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream("data.txt"));

            long start = System.currentTimeMillis();

            byte bytes[]=new byte[1024];

            int n=-1;

            while ((n=bufferedInputStream.read(bytes,0,1024))!=-1){

                String s = new String(bytes,0,n,"utf-8");

                System.out.println(s);
            }

            long end = System.currentTimeMillis();
            System.out.println("缓冲流io共耗时："+(end-start)+"ms");
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
```

> Nio-FileChannel读取文件

```java

//方式1
@Test
    public void test3(){

        try {
            //获取channel,FileInputStream生成的channel只有读的权利
            FileChannel channel = new FileInputStream("data.txt").getChannel();
            ByteBuffer byteBuffer = ByteBuffer.allocate(1024); //开辟一块缓冲区

            long start = System.currentTimeMillis();
            while (true){

                //写入操作
                int read = channel.read(byteBuffer); //如果read=-1，说明缓存“块”没有数据了

                if(read==-1){
                    break;
                }else {

                    byteBuffer.flip();//读写切换，切换为读的操作，实质上就是把limit=position,position=0

                    String de = StandardCharsets.UTF_8.decode(byteBuffer).toString();
                    System.out.println(de);

                    byteBuffer.clear(); //切换为写
                }
            }
            long end = System.currentTimeMillis();
            System.out.println("heap nio共耗时："+(end-start)+"ms");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

//方式2
@Test
    public void test4(){

        ByteBuffer byteBuffer = ByteBuffer.allocate(10);

        byteBuffer.put("helloWorld".getBytes());

        debugAll(byteBuffer);

        byteBuffer.flip(); //读模式

        while (byteBuffer.hasRemaining()){

            System.out.println((char)byteBuffer.get());
        }


        byteBuffer.flip();

        System.out.println(StandardCharsets.UTF_8.decode(byteBuffer).toString());

    }

```

#### ByteBuffer

**创建ByteBuffer缓冲区：**
* ByteBuffer.allocate(int capacity)
* ByteBuffer.allocateDirect(int capacity)
* ByteBuffer.wrap(byte[] array,int offset, int length)

**ByteBuffer常用方法：**
* get()
* get(int index)
* put(byte b)
* put(byte[] src)
* limit(int newLimit)
* mark()
* reset()
* clear()
* flip()
* compact()


#### 字符串与ByteBuffer的相互转换

> 字符串转换成ByteBuffer

```java
ByteBuffer byteBuffer = StandardCharsets.UTF_8.encode("hello world\nabc\n\baaa");
```

> ByteBuffer转换成String

```java
String str = StandardCharsets.UTF_8.decode(byteBuffer).toString();
```

> 整个Demo

```java
 @Test
    public void test5(){
        //字符串转换成ByteBuffer
        ByteBuffer byteBuffer = StandardCharsets.UTF_8.encode("hello world\nabc\n\baaa");
        //通过StandardCharsets的encode方法获得ByteBuffer，此时获得的ByteBuffer为读模式，无需通过flip切换模式
//        byteBuffer.flip(); //这句话不能加，encode转换成ByteBuffer默认是读模式
        while (byteBuffer.hasRemaining()){

            System.out.printf("%c",(char)byteBuffer.get());
        }

        byteBuffer.flip();
        //ByteBuffer转换成String
        String str = StandardCharsets.UTF_8.decode(byteBuffer).toString();
        System.out.println("\n--------------");
        System.out.println(str);
    }
```

#### 解决粘包和拆包问题

```java
@Test
    public void test6(){

        String msg = "hello,world\nI'm abc\nHo";

        ByteBuffer byteBuffer = ByteBuffer.allocate(32);

        byteBuffer.put(msg.getBytes());

        byteBuffer=splitGetBuffer(byteBuffer);

        byteBuffer.put("w are you?\n".getBytes()); //多段发送数据

        byteBuffer=splitGetBuffer(byteBuffer);

        byteBuffer.put("aa  bccdd?\n".getBytes()); //多段发送数据

        byteBuffer=splitGetBuffer(byteBuffer);

    }

    private ByteBuffer splitGetBuffer(ByteBuffer byteBuffer) {

        byteBuffer.flip();
        StringBuilder stringBuilder = new StringBuilder();
        int index=-1;
        for (int i = 0; i < byteBuffer.limit(); i++) {

            if(byteBuffer.get(i)!='\n'){ //get(i)不会让position+1

                stringBuilder.append((char) byteBuffer.get(i));


            }else{
                index=i; //记录最后一个分隔符下标
                String data = stringBuilder.toString();
                ByteBuffer dataBuf = ByteBuffer.allocate(data.length());
                dataBuf.put(data.getBytes());
                dataBuf.flip();
                debugAll(dataBuf);
                dataBuf.clear();
                stringBuilder=new StringBuilder();
            }
        }

        ++index;
        ByteBuffer temp = ByteBuffer.allocate(byteBuffer.capacity());
        for (;index<byteBuffer.limit();++index){

            temp.put(byteBuffer.get(index));
        }

        return temp;
    }
```

### 文件编程


#### FileChannel

**因为FileChannel只能工作在阻塞环境下，而Selector是非阻塞的，所以FileChannel无法注册到Selector里面去。**

##### 获取FileChannel

FileChannel不能直接打开,一定要用FileInputStream或者FileOutputStream或者RandomAccessFile来获取FileChannel对象，
使用**getChannel**方法即可。

**注意以下几点：**
* 通过FileInputStream获取的channel只能读
* 通过FileOutputStream获取的channel只能写
* 通过 RandomAccessFile 是否能读写根据构造 RandomAccessFile 时的读写模式决定

##### FileChannel读取

通过 FileInputStream 获取channel，通过read方法将数据写入到ByteBuffer中，read方法的返回值表示读到了多少字节，若读到了文件末尾则返回-1

```java
int read = channel.read(buffer);
```

##### FileChannel写入

因为channel也是有大小的，所以 write方法并不能保证一次将 buffer中的内容全部写入channel。必须需要按照以下规则进行写入

```java
// 通过hasRemaining()方法查看缓冲区中是否还有数据未写入到通道中
while(buffer.hasRemaining()) {
	channel.write(buffer);
}
```

##### 强制写入

操作系统出于性能的考虑，会将数据缓存，不是立刻写入磁盘，而是等到缓存满了以后将所有数据一次性的写入磁盘。可以调用force(true)方法将文件内容和元数据（文件的权限等信息）立刻写入磁盘



#### 两个Channel传输数据

##### transferTo方法的使用

```java

        //方法一：
        FileInputStream fileInputStream = new FileInputStream("data.txt"); //读的通道
        FileChannel from = fileInputStream.getChannel();

        FileOutputStream fileInputStream1 = new FileOutputStream("to.txt"); //写的通道
        FileChannel to = fileInputStream1.getChannel();


        long l = from.transferTo(0, from.size(), to);
        

        //方法二：
        RandomAccessFile r1 = new RandomAccessFile("data.txt", "rw"); //都开启rw权限
        FileChannel from1 = r1.getChannel();


        RandomAccessFile r2 = new RandomAccessFile("to.txt", "rw");
        FileChannel to2 = r2.getChannel();

        from1.transferTo(0,r1.length(),to2);

```

##### transferTo方法介绍

使用transferTo方法可以快速、高效地将一个channel中的数据传输到另一个channel中，但**一次只能传输2G**的内容，
**transferTo方法的底层使用了零拷贝技术**，



#### Path与Paths

* Path用来表示文件路径
* Paths是工具类，用来获取Path实例

```java
 Path path = Paths.get("data.txt");

 Path path1 = Paths.get("D:\\java code\\netty-study\\data.txt");
```

#### Files

##### 判断文件是否存在
```java
    Path path = Paths.get("data.txt");
    boolean exists = Files.exists(path);
```

##### 创建一级目录

* createDirectory(path)

**如果文件夹已存在**，则会报错。FileAlreadyExistsException,
此方法只能创建一级目录，**如果用此方法创建多级目录则会报错**NoSuchFileException。

```java
    Path path = Paths.get("D:\\img");
    Path directory = Files.createDirectory(path);
```

##### 创建多级目录

* createDirectories(path)

```java
    Path path = Paths.get("D:\\img\\a\\b");
    Path directories = Files.createDirectories(path);
```


##### 拷贝文件

```java
    //这种方式如果目标文件‘to’存在则会报错FileAlreadyExistsException
    Path from = Paths.get("data.txt");
    Path to = Paths.get("D:\\img\\target.txt"); //文件名也要写
    Files.copy(from,to);
    //只需要加StandardCopyOption.REPLACE_EXISTING就不会报错，因为它会直接替换掉目标文件
    Path from = Paths.get("data.txt");
    Path path = Paths.get("D:\\img\\target.txt"); //文件名也要写
    Files.copy(from,path, StandardCopyOption.REPLACE_EXISTING);
```

##### 移动文件

```java
    Path source = Paths.get("data.txt");
    Path target = Paths.get("D:\\img\\target.txt");
    Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
```

* StandardCopyOption.ATOMIC_MOVE保证文件移动的原子性


##### 删除文件

```java
    Path target = Paths.get("D:\\img\\target.txt");
    Files.delete(target); //删除文件
```


##### 遍历文件夹

* walkFileTree(Path, FileVisitor)方法
  * Path：文件起始路径
  * FileVisitor：文件访问器，使用访问者模式，这个接口有如下方法
    * preVisitDirectory：访问目录前的操作
    * visitFile：访问文件的操作
    * visitFileFailed：访问文件失败时的操作
    * postVisitDirectory：访问目录后的操作

```java
    Path target = Paths.get("D:\\cTest");

    Files.walkFileTree(target,new SimpleFileVisitor<Path>(){

      @Override
      public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
        System.out.println("1:"+dir);
        return super.preVisitDirectory(dir, attrs);
      }

      @Override
      public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
        System.out.println("2:"+file);
        return super.visitFile(file, attrs);
      }
    });
```

### 网络编程

#### NIO通信(阻塞模式)

这里有一段简易的通信代码:

**服务器端：**

```java
      ServerSocketChannel serverSocketChannel = ServerSocketChannel.open(); //打开serverSocketChannel

      serverSocketChannel.bind(new InetSocketAddress(8080));

      while (true){

      System.out.println("waiting.....");
          SocketChannel socketChannel = serverSocketChannel.accept(); //阻塞
      System.out.println("connect success");
          ByteBuffer byteBuffer = ByteBuffer.allocate(100);
          socketChannel.read(byteBuffer); //阻塞，等待消息发送过来即可封装到缓存里去
          byteBuffer.flip();
      System.out.println(StandardCharsets.UTF_8.decode(byteBuffer).toString());
      }
```

**客户端：**

```java
      SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress( 8080));
      ByteBuffer byteBuffer = StandardCharsets.UTF_8.encode("this is nio");
      socketChannel.write(byteBuffer);
```

**实际上，这个和以前的IO+Socket进行通信是一样的，都是属于阻塞状态。**


#### NIO通信(非阻塞模式)

* configureBlocking(false)

可以通过ServerSocketChannel的configureBlocking(false)方法将获得连接设置为非阻塞的。此时若没有连接，accept会返回null,
可以通过SocketChannel的configureBlocking(false)方法将从通道中读取数据设置为非阻塞的。若此时通道中没有数据可读，read会返回-1

**服务器端：**
```java
      ByteBuffer byteBuffer = ByteBuffer.allocate(100);
      ServerSocketChannel serverSocketChannel = ServerSocketChannel.open(); //打开通道

      serverSocketChannel.bind(new InetSocketAddress(8082));
      //由于accept方法是阻塞的，我们只需要一行代码就能让它变成非阻塞的
      //开启非阻塞的之后accept方法如果没有连接到客户端就会从阻塞变成返回'null'
      serverSocketChannel.configureBlocking(false);//开启非阻塞
      while (true){
//          System.out.println("waiting...");
          SocketChannel socketChannel = serverSocketChannel.accept(); //阻塞方法

//          System.out.println(socketChannel);

              if(socketChannel!=null){
                  System.out.println("等待读取");
                  socketChannel.configureBlocking(false); //设置SocketChannel为非阻塞
                  int read = socketChannel.read(byteBuffer);//阻塞方法
                  System.out.println("读取到"+read+"字节");
                  if(read>0){

                      byteBuffer.flip();
                      System.out.println(StandardCharsets.UTF_8.decode(byteBuffer).toString());
                  }
              }
      }
```

**客户端：**

```java
      SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress(8082));
      ByteBuffer byteBuffer = StandardCharsets.UTF_8.encode("hello");
      socketChannel.write(byteBuffer);
```

#### Selector

**Selector是基于事件驱动的**

##### 多路复用
单线程可以配合Selector完成对多个Channel读写事件的监控，这称之为多路复用。

**注意：**
* 多路复用只能用于网络IO上，文件IO由于只能处于阻塞环境下才能进行，所以无法多路复用
* 如果不用Selector的非阻塞模式，线程大部分时间都在做无用功，而Selector能够保证以下几点
  * 有可连接事件时才去连接
  * 有可读事件才去读取
  * 有可写事件才去写入

##### 4种事件类型

进入**SelectionKey**这个类可以看到：
```java
public static final int OP_READ = 1 << 0; //read事件
public static final int OP_WRITE = 1 << 2; //write事件
public static final int OP_CONNECT = 1 << 3; //connect事件
public static final int OP_ACCEPT = 1 << 4; //accept事件
```


##### 核心方法select

* select()
**select方法会一直阻塞直到绑定事件发生**


##### accept事件

**服务器端：**
```java
    Selector selector = Selector.open(); // 创建选择器
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

    serverSocketChannel.bind(new InetSocketAddress(8081));

    serverSocketChannel.configureBlocking(false); // 通道必须是非阻塞的

    serverSocketChannel.register(
        selector, SelectionKey.OP_ACCEPT); // 把channel注册到selector，并选择accept事件

    for (; ; ) {

      selector.select(); // 选择事件，此时会阻塞，当事件发生时会自动解除阻塞

      System.out.println("begin");

      // 遍历事件发生的集合，获取对应事件
      selector
          .selectedKeys()
          .forEach(
              selectionKey -> {
                if (selectionKey.isAcceptable()) {
                  try {
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    System.out.println("已连接");
                    // 处理完之后记得在发生事件的集合中移除该事件
                    selector.selectedKeys().remove(selectionKey);

                  } catch (IOException e) {
                    e.printStackTrace();
                  }
                }
              });
    }
```

##### read事件

**原生NIO是真tmd难用，恶心**
**当accept事件处理之后立刻设置read事件,但不处理read事件，因为用户可能只是连接，但是没有写数据，所以要基于事件触发**
**别忘了accept事件处理之后要设置为非阻塞模式configureBlocking(false)**

```java
    Selector selector = Selector.open(); // 创建选择器
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

    serverSocketChannel.bind(new InetSocketAddress(8081));

    serverSocketChannel.configureBlocking(false); // 通道必须是非阻塞的

    serverSocketChannel.register(
        selector, SelectionKey.OP_ACCEPT); // 把channel注册到selector，并选择accept事件

    try {
      while (true) {

        int count = selector.select(); // 选择事件，此时会阻塞，当事件发生时会自动解除阻塞

        Set<SelectionKey> selectionKeys = selector.selectedKeys();
        Iterator<SelectionKey> iterator = selectionKeys.iterator();

        while (iterator.hasNext()){
          SelectionKey selectionKey = iterator.next();
          if (selectionKey.isAcceptable()) { // 处理accept事件
            try {
              ServerSocketChannel serverSocket = (ServerSocketChannel) selectionKey.channel();
              System.out.println("已连接");

              SocketChannel socketChannel = serverSocket.accept();
              socketChannel.configureBlocking(false);
              socketChannel.register(selector, SelectionKey.OP_READ); // 读事件
              iterator.remove();

            } catch (IOException e) {

            }
          } else if (selectionKey.isReadable()) { // 处理read事件
            // 获取socketChannel,实际上这个channel就是上面注册进selector的对象
            SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
            ByteBuffer byteBuffer = ByteBuffer.allocate(100);
            try{
              int read = socketChannel.read(byteBuffer);
              System.out.println("read:"+read);
            }catch (Exception e){
//              e.printStackTrace();
             continue;  //一定要这样写。。。。。。。防止多次read报错
            }
            byteBuffer.flip();
            debugAll(byteBuffer);
            byteBuffer.clear();
            iterator.remove();

          }

        }

      }
    } catch (Exception e) {
      e.printStackTrace();
    }
```
 
##### selector注意点
* 事件发生后，要么处理，要么取消（cancel），不能什么都不做，否则下次该事件仍会触发
* 事件处理之后一定要把selector.**selectedKeys**这个集合中当前处理完成的事件**remove**掉


#### 零拷贝
零拷贝指的是数据**无需拷贝到JVM内存**中，同时具有以下三个优点:
* 更少的用户态与内核态的切换
* 不利用cpu计算，减少cpu缓存伪共享
* 零拷贝适合小文件传输


#### NIO优化

使用DirectByteBuffer
* ByteBuffer.allocate(10)底层对应 HeapByteBuffer，使用的还是Java堆内存
* ByteBuffer.allocateDirect(10)底层对应DirectByteBuffer，使用的是操作系统内存，不过需要手动释放内存

**优点：**
* 减少了一次数据拷贝，用户态与内核态的切换次数没有减少
* 这块内存不受 JVM 垃圾回收的影响，因此内存地址固定，有助于 IO 读写


##### linux2.4优化

* Java 调用 transferTo 方法后，要从 Java 程序的用户态切换至内核态，使用 DMA将数据读入内核缓冲区，不会使用 CPU
* 只会将一些 offset 和 length 信息拷入 socket 缓冲区，几乎无消耗
* 使用 DMA 将 内核缓冲区的数据写入网卡，不会使用 CPU
* 整个过程仅只发生了1次用户态与内核态的切换，数据拷贝了 2 次


### Netty入门

**Netty提供异步的、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序**

#### Netty著名项目

**由Netty开发的开源框架：**
* dubbo
* Zookeeper
* RocketMQ

#### Netty的优势

* 不需要自己构建协议，Netty自带了多种协议，例如HTTP协议
* 解决了TCP传输问题，如粘包、半包
* 解决了一个epoll空轮询的JDK bug。（作者遇到过）,即selector的select方法默认是阻塞的，但是并没有阻塞会一直空轮询。
* Netty对JDK的NIO API进行增强,如下：
  * ThreadLocal==>FastThreadLocal
  * ByteBuffer==>ByteBuf(重要)，支持动态扩容，不像原厂的JDK的ByteBuffer超过缓存就报错


#### Netty Maven

```xml
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.65.Final</version>
        </dependency>
```

**依赖说明：暂时不推荐使用Netty5，使用Netty4即可**


#### 第一个Netty应用

**服务器端：**

```java
  private static final Logger log = LoggerFactory.getLogger(NettyServer.class);

  public static void main(String[] args) {

    // Netty的服务器端启动器，装配Netty组件
    new ServerBootstrap()
        // NioEventLoopGroup底层就是线程池+selector
        .group(new NioEventLoopGroup())
        // 通道
        .channel(NioServerSocketChannel.class)
        //“每一个”SocketChannel客户端连接上服务器端“都会”执行这个初始化器ChannelInitializer
        //但是每一个SocketChannel只能够让这个初始化器执行一次
        .childHandler(
            new ChannelInitializer<NioSocketChannel>() {
              @Override
              protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                  log.info("initChannel start......");
                  //往处理器流水线pipeline添加处理器
                  //因为'客户端'发送数据会进行'字符串的编码'再发送到服务器端，所以这里要'创建一个字符串解码器'StringDecoder
                  nioSocketChannel.pipeline().addLast(new StringDecoder());
                  //添加接收数据需要的处理器适配器
                  nioSocketChannel.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                      //重写通道的‘’读‘’方法,msg就是接收到的数据
                      @Override
                      public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                          log.warn(msg.toString()); //打印数据
                          super.channelRead(ctx, msg);
                      }
                  });
                  log.info("initChannel end......");
              }
            })
        .bind(8082);
  }
```

**客户端：**

```java
  private static final Logger log = LoggerFactory.getLogger(NettyClient.class);

  public static void main(String[] args) throws InterruptedException {

      //创建Netty客户端的启动器，装配Netty组件
    new Bootstrap()
        .group(new NioEventLoopGroup())
        .channel(NioSocketChannel.class)
        //一旦执行这个应用立刻初始化，这个和childHandler有所不同
        //childHandler是需要socket连接上在初始化，这个不需要。。。。。
        .handler(
            new ChannelInitializer<Channel>() {
              @Override
              protected void initChannel(Channel channel) throws Exception {
                  log.info("initChannel start......");
                  //由于发送的数据需要进行编码再发送，所以需要一个字符串编码器
                  //往通道流水线添加一个字符串编码器
                  channel.pipeline().addLast(new StringEncoder());
                  log.info("initChannel end......");
              }
            })
        // connect方法是“”异步“”的
        .connect("localhost", 8082)
        //坑点：由于connect方法是异步的，所以要同步。。。。。
        //由于connect方法是异步的，如果没有进行同步，可能会造成发送数据在连接服务器之前。
        //一般来说connect连接服务器大概需要>1s，而writeAndFlush是立刻发送数据，所以这里一定要使用sync方法进行同步
        .sync()
        // 获取通道。然后发送数据
        .channel()
        .writeAndFlush("hello你好");
  }
```

#### Netty组件

> 查看CPU最大核心数

```java
int hx = NettyRuntime.availableProcessors(); //cpu核心数
```

##### EventLoop

**事件循环对象**EventLoop

EventLoop本质是一个单线程执行器（同时维护了一个 Selector），里面有run方法处理一个或多个Channel上源源不断的io事件

**事件循环组**EventLoopGroup

EventLoopGroup是一组EventLoop，而每一个EventLoop都维护着一个selector，Channel 一般会调用EventLoopGroup的register方法来绑定其中一个EventLoop。


```java
      int count=3;
      EventLoopGroup ev=new NioEventLoopGroup(count);
      System.out.println(ev.next().hashCode());//1
      System.out.println(ev.next().hashCode());//2
      System.out.println(ev.next().hashCode());//3
      System.out.println(ev.next().hashCode());//4
```

通过上面的代码可以看出1和4是同一个对象，因为他们的hashCode相同。得出EventLoopGroup是一个**线程池**，里面装载着>1个的EventLoop，
EventLoop底层维护了一个线程和selector，而count可以指定EventLoopGroup的线程池大小。


> EventLoop普通任务与定时任务

```java
      EventLoopGroup ev=new NioEventLoopGroup(3);
      //普通任务
      ev.next().submit(()->{

          System.out.println("111");

      });

      System.out.println("222");

      //定时任务
      ev.next().scheduleAtFixedRate(()->{

          System.out.println("333");

      },0,1,TimeUnit.SECONDS);

```

> 关闭EventLoopGroup

```java
      EventLoopGroup eventLoopGroup = new NioEventLoopGroup();
      eventLoopGroup.shutdownGracefully(); //优雅的关闭EventLoopGroup
```


> 分工

```java
      // Netty的服务器端启动器，装配Netty组件
      new ServerBootstrap()
               //******NioEventLoopGroup的分工合作，第一个NioEventLoopGroup处理accept事件
              //第二个NioEventLoopGroup处理读写事件
              .group(new NioEventLoopGroup(),new NioEventLoopGroup())
              // 通道
              .channel(NioServerSocketChannel.class)
              //“每一个”SocketChannel客户端连接上服务器端“都会”执行这个初始化器ChannelInitializer
              //但是每一个SocketChannel只能够让这个初始化器执行一次
              .childHandler(
                      new ChannelInitializer<NioSocketChannel>() {
                          @Override
                          protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                              log.info("initChannel start......");
                              //往处理器流水线pipeline添加处理器
                              //因为'客户端'发送数据会进行'字符串的编码'再发送到服务器端，所以这里要'创建一个字符串解码器'StringDecoder
                              nioSocketChannel.pipeline().addLast(new StringDecoder());
                              //添加接收数据需要的处理器适配器
                              nioSocketChannel.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                                  //重写通道的‘’读‘’方法,msg就是接收到的数据
                                  @Override
                                  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                      log.warn(msg.toString()); //打印数据
                                      super.channelRead(ctx, msg);
                                  }
                              });
                              log.info("initChannel end......");
                          }
                      })
              .bind(8082);
```


##### Channel

**Channel常用方法：**
* close()
  * 可以用来关闭Channel
* closeFuture()
  * 用来处理 Channel 的关闭
* pipeline()
  * 添加处理器
* write()
  * 写入数据，只有当缓冲满了或者调用了flush()方法后，才会将数据通过 Channel 发送出去
* writeAndFlush()
  * 立即发送数据，相当于同时调用write和flush方法，好处是不用等缓存满了才能发出数据的问题

> ChannelFuture

**获取ChannelFuture**

```java
      //创建Netty客户端的启动器，装配Netty组件
      ChannelFuture channelFuture = new Bootstrap()
              .group(new NioEventLoopGroup())
              .channel(NioSocketChannel.class)
              //一旦执行这个应用立刻初始化，这个和childHandler有所不同
              //childHandler是需要socket连接上在初始化，这个不需要。。。。。
              .handler(
                      new ChannelInitializer<Channel>() {
                          @Override
                          protected void initChannel(Channel channel) throws Exception {
                              //由于发送的数据需要进行编码再发送，所以需要一个字符串编码器
                              //往通道流水线添加一个字符串编码器
                              channel.pipeline().addLast(new StringEncoder());
                          }
                      })
              // connect方法是“”异步“”的
              .connect("localhost", 8082);
```

**发送数据的两种方式**

* sync同步channelFuture再发送数据
* channelFuture添加监听器

**这两种方法本质上都是为了让channelFuture成功创建也就是connect方法完成调用之后才发送数据**

```java
      //创建Netty客户端的启动器，装配Netty组件
      ChannelFuture channelFuture = new Bootstrap()
              .group(new NioEventLoopGroup())
              .channel(NioSocketChannel.class)
              //一旦执行这个应用立刻初始化，这个和childHandler有所不同
              //childHandler是需要socket连接上在初始化，这个不需要。。。。。
              .handler(
                      new ChannelInitializer<Channel>() {
                          @Override
                          protected void initChannel(Channel channel) throws Exception {
                              //由于发送的数据需要进行编码再发送，所以需要一个字符串编码器
                              //往通道流水线添加一个字符串编码器
                              channel.pipeline().addLast(new StringEncoder());
                          }
                      })
              // connect方法是“”异步“”的
              .connect("localhost", 8082);

      //"方法一"：
      //由于connect方法是异步的，如果没有进行同步，可能会造成发送数据在连接服务器之前。
      //一般来说connect连接服务器大概需要>1s，而writeAndFlush是立刻发送数据，所以这里一定要使用sync方法进行同步

//      channelFuture.sync();
//      Channel channel = channelFuture.channel();
//      channel.writeAndFlush("你好");

      //方法二：使用监听器，监听channelFuture是否完成连接。因为channelFuture只有connect完成之后才会创建
      //使用这种监听器方法就不需要sync进行同步了
      channelFuture.addListener(new ChannelFutureListener() {
          //当connect成功连接之后就会进入这个方法
          @Override
          public void operationComplete(ChannelFuture future) throws Exception {

              Channel channel = future.channel();
              channel.writeAndFlush("operationComplete");
          }
      });
```

> 关闭通道channel

```java
      EventLoopGroup eventLoopGroup = new NioEventLoopGroup();
      //创建Netty客户端的启动器，装配Netty组件
      ChannelFuture channelFuture = new Bootstrap()
              .group(eventLoopGroup)
              .channel(NioSocketChannel.class)
              //一旦执行这个应用立刻初始化，这个和childHandler有所不同
              //childHandler是需要socket连接上在初始化，这个不需要。。。。。
              .handler(
                      new ChannelInitializer<Channel>() {
                          @Override
                          protected void initChannel(Channel channel) throws Exception {
                             //日志
                              channel.pipeline().addLast(new LoggingHandler(LogLevel.INFO));
                            //由于发送的数据需要进行编码再发送，所以需要一个字符串编码器
                              //往通道流水线添加一个字符串编码器
                              channel.pipeline().addLast(new StringEncoder());
                          }
                      })
              // connect方法是“”异步“”的
              .connect("localhost", 8082);

      //"方法一"：
      //由于connect方法是异步的，如果没有进行同步，可能会造成发送数据在连接服务器之前。
      //一般来说connect连接服务器大概需要>1s，而writeAndFlush是立刻发送数据，所以这里一定要使用sync方法进行同步

//      channelFuture.sync();
//      Channel channel = channelFuture.channel();
//      channel.writeAndFlush("你好");

      //方法二：使用监听器，监听channelFuture是否完成连接。因为channelFuture只有connect完成之后才会创建
      //使用这种监听器方法就不需要sync进行同步了
      channelFuture.addListener(new ChannelFutureListener() {
          //当connect成功连接之后就会进入这个方法
          @Override
          public void operationComplete(ChannelFuture future) throws Exception {

              Channel channel = future.channel();
              channel.writeAndFlush("operationComplete");
              //只有close之后才会调用下面的关闭监听器
              channel.close(); //关闭channel,这个关闭方法也是**异步**的，所以也需要进行监听

              ChannelFuture closeFuture = channel.closeFuture();

              //关闭通道监听器
              closeFuture.addListener(new ChannelFutureListener() {
                  @Override
                  public void operationComplete(ChannelFuture future) throws Exception {
                      log.info("已经关闭channel");
                      //关闭group
                      eventLoopGroup.shutdownGracefully();
                  }
              });

          }
      });
```

##### Future&Promise

**Future都是用线程池去返回得到的，所以JDK Future需要依赖线程池，Netty Future需要依赖于EventLoopGroup**


JDK Futhure和Netty Future、Netty Promise区别：

**Netty的Future继承与JDK的Future，Netty Promise又对Netty Future进行扩展**。

* JDK Future只能同步等待任务结束（或成功、或失败）才能得到结果,例如JDK Future的get是阻塞的获取结果
* Netty Future既阻塞的获取结果，也可以非阻塞的获取结果，阻塞就是get,非阻塞就是getNow。
* Netty Promise有Netty Future所有的功能且增加了几个方法，setSuccess、setFailure，而且脱离了任务独立存在，只作为两个线程间传递结果的容器。


> JDK Future

```java
      ExecutorService executorService = Executors.newFixedThreadPool(2); //创建一个固定大小的线程池
      //Callable有返回值。
      Future<String> future = executorService.submit(new Callable<String>() {
          @Override
          public String call() throws Exception {
              Thread.sleep(1000);
              return "hello";
          }
      });

      String res = future.get(); //get方法会阻塞，直到线程池的submit执行完毕，返回了future对象才会解除阻塞
      System.out.println(res);
      executorService.shutdown(); //关闭线程池
```

> Netty Future

```java
      EventLoopGroup eventLoopGroup = new NioEventLoopGroup(2);
      Future<String> future = eventLoopGroup.next().submit(new Callable<String>() {
          @Override
          public String call() throws Exception {
              Thread.sleep(1000);
              return "Netty Future";
          }
      });

//      String s1 = future.get(); //阻塞方法，这个方法和jdk的future一样
//      System.out.println(s1);

      String s2 = future.getNow(); //非阻塞方法，如果future没有立刻返回值则不会等待，直接返回null
      System.out.println(s2);
```

> Netty Promise

Promise相当于一个容器，可以用于存放各个线程中的结果，然后让其他线程去获取该结果

```java
      NioEventLoopGroup eventLoopGroup = new NioEventLoopGroup();
      EventLoop executors = eventLoopGroup.next();
      DefaultPromise<Integer> promise = new DefaultPromise<>(executors);
      new Thread(()->{

          try {
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
          promise.setSuccess(100);

      }).start();
      Integer res = promise.get();
      System.out.println(res);
```



##### Handler&Pipeline

**服务端：**

```java
              new ServerBootstrap()
              .group(new NioEventLoopGroup(),new NioEventLoopGroup(2))
              .channel(NioServerSocketChannel.class)
              .childHandler(new ChannelInitializer<SocketChannel>() {

                  //pipeline结构
                  //head->handle1->handle2->handle3->handle4->handle5->handle6->tail
                  //且为‘双向链表’，触发Inbound事件则会从head->tail一直走Inbound方法。
                  //触发Outbound事件则会从tail->head一直走Outbound方法。只有触发了对应事件才会走对应的方法。。。。。。
                  @Override
                  protected void initChannel(SocketChannel socketChannel) throws Exception {

                      socketChannel.pipeline().addLast(new StringDecoder());

                      //Inbound处理器
                      //为处理器取名字
                      socketChannel.pipeline().addLast("handle1",new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              log.warn(Thread.currentThread().getName()+"==>"+"handle1");
                              super.channelRead(ctx, msg); //向下传递
                          }
                      });

                      socketChannel.pipeline().addLast("handle2",new ChannelInboundHandlerAdapter(){
                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              log.warn(msg.toString());
                              log.warn(Thread.currentThread().getName()+"==>"+"handle2");
                              super.channelRead(ctx, msg); //向下传递
                          }
                      });

                      socketChannel.pipeline().addLast("handle3",new ChannelInboundHandlerAdapter(){
                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

                              //***不能用这种方法，client会收不到
//                              ByteBuffer buffer = StandardCharsets.UTF_8.encode("hello world");


                              //***用这种,记住*****一定要指定字符类型UTF-8***
                              ByteBuf byteBuf = ctx.alloc().buffer().writeBytes("hello".getBytes("utf-8"));
                              //发送数据，触发OutBound事件
                              socketChannel.writeAndFlush(byteBuf);

                              log.warn(Thread.currentThread().getName()+"==>"+"handle3");
                              super.channelRead(ctx, msg); //向下传递
                          }
                      });

                      //Outbound处理器
                      socketChannel.pipeline().addLast("handle4",new ChannelOutboundHandlerAdapter(){

                          @Override
                          public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                              log.warn(Thread.currentThread().getName()+"==>"+"handle4");
                              super.write(ctx, msg, promise);
                          }
                      });

                      socketChannel.pipeline().addLast("handle5",new ChannelOutboundHandlerAdapter(){

                          @Override
                          public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                              log.warn(Thread.currentThread().getName()+"==>"+"handle5");
                              super.write(ctx, msg, promise);
                          }
                      });

                      socketChannel.pipeline().addLast("handle6",new ChannelOutboundHandlerAdapter(){

                          @Override
                          public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) throws Exception {
                              log.warn(Thread.currentThread().getName()+"==>"+"handle6");
                              super.write(ctx, msg, promise);
                          }
                      });
                  }
              }).bind(8080);
```

**客户端：**

```java
      NioEventLoopGroup eventLoopGroup = new NioEventLoopGroup();
      ChannelFuture channelFuture = new Bootstrap()
              .group(eventLoopGroup)
              .channel(NioSocketChannel.class)
              .handler(new ChannelInitializer<Channel>() {
                  @Override
                  protected void initChannel(Channel ch) throws Exception {

                      ch.pipeline().addLast(new StringEncoder());
                      ch.pipeline().addLast(new LoggingHandler());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              System.out.println("--------------"+msg.toString());
                              super.channelRead(ctx, msg);
                          }
                      });

                  }
              }).connect("localhost", 8080);

      channelFuture.addListener(new ChannelFutureListener() {
          @Override
          public void operationComplete(ChannelFuture future) throws Exception {

              Channel channel = future.channel();
              channel.writeAndFlush("client-----");
//              channel.close();
//              ChannelFuture closeFuture = channel.closeFuture();
//              closeFuture.addListener(new ChannelFutureListener() {
//                  @Override
//                  public void operationComplete(ChannelFuture future) throws Exception {
//                      eventLoopGroup.shutdownGracefully();
//                  }
//              });
          }
      });
```

通过channel.pipeline().addLast(name, handler)添加handler时，记得给handler取名字。这样可以调用pipeline的addAfter、addBefore等方法更灵活地向pipeline中添加handler.

**handler需要放入通道的pipeline中，才能根据放入顺序来使用handler:**
* pipeline是结构是一个带有head与tail指针的双向链表，其中的节点为handler处理器
  * 要通过ctx.fireChannelRead(msg)等方法，将当前handler的处理结果传递给下一个handler
* 当有入站（Inbound）操作时，会从head开始向tail方向调用handler，直到handler不是处理Inbound操作为止
* 当有出站（Outbound）操作时，会从tail开始向head方向调用handler，直到handler不是处理Outbound操作为止

**结构图：**

![p](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/p.png)


##### ByteBuf

> 创建ByteBuf

```java
      //创建ByteBuf
      ByteBuf byteBuf = ByteBufAllocator.DEFAULT.buffer(10);

      log(byteBuf);

      StringBuffer stringBuffer = new StringBuffer();

    for (int i = 0; i < 50; i++) {
      stringBuffer.append('1');
    }
    byteBuf.writeBytes(stringBuffer.toString().getBytes("utf-8"));
    log(byteBuf);
```

**运行结果：**

```java
read index:0 write index:0 capacity:10

read index:0 write index:50 capacity:64
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 31 31 31 31 31 31 31 31 31 31 31 31 31 31 31 31 |1111111111111111|
|00000010| 31 31 31 31 31 31 31 31 31 31 31 31 31 31 31 31 |1111111111111111|
|00000020| 31 31 31 31 31 31 31 31 31 31 31 31 31 31 31 31 |1111111111111111|
|00000030| 31 31                                           |11              |
+--------+-------------------------------------------------+----------------+

Process finished with exit code 0
```

**根据打印的capacity可知ByteBuf是会自动扩容的，而NIO的ByteBuffer是不能超出容量的。**

```java
public abstract class AbstractByteBufAllocator implements ByteBufAllocator {
    static final int DEFAULT_INITIAL_CAPACITY = 256; //默认初始化容量
    static final int DEFAULT_MAX_CAPACITY = Integer.MAX_VALUE; //最大容量
    static final int DEFAULT_MAX_COMPONENTS = 16;
```

**ByteBuf通过ByteBufAllocator选择allocator并调用对应的buffer()方法来创建的，默认使用直接内存作为ByteBuf，容量为256个字节，可以指定初始容量的大小**

**如果在handler中创建ByteBuf，建议使用ChannelHandlerContext ctx.alloc().buffer()来创建**

> 3种创建池化的ByteBuf方式

```java
      ByteBuf byteBuf1 = ByteBufAllocator.DEFAULT.buffer(10); //默认创建的是‘’直接内存‘’的ByteBuf

      ByteBuf byteBuf2 = ByteBufAllocator.DEFAULT.heapBuffer(10);//指定创建‘’堆内存‘’的ByteBuf

      ByteBuf byteBuf3 = ByteBufAllocator.DEFAULT.directBuffer(10);//指定创建‘’直接内存‘’的ByteBuf
```

> 查看当前ByteBuf对象类型

```java
      ByteBuf byteBuf1 = ByteBufAllocator.DEFAULT.buffer(10); //默认创建的是‘’直接内存‘’的ByteBuf

      ByteBuf byteBuf2 = ByteBufAllocator.DEFAULT.heapBuffer(10);//指定创建‘’堆内存‘’的ByteBuf

      ByteBuf byteBuf3 = ByteBufAllocator.DEFAULT.directBuffer(10);//指定创建‘’直接内存‘’的ByteBuf

      System.out.println(byteBuf1.getClass());
      System.out.println(byteBuf2.getClass());
      System.out.println(byteBuf3.getClass());
```

**输出结果：**

```java
class io.netty.buffer.PooledUnsafeDirectByteBuf
class io.netty.buffer.PooledUnsafeHeapByteBuf
class io.netty.buffer.PooledUnsafeDirectByteBuf
```

> 池化和非池化

* Netty4.1**之前**默认是非池化
* Netty4.1**之后**默认是池化，但是Android平台默认是**非池化**

**池化优点：**

* 本质上池化的意义就是可重用ByteBuf
  * 没有池化的话每次需要使用ByteBuf都要重新申请内存。即使是堆内存，释放内存也会增大GC的压力
  * 有了池化，则可以重用池中ByteBuf实例，并且采用了与jemalloc类似的内存分配算法提升分配效率
  * 高并发下，池化更节约内存，减少内存溢出的可能。

> IDEA IDE如何设置为非池化

**只需要在IDEA IDE的VM options里面设置下面一段代码即可：**
```text
-Dio.netty.allocator.type={unpooled|pooled}
```

> ByteBuf组成

* 最大容量与当前容量
  * 在构造ByteBuf时，可传入两个参数，分别代表初始容量和最大容量，若未传入第二个参数（最大容量），最大容量默认为Integer.MAX_VALUE
  * 当ByteBuf容量无法容纳所有数据时，会进行扩容操作，若超出最大容量，会抛出**java.lang.IndexOutOfBoundsException**异常
* 读写操作不同于ByteBuffer只用position进行控制，ByteBuf分别由**读指针**和**写指针**两个指针控制。进行**读写操作**时，**无需进行模式的切换**
  * 读指针前的部分被称为废弃部分，是已经读过的内容
  * 读指针与写指针之间的空间称为可读部分
  * 写指针与当前容量之间的空间称为可写部分

![20210423143030](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/20210423143030.png)


> ByteBuf写入

```java
      ByteBuf byteBuf1 = ByteBufAllocator.DEFAULT.buffer(10); //默认创建的是‘’直接内存‘’的ByteBuf
      
      byteBuf1.writeBytes("hello".getBytes("utf-8"));
```

**write和set方法的区别：**

ByteBuf中**set开头**的一系列方法，也可以写入数据，但**不会改变写指针位置**


> ByteBuf的扩容机制

**当ByteBuf中的当前容量无法容纳写入的数据时，会自动进行扩容**

**触发扩容：**
```java
      ByteBuf byteBuf1 = ByteBufAllocator.DEFAULT.buffer(10); //默认创建的是‘’直接内存‘’的ByteBuf
      log(byteBuf1);
      byteBuf1.writeBytes("helloaaaaaaaa".getBytes("utf-8"));
      log(byteBuf1);
```

**结果：**

```java
read index:0 write index:0 capacity:10

read index:0 write index:13 capacity:16
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 61 61 61 61 61 61 61 61          |helloaaaaaaaa   |
+--------+-------------------------------------------------+----------------+

```


**扩容机制如下：**

有两种情况：

* 写入后的数据小于512字节
  * 这种情况会选择使用16的整数倍进行扩容，比如写入后的数据是14字节，则16*1为最小整数倍，则会扩容到16字节
* 写入后的数据大于512字节
  * 这种情况会以2的n次方扩容，例如写入后的数据是600字节，此时大于512字节，那么容纳它的容量为2的10次方，因为2的9次方是512容纳不了，所以会扩容到1024字节
  * 如果扩容后的大小大于**maxCapacity**，则会抛出**java.lang.IndexOutOfBoundsException**异常

> ByteBuf读取

**读取后会移动读指针**

```java
      ByteBuf byteBuf = ByteBufAllocator.DEFAULT.buffer(10);

      byteBuf.writeBytes("hello".getBytes("utf-8"));

      byte b[]=new byte[5];
      byteBuf.readBytes(b);

     System.out.println(Arrays.toString(b));
```
ByteBuf以**get开头**的方法，这些方法**不会改变读指针的位置**

##### ByteBuf日志工具类

```java
    public class ByteBufferUtil {
    private static final char[] BYTE2CHAR = new char[256];
    private static final char[] HEXDUMP_TABLE = new char[256 * 4];
    private static final String[] HEXPADDING = new String[16];
    private static final String[] HEXDUMP_ROWPREFIXES = new String[65536 >>> 4];
    private static final String[] BYTE2HEX = new String[256];
    private static final String[] BYTEPADDING = new String[16];

    static {
        final char[] DIGITS = "0123456789abcdef".toCharArray();
        for (int i = 0; i < 256; i++) {
            HEXDUMP_TABLE[i << 1] = DIGITS[i >>> 4 & 0x0F];
            HEXDUMP_TABLE[(i << 1) + 1] = DIGITS[i & 0x0F];
        }

        int i;

        // Generate the lookup table for hex dump paddings
        for (i = 0; i < HEXPADDING.length; i++) {
            int padding = HEXPADDING.length - i;
            StringBuilder buf = new StringBuilder(padding * 3);
            for (int j = 0; j < padding; j++) {
                buf.append("   ");
            }
            HEXPADDING[i] = buf.toString();
        }

        // Generate the lookup table for the start-offset header in each row (up to 64KiB).
        for (i = 0; i < HEXDUMP_ROWPREFIXES.length; i++) {
            StringBuilder buf = new StringBuilder(12);
            buf.append(StringUtil.NEWLINE);
            buf.append(Long.toHexString(i << 4 & 0xFFFFFFFFL | 0x100000000L));
            buf.setCharAt(buf.length() - 9, '|');
            buf.append('|');
            HEXDUMP_ROWPREFIXES[i] = buf.toString();
        }

        // Generate the lookup table for byte-to-hex-dump conversion
        for (i = 0; i < BYTE2HEX.length; i++) {
            BYTE2HEX[i] = ' ' + StringUtil.byteToHexStringPadded(i);
        }

        // Generate the lookup table for byte dump paddings
        for (i = 0; i < BYTEPADDING.length; i++) {
            int padding = BYTEPADDING.length - i;
            StringBuilder buf = new StringBuilder(padding);
            for (int j = 0; j < padding; j++) {
                buf.append(' ');
            }
            BYTEPADDING[i] = buf.toString();
        }

        // Generate the lookup table for byte-to-char conversion
        for (i = 0; i < BYTE2CHAR.length; i++) {
            if (i <= 0x1f || i >= 0x7f) {
                BYTE2CHAR[i] = '.';
            } else {
                BYTE2CHAR[i] = (char) i;
            }
        }
    }

    /**
     * 打印所有内容
     * @param buffer
     */
    public static void debugAll(ByteBuffer buffer) {
        int oldlimit = buffer.limit();
        buffer.limit(buffer.capacity());
        StringBuilder origin = new StringBuilder(256);
        appendPrettyHexDump(origin, buffer, 0, buffer.capacity());
        System.out.println("+--------+-------------------- all ------------------------+----------------+");
        System.out.printf("position: [%d], limit: [%d]\n", buffer.position(), oldlimit);
        System.out.println(origin);
        buffer.limit(oldlimit);
    }

    /**
     * 打印可读取内容
     * @param buffer
     */
    public static void debugRead(ByteBuffer buffer) {
        StringBuilder builder = new StringBuilder(256);
        appendPrettyHexDump(builder, buffer, buffer.position(), buffer.limit() - buffer.position());
        System.out.println("+--------+-------------------- read -----------------------+----------------+");
        System.out.printf("position: [%d], limit: [%d]\n", buffer.position(), buffer.limit());
        System.out.println(builder);
    }

    private static void appendPrettyHexDump(StringBuilder dump, ByteBuffer buf, int offset, int length) {
        if (MathUtil.isOutOfBounds(offset, length, buf.capacity())) {
            throw new IndexOutOfBoundsException(
                    "expected: " + "0 <= offset(" + offset + ") <= offset + length(" + length
                            + ") <= " + "buf.capacity(" + buf.capacity() + ')');
        }
        if (length == 0) {
            return;
        }
        dump.append(
                "         +-------------------------------------------------+" +
                        StringUtil.NEWLINE + "         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |" +
                        StringUtil.NEWLINE + "+--------+-------------------------------------------------+----------------+");

        final int startIndex = offset;
        final int fullRows = length >>> 4;
        final int remainder = length & 0xF;

        // Dump the rows which have 16 bytes.
        for (int row = 0; row < fullRows; row++) {
            int rowStartIndex = (row << 4) + startIndex;

            // Per-row prefix.
            appendHexDumpRowPrefix(dump, row, rowStartIndex);

            // Hex dump
            int rowEndIndex = rowStartIndex + 16;
            for (int j = rowStartIndex; j < rowEndIndex; j++) {
                dump.append(BYTE2HEX[getUnsignedByte(buf, j)]);
            }
            dump.append(" |");

            // ASCII dump
            for (int j = rowStartIndex; j < rowEndIndex; j++) {
                dump.append(BYTE2CHAR[getUnsignedByte(buf, j)]);
            }
            dump.append('|');
        }

        // Dump the last row which has less than 16 bytes.
        if (remainder != 0) {
            int rowStartIndex = (fullRows << 4) + startIndex;
            appendHexDumpRowPrefix(dump, fullRows, rowStartIndex);

            // Hex dump
            int rowEndIndex = rowStartIndex + remainder;
            for (int j = rowStartIndex; j < rowEndIndex; j++) {
                dump.append(BYTE2HEX[getUnsignedByte(buf, j)]);
            }
            dump.append(HEXPADDING[remainder]);
            dump.append(" |");

            // Ascii dump
            for (int j = rowStartIndex; j < rowEndIndex; j++) {
                dump.append(BYTE2CHAR[getUnsignedByte(buf, j)]);
            }
            dump.append(BYTEPADDING[remainder]);
            dump.append('|');
        }

        dump.append(StringUtil.NEWLINE +
                "+--------+-------------------------------------------------+----------------+");
    }

    private static void appendHexDumpRowPrefix(StringBuilder dump, int row, int rowStartIndex) {
        if (row < HEXDUMP_ROWPREFIXES.length) {
            dump.append(HEXDUMP_ROWPREFIXES[row]);
        } else {
            dump.append(StringUtil.NEWLINE);
            dump.append(Long.toHexString(rowStartIndex & 0xFFFFFFFFL | 0x100000000L));
            dump.setCharAt(dump.length() - 9, '|');
            dump.append('|');
        }
    }

    public static short getUnsignedByte(ByteBuffer buffer, int index) {
        return (short) (buffer.get(index) & 0xFF);
    }

    public static void log(ByteBuf buffer) {
        int length = buffer.readableBytes();
        int rows = length / 16 + (length % 15 == 0 ? 0 : 1) + 4;
        StringBuilder buf = new StringBuilder(rows * 80 * 2)
                .append("read index:").append(buffer.readerIndex())
                .append(" write index:").append(buffer.writerIndex())
                .append(" capacity:").append(buffer.capacity())
                .append(NEWLINE);
        io.netty.buffer.ByteBufUtil.appendPrettyHexDump(buf, buffer);
        System.out.println(buf.toString());
    }
}
```

> ByteBuf的释放

由于ByteBuf中有堆外内存（直接内存）的实现，**堆外内存**最好是**手动来释放**，而不是等GC来进行垃圾回收。
* UnpooledHeapByteBuf使用的是JVM内存，只需等GC回收内存即可。
* UnpooledDirectByteBuf使用的是直接内存，需要特殊的方法来回收内存
* PooledByteBuf和它的子类使用了池化机制，需要更复杂的规则来回收内存


Netty这里采用了**引用计数法**来控制**回收内存**，每个**ByteBuf**都实现了**ReferenceCounted**接口

**具体如下：**

* **新创建**的ByteBuf**默认计数为1**
* 调用**release方法**会使**计数-1**，**如果计数为0，则内存将会被回收**。
* 调用**retain方法**会使**计数+1**，表示调用者没用完之前，其它handler即使调用了release也不会造成回收
* 当计数为 0 时，底层内存会被回收，这时即使ByteBuf对象还在，其各个方法均无法正常使用。

**ByteBuf内存释放规则是：谁最后使用这块内存，谁就要调用release方法进行释放。**

* 入站(Inbound处理器链)ByteBuf处理原则：
  * 可以遵循谁最后使用内存谁就release。也可以让尾释放内存。
    * 我们知道Inbound是从head->tail，所以tail是入站的终点，**TailContext**也会处理**内存释放**的问题。
* 出站(Outbound处理器链)ByteBuf处理原则
  * 可以遵循谁最后使用内存谁就release。也可以让头释放内存。
  * 我们知道Outbound是从tail->head，所以head是出站的终点，**HeadContext**也会处理**内存释放**的问题。
* 有时候不清楚ByteBuf的计数是多少次，但又必须彻底释放，可以**循环调用**release直到返回true

```java
while (!buffer.release()) {}
```

> 内存释放源码

```java
public interface ReferenceCounted {
    ReferenceCounted retain();
    /**
         * Decreases the reference count by {@code 1} and deallocates this object if the reference count reaches at
         * {@code 0}.
         *
         * @return {@code true} if and only if the reference count became {@code 0} and this object has been deallocated
         */
    boolean release();
}
```

**从注释可以看出，让release成功释放内存后将会返回true。**

**头尾释放内存源码：**

```java
    /**
     * Called once a message hit the end of the {@link ChannelPipeline} without been handled by the user
     * in {@link ChannelInboundHandler#channelRead(ChannelHandlerContext, Object)}. This method is responsible
     * to call {@link ReferenceCountUtil#release(Object)} on the given msg at some point.
     */
    protected void onUnhandledInboundMessage(Object msg) {
        try {
            logger.debug(
                    "Discarded inbound message {} that reached at the tail of the pipeline. " +
                            "Please check your pipeline configuration.", msg);
        } finally {
            ReferenceCountUtil.release(msg);
        }
    }
```

```java
    /**
     * Try to call {@link ReferenceCounted#release()} if the specified message implements {@link ReferenceCounted}.
     * If the specified message doesn't implement {@link ReferenceCounted}, this method does nothing.
     */
    public static boolean release(Object msg) {
        if (msg instanceof ReferenceCounted) {
            return ((ReferenceCounted) msg).release();
        }
        return false;
    }
```


> 使用被释放的内存会怎样

```java
     ByteBuf byteBuf = ByteBufAllocator.DEFAULT.buffer(10);

      byteBuf.writeBytes("helloWorld".getBytes("utf-8"));

      byteBuf.release(); //释放内存
      ByteBufferUtil.log(byteBuf);
```

**结果：**

```java
Exception in thread "main" io.netty.util.IllegalReferenceCountException: refCnt: 0
```


> 注意：一旦ByteBuf的计数到0，再进行retain也没用

```java
      ByteBuf byteBuf = ByteBufAllocator.DEFAULT.buffer(10);

      byteBuf.writeBytes("helloWorld".getBytes("utf-8"));

      byteBuf.release(); //-1
      byteBuf.retain(); //+1
      ByteBufferUtil.log(byteBuf);
```

**结果**

```java
Exception in thread "main" io.netty.util.IllegalReferenceCountException: refCnt: 0, increment: 1
```

> 内存切片slice

```java
public abstract ByteBuf slice(int index, int length);
```

* ByteBuf的内存切片也是零拷贝的体现之一,切片后的内存还是原来ByteBuf的内存，过程没有发生过内存复制，切片后的 ByteBuf 维护独立的 read，write 指针.
* 切片后的ByteBuf需要调用retain使计数+1，防止原来的ByetBuf调用release释放内存导致切片的内存不可用。
* 修改原ByteBuf中的值，也会影响切片后得到的ByteBuf。

**代码案例：**

```java
      ByteBuf byteBuf = ByteBufAllocator.DEFAULT.buffer(10);

      byteBuf.writeBytes("helloWorld".getBytes("utf-8"));

      ByteBufferUtil.log(byteBuf);

      ByteBuf buf = byteBuf.slice(0, 5); //内存分片

      ByteBufferUtil.log(buf);

      System.out.println("---------------");

      buf.setByte(1,'g'); //修改分片内存的值

      //重新打印
      ByteBufferUtil.log(byteBuf);

      ByteBufferUtil.log(buf);
```

**结果：**

```java
read index:0 write index:10 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 57 6f 72 6c 64                   |helloWorld      |
+--------+-------------------------------------------------+----------------+
read index:0 write index:5 capacity:5
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f                                  |hello           |
+--------+-------------------------------------------------+----------------+
---------------
read index:0 write index:10 capacity:10
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 67 6c 6c 6f 57 6f 72 6c 64                   |hglloWorld      |
+--------+-------------------------------------------------+----------------+
read index:0 write index:5 capacity:5
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 67 6c 6c 6f                                  |hgllo           |
+--------+-------------------------------------------------+----------------+

Process finished with exit code 0

```

**结论：可以看出修改分片的内存的值，原内存也会受到影响，因为他们都是用同一块内存。**

> ByteBuf优势

* 池化思想-可以重用池中ByteBuf实例，更节约内存，减少内存溢出的可能
* 读写指针分离，不需要像 ByteBuffer一样**切换**读写模式
* 可以自动扩容
* 支持链式调用，使用更流畅
* 很多地方体现零拷贝，例如slice、duplicate、CompositeByteBuf


### Netty进阶

#### 粘包和半包/拆包问题

> 粘包问题演示

**服务器端：**
```java
  private static final Logger log= LoggerFactory.getLogger(NettyServer.class);

  public static void main(String[] args) {

      NioEventLoopGroup boss = new NioEventLoopGroup(1);
      NioEventLoopGroup worker = new NioEventLoopGroup(6);

      new ServerBootstrap()
              .group(boss,worker)
              .channel(NioServerSocketChannel.class)
              .childHandler(new ChannelInitializer<NioSocketChannel>() {
                  @Override
                  protected void initChannel(NioSocketChannel ch) throws Exception {

                      //不进行加解密不然展示不出粘包效果
//                      ch.pipeline().addLast(new StringDecoder());

                      ch.pipeline().addLast(new LoggingHandler());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {
                              log.info("客户端已成功连接服务器");
                              super.channelActive(ctx);
                          }

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

                              log.info("msg={}",msg);
                              super.channelRead(ctx, msg);
                          }
                      });


                  }
              }).bind(8080);
  }
```


**客户端：**

```java
  private static final Logger log= LoggerFactory.getLogger(NettyClient.class);

  public static void main(String[] args) {

    NioEventLoopGroup eventLoopGroup = new NioEventLoopGroup();
    ChannelFuture channelFuture = new Bootstrap()
            .group(eventLoopGroup)
            .channel(NioSocketChannel.class)
            .handler(new ChannelInitializer<Channel>() {
              @Override
              protected void initChannel(Channel ch) throws Exception {

                //不进行加解密不然展示不出粘包效果
//                ch.pipeline().addLast(new StringEncoder());
                ch.pipeline().addLast(new LoggingHandler());



              }
            }).connect("localhost", 8080);

    channelFuture.addListener(new ChannelFutureListener() {
      @Override
      public void operationComplete(ChannelFuture future) throws Exception {

        Channel channel = future.channel();

        ByteBuf byteBuf = channel.alloc().buffer(16);
        for (int i=0;i<10;i++){
          byteBuf.retain();
          byteBuf.writeBytes(("hello").getBytes("utf-8"));
          channel.writeAndFlush(byteBuf);
          byteBuf.clear();
        }

        channel.close();

        ChannelFuture closeFuture = channel.closeFuture();
        closeFuture.addListener(new ChannelFutureListener() {
          @Override
          public void operationComplete(ChannelFuture future) throws Exception {
            eventLoopGroup.shutdownGracefully();
          }
        });
      }
    });

  }
```

**服务器端输出结果：**

```java
16:00:37.869 [nioEventLoopGroup-3-1] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0xa191631f, L:/127.0.0.1:8080 - R:/127.0.0.1:53693] READ: 50B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 68 65 6c 6c 6f 68 65 6c 6c 6f 68 |hellohellohelloh|
|00000010| 65 6c 6c 6f 68 65 6c 6c 6f 68 65 6c 6c 6f 68 65 |ellohellohellohe|
|00000020| 6c 6c 6f 68 65 6c 6c 6f 68 65 6c 6c 6f 68 65 6c |llohellohellohel|
|00000030| 6c 6f                                           |lo              |
+--------+-------------------------------------------------+----------------+
```

**可以看出原来我们是在客户端分10次发送，而服务器端却一下把10次的数据都粘在一起了，这就是粘包问题。**


> 半包问题展示

**服务器端：**

```java
  private static final Logger log= LoggerFactory.getLogger(NettyServer.class);

  public static void main(String[] args) {

      NioEventLoopGroup boss = new NioEventLoopGroup(1);
      NioEventLoopGroup worker = new NioEventLoopGroup(6);

    new ServerBootstrap()
        .group(boss, worker)
        .channel(NioServerSocketChannel.class)
        // 半包问题：例如，发送方发送100字节数据，而接收方最多只能接收30字节数据，这就是半包问题
            //option(ChannelOption.SO_RCVBUF,10),调整接收缓冲区大小（滑动窗口）
        .option(ChannelOption.SO_RCVBUF,10)
        .childHandler(
            new ChannelInitializer<NioSocketChannel>() {
              @Override
              protected void initChannel(NioSocketChannel ch) throws Exception {

                // 不进行加解密不然展示不出粘包效果
                //                      ch.pipeline().addLast(new StringDecoder());

                ch.pipeline().addLast(new LoggingHandler());

                ch.pipeline()
                    .addLast(
                        new ChannelInboundHandlerAdapter() {

                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {
                            log.info("客户端已成功连接服务器");
                            super.channelActive(ctx);
                          }

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg)
                              throws Exception {

                            log.info("msg={}", msg);
                            super.channelRead(ctx, msg);
                          }
                        });
              }
            })
        .bind(8080);
  }
```

**只需使用这个方法即可**
* option(ChannelOption.SO_RCVBUF,10)

**option(ChannelOption.SO_RCVBUF,10),调整接收缓冲区大小。由于接收缓存区的大小<发送方发送的数据大小，所以产生了半包问题。**


> 现象分析

**粘包：**

* 产生现象
  * 第一次发送abc，第二次发送def，接收到的是一整个abcdef
* 原因
  * Netty层
    * 接收方的**接收缓冲区太大**，Netty的接收缓冲区默认是1024字节
  * 网络层
    * TCP滑动窗口：假如发送方发送100字节数据,而滑动窗口缓冲区可容纳>100字节数据，这时候就会出现粘包问题。
    * Nagle 算法：会造成粘包

**半包/拆包：**

* 产生现象
  * 发送abcdef数据，接收方第一次收到ab,第二次收到cd，第三次收到ef
* 原因
  * Netty层
    * 接收方的**接收缓冲区太小**，发送方的**数据过大**，导致接收方无法一次接收下所有数据，就会半包/拆包
  * 网络层
    * 滑动窗口：假设接收方的窗口只剩了128bytes，发送方的报文大小是256bytes，这时接收方窗口中无法容纳发送方的全部报文，发送方只能先发送前128bytes，等待ack后才能发送剩余部分，这就造成了半包
  * 数据链路层
    * MSS 限制：当发送的数据超过MSS限制后，会将数据切分发送，就会造成半包

**发送这些问题的本质：因为 TCP 是流式协议，消息无边界**



#### 粘包和半包/拆包解决方案

##### 短连接

**短连接：即每发送一条数据就重新连接再次发送，反复此操作。**

**短连接的缺点是显而易见的，每次发送一条数据都要重新连接，这样会大大的浪费时间，因为连接是需要时间的。**

客户端每次向服务器发送数据以后，就与服务器断开连接，此时的消息边界为连接建立到连接断开。
这时**便无需使用**滑动窗口等技术来缓冲数据，则**不会发生粘包**现象。
但如果一次性数据发送过多，接收方无法一次性容纳所有数据，还是会发生半包现象，所以**短链接无法解决半包现象**


> 采用短连接解决粘包代码

**服务端：**

```java
  private static final Logger log = LoggerFactory.getLogger(NettyServer.class);

  public static void main(String[] args) {

    NioEventLoopGroup boss = new NioEventLoopGroup(1);
    NioEventLoopGroup worker = new NioEventLoopGroup(6);

    new ServerBootstrap()
        .group(boss, worker)
        .channel(NioServerSocketChannel.class)
        .childHandler(
            new ChannelInitializer<NioSocketChannel>() {
              @Override
              protected void initChannel(NioSocketChannel ch) throws Exception {

                ch.pipeline().addLast(new LoggingHandler());

                ch.pipeline()
                    .addLast(
                        new ChannelInboundHandlerAdapter() {

                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {
                            log.info("客户端已成功连接服务器");
                            super.channelActive(ctx);
                          }

                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              log.info("msg="+msg);
                              super.channelRead(ctx, msg);
                            }
                        });
              }
            })
        .bind(8080);
  }
```

**客户端：**

```java
  private static final Logger log= LoggerFactory.getLogger(NettyClient.class);

  public static void main(String[] args) {

    //采用短连接解决“”粘包“”问题，无法解决半包问题
    for (int i = 0; i < 10; i++) {
      sendMessage("hello");
    }


  }


  public static void sendMessage(String msg){

     NioEventLoopGroup eventLoopGroup = new NioEventLoopGroup();
     ChannelFuture channelFuture = new Bootstrap()
            .group(eventLoopGroup)
            .channel(NioSocketChannel.class)
            .handler(new ChannelInitializer<Channel>() {
              @Override
              protected void initChannel(Channel ch) throws Exception {

                //不进行加解密不然展示不出粘包效果
//                ch.pipeline().addLast(new StringEncoder());
                ch.pipeline().addLast(new LoggingHandler());

                ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
                  @Override
                  public void channelActive(ChannelHandlerContext ctx) throws Exception {
                    ByteBuf buffer = ctx.alloc().buffer(16);
                    buffer.writeBytes(msg.getBytes("utf-8"));
                    ch.writeAndFlush(buffer);
                    ch.close();
                    ChannelFuture closeFuture = ch.closeFuture();
                    closeFuture.addListener(new ChannelFutureListener() {
                      @Override
                      public void operationComplete(ChannelFuture future) throws Exception {
                        eventLoopGroup.shutdownGracefully();
                      }
                    });
                  }
                });

              }
            }).connect("localhost", 8080);


  }
```

##### 定长解码器

客户端于服务器约定一个最大长度，保证客户端每次发送的数据长度都不会大于该长度。若发送数据长度不足则需要补齐至该长度。
服务器接收数据时，将接收到的数据按照约定的最大长度进行拆分，即使发送过程中产生了粘包，也可以通过定长解码器将数据正确地进行拆分。服务端需要用到**FixedLengthFrameDecoder**对数据进行定长解码

##### 行解码器(推荐)

**对于其他解码器，我还是更喜欢行解码器。行解码器主要是靠分隔符\n来判断行进行解码，不过需要进行限制长度，以免服务器一直搜索\n造成卡死。**

> 改造前的粘包代码

**服务端：**

```java
      NioEventLoopGroup boss = new NioEventLoopGroup(1);
      NioEventLoopGroup worker = new NioEventLoopGroup(6);
      new ServerBootstrap()
              .group(boss,worker)
              .channel(NioServerSocketChannel.class)
              .childHandler(new ChannelInitializer<NioSocketChannel>() {
                  @Override
                  protected void initChannel(NioSocketChannel ch) throws Exception {

                      ch.pipeline().addLast(new LoggingHandler());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              log.info("msg={}",msg);
                              super.channelRead(ctx, msg);
                          }
                      });

                  }
              }).bind(8080);
```

**客户端：**

```java
     NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
      try{
      new Bootstrap()
              .group(nioEventLoopGroup)
              .channel(NioSocketChannel.class)
              .handler(new ChannelInitializer<Channel>() {
                  @Override
                  protected void initChannel(Channel ch) throws Exception {

                      ch.pipeline().addLast(new LoggingHandler());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {

                              ByteBuf buffer = ctx.alloc().buffer(16);

                              for(int i=0;i<10;i++){
                                  buffer.retain();
                                  buffer.writeBytes("hello world".getBytes("utf-8"));
                                  ctx.channel().writeAndFlush(buffer);
                              }

                              ch.close();//关闭Channel
                              ChannelFuture closeFuture = ch.closeFuture();
                              closeFuture.addListener(new ChannelFutureListener() {
                                  @Override
                                  public void operationComplete(ChannelFuture future) throws Exception {
                                      nioEventLoopGroup.shutdownGracefully();
                                  }
                              });
                          }
                      });
                  }
              }).connect("localhost",8080);
      }catch (Exception e){
          e.printStackTrace();
      }
```

**结果：**

```java
13:37:15.286 [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x36ce6c5f, L:/127.0.0.1:8080 - R:/127.0.0.1:64550] READ: 110B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64 68 65 6c 6c 6f |hello worldhello|
|00000010| 20 77 6f 72 6c 64 68 65 6c 6c 6f 20 77 6f 72 6c | worldhello worl|
|00000020| 64 68 65 6c 6c 6f 20 77 6f 72 6c 64 68 65 6c 6c |dhello worldhell|
|00000030| 6f 20 77 6f 72 6c 64 68 65 6c 6c 6f 20 77 6f 72 |o worldhello wor|
|00000040| 6c 64 68 65 6c 6c 6f 20 77 6f 72 6c 64 68 65 6c |ldhello worldhel|
|00000050| 6c 6f 20 77 6f 72 6c 64 68 65 6c 6c 6f 20 77 6f |lo worldhello wo|
|00000060| 72 6c 64 68 65 6c 6c 6f 20 77 6f 72 6c 64       |rldhello world  |
+--------+-------------------------------------------------+----------------+
```


> 接收方使用行解码器改造后

**服务端：**

```java
  private static final Logger log= LoggerFactory.getLogger(NettyServer.class);

  public static void main(String[] args) {

      NioEventLoopGroup boss = new NioEventLoopGroup(1);
      NioEventLoopGroup worker = new NioEventLoopGroup(6);
      new ServerBootstrap()
              .group(boss,worker)
              .channel(NioServerSocketChannel.class)
              .childHandler(new ChannelInitializer<NioSocketChannel>() {
                  @Override
                  protected void initChannel(NioSocketChannel ch) throws Exception {

                      ch.pipeline().addLast(new LineBasedFrameDecoder(1024));//配置行解码器
                      
                      ch.pipeline().addLast(new LoggingHandler());
                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              log.info("msg={}",msg);
                              super.channelRead(ctx, msg);
                          }
                      });

                  }
              }).bind(8080);
```

**客户端：**

```java
 //把消息加工成可以被行解码器识别的消息
    private static String getMsg(String oldMsg){

        oldMsg+='\n';
        return oldMsg;
    }

  public static void main(String[] args) {

      NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
      try{
      new Bootstrap()
              .group(nioEventLoopGroup)
              .channel(NioSocketChannel.class)
              .handler(new ChannelInitializer<Channel>() {
                  @Override
                  protected void initChannel(Channel ch) throws Exception {

                      ch.pipeline().addLast(new LoggingHandler());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {

                              ByteBuf buffer = ctx.alloc().buffer(16);

                              for(int i=0;i<10;i++){
                                  buffer.retain();
                                  String msg = getMsg("hello world");
                                  buffer.writeBytes(msg.getBytes("utf-8"));
                                  ctx.channel().writeAndFlush(buffer);
                                  //清理缓存,防止数据堆叠
                                  buffer.clear();
                              }

                              ch.close();//关闭Channel
                              ChannelFuture closeFuture = ch.closeFuture();
                              closeFuture.addListener(new ChannelFutureListener() {
                                  @Override
                                  public void operationComplete(ChannelFuture future) throws Exception {
                                      nioEventLoopGroup.shutdownGracefully();
                                  }
                              });
                          }
                      });
                  }
              }).connect("localhost",8080);
      }catch (Exception e){
          e.printStackTrace();
      }

  }
```

**输出结果：**

```java
13:47:15.199 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] REGISTERED
13:47:15.199 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] ACTIVE
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ: 11B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64                |hello world     |
+--------+-------------------------------------------------+----------------+
13:47:15.224 [nioEventLoopGroup-3-4] INFO com.netty.netty.high.demo3.NettyServer - msg=PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 12, cap: 2048))
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 12, cap: 2048)) that reached at the tail of the pipeline. Please check your pipeline configuration.
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, NettyServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835].
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ COMPLETE
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ: 11B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64                |hello world     |
+--------+-------------------------------------------------+----------------+
13:47:15.224 [nioEventLoopGroup-3-4] INFO com.netty.netty.high.demo3.NettyServer - msg=PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 24, cap: 2048))
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 24, cap: 2048)) that reached at the tail of the pipeline. Please check your pipeline configuration.
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, NettyServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835].
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ: 11B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64                |hello world     |
+--------+-------------------------------------------------+----------------+
13:47:15.224 [nioEventLoopGroup-3-4] INFO com.netty.netty.high.demo3.NettyServer - msg=PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 24, widx: 24, cap: 2048))
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 24, widx: 24, cap: 2048)) that reached at the tail of the pipeline. Please check your pipeline configuration.
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, NettyServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835].
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ COMPLETE
13:47:15.224 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ: 11B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64                |hello world     |
+--------+-------------------------------------------------+----------------+
13:47:15.225 [nioEventLoopGroup-3-4] INFO com.netty.netty.high.demo3.NettyServer - msg=PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 24, cap: 1024))
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 24, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, NettyServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835].
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ: 11B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64                |hello world     |
+--------+-------------------------------------------------+----------------+
13:47:15.225 [nioEventLoopGroup-3-4] INFO com.netty.netty.high.demo3.NettyServer - msg=PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 24, widx: 24, cap: 1024))
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 24, widx: 24, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, NettyServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835].
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ COMPLETE
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ: 11B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64                |hello world     |
+--------+-------------------------------------------------+----------------+
13:47:15.225 [nioEventLoopGroup-3-4] INFO com.netty.netty.high.demo3.NettyServer - msg=PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 36, cap: 1024))
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 36, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, NettyServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835].
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ: 11B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64                |hello world     |
+--------+-------------------------------------------------+----------------+
13:47:15.225 [nioEventLoopGroup-3-4] INFO com.netty.netty.high.demo3.NettyServer - msg=PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 24, widx: 36, cap: 1024))
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 24, widx: 36, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, NettyServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835].
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ: 11B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64                |hello world     |
+--------+-------------------------------------------------+----------------+
13:47:15.225 [nioEventLoopGroup-3-4] INFO com.netty.netty.high.demo3.NettyServer - msg=PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 36, widx: 36, cap: 1024))
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 36, widx: 36, cap: 1024)) that reached at the tail of the pipeline. Please check your pipeline configuration.
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, NettyServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835].
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ COMPLETE
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ: 11B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64                |hello world     |
+--------+-------------------------------------------------+----------------+
13:47:15.225 [nioEventLoopGroup-3-4] INFO com.netty.netty.high.demo3.NettyServer - msg=PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 24, cap: 512))
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message PooledSlicedByteBuf(ridx: 0, widx: 11, cap: 11/11, unwrapped: PooledUnsafeDirectByteBuf(ridx: 12, widx: 24, cap: 512)) that reached at the tail of the pipeline. Please check your pipeline configuration.
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LineBasedFrameDecoder#0, LoggingHandler#0, NettyServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835].
13:47:15.225 [nioEventLoopGroup-3-4] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x2596d7b1, L:/127.0.0.1:8080 - R:/127.0.0.1:64835] READ: 11B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 68 65 6c 6c 6f 20 77 6f 72 6c 64                |hello world     |
+--------+-------------------------------------------------+----------------+
```

**可以看出已经解决了粘包问题。**

##### 自定义分隔符解码器

**核心类DelimiterBasedFrameDecoder**
```java
    /**
     * Creates a new instance.
     *
     * @param maxFrameLength  the maximum length of the decoded frame.
     *                        A {@link TooLongFrameException} is thrown if
     *                        the length of the frame exceeds this value.
     * @param delimiter  the delimiter
     */
    public DelimiterBasedFrameDecoder(int maxFrameLength, ByteBuf delimiter) {
        this(maxFrameLength, true, delimiter);
    }
```

**服务端：**

```java
  private static final Logger log= LoggerFactory.getLogger(NettyServer.class);

  public static void main(String[] args) {

      NioEventLoopGroup boss = new NioEventLoopGroup(1);
      NioEventLoopGroup worker = new NioEventLoopGroup(6);
      new ServerBootstrap()
              .group(boss,worker)
              .channel(NioServerSocketChannel.class)
              .childHandler(new ChannelInitializer<NioSocketChannel>() {
                  @Override
                  protected void initChannel(NioSocketChannel ch) throws Exception {

                      ByteBuf delimiter = ch.alloc().buffer(6);
                      delimiter.writeBytes("\r".getBytes("utf-8")); //自定义分隔符
                      ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024,delimiter));

                      ch.pipeline().addLast(new LoggingHandler());
                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              log.info("msg={}",msg);
                              super.channelRead(ctx, msg);
                          }
                      });

                  }
              }).bind(8080);
  }
```

**客户端：**

```java
 //把消息加工成可以被行解码器识别的消息
    private static String getMsg(String oldMsg){

        oldMsg+='\r';
        return oldMsg;
    }

  public static void main(String[] args) {

      NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
      try{
      new Bootstrap()
              .group(nioEventLoopGroup)
              .channel(NioSocketChannel.class)
              .handler(new ChannelInitializer<Channel>() {
                  @Override
                  protected void initChannel(Channel ch) throws Exception {

                      ch.pipeline().addLast(new LoggingHandler());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {

                              ByteBuf buffer = ctx.alloc().buffer(16);

                              for(int i=0;i<10;i++){
                                  buffer.retain();
                                  String msg = getMsg("hello world");
                                  buffer.writeBytes(msg.getBytes("utf-8"));
                                  ctx.channel().writeAndFlush(buffer);
                                  //清理缓存,防止数据堆叠
                                  buffer.clear();
                              }

                              ch.close();//关闭Channel
                              ChannelFuture closeFuture = ch.closeFuture();
                              closeFuture.addListener(new ChannelFutureListener() {
                                  @Override
                                  public void operationComplete(ChannelFuture future) throws Exception {
                                      nioEventLoopGroup.shutdownGracefully();
                                  }
                              });
                          }
                      });
                  }
              }).connect("localhost",8080);
      }catch (Exception e){
          e.printStackTrace();
      }

  }
```

#### Netty协议解析


##### Redis协议

**我们要用netty执行Redis命令就需要遵循Redis协议。**

> Redis协议格式

```text
*<参数数量> \r\n
$<参数1的字节数量> \r\n
<参数1的数据> \r\n
...
$<参数N的字节数量> \r\n
<参数N的数据> \r\n
```


> 使用Netty搭建一个Redis client

```java
/**
 * @author 游政杰
 * @date 2022/1/13
 * 模拟 Redis client
 */
public class RedisSender {

    //true为继续循环，false是退出循环
  private static ThreadLocal<Boolean> threadLocal=new ThreadLocal<Boolean>();
  private static final Logger log= LoggerFactory.getLogger(RedisSender.class);

  public static void main(String[] args) {

      //Netty“”客户端“”执行Redis命令
      NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
      try{

      new Bootstrap()
              .group(nioEventLoopGroup)
              .channel(NioSocketChannel.class)
              .handler(new ChannelInitializer<Channel>() {
                  @Override
                  protected void initChannel(Channel ch) throws Exception {

                      threadLocal.set(true); //默认是继续循环

                      ch.pipeline().addLast(new LoggingHandler());


                      ch.pipeline().addLast("handle1",new ChannelInboundHandlerAdapter(){

                          //连接成功之后调用
                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {

                              Scanner sc = new Scanner(System.in);
                              for (;;){
                                  if(!threadLocal.get()){
                                      System.out.println("退出成功");
                                      break;
                                  }
                                  printInfo();
                                  String sel = sc.next();
                                  switch (sel)
                                  {
                                      case "a":
                                          sc.nextLine(); //***上面会传下来字符，导致无法输入字符串，所以要加上这句，目的是吸收上面传下来的多余字符串
                                          System.out.println("请输入Redis命令[以单空格分隔]：");
                                          String redis_cmd = sc.nextLine();
                                          String decodeProtocol = decodeProtocol(redis_cmd);
                                          ByteBuf buffer = ctx.alloc().buffer(16);
                                          buffer.writeBytes(decodeProtocol.getBytes("utf-8"));
                                          ctx.writeAndFlush(buffer);
                                          buffer.clear();
                                          break;
                                      case "q":
                                          ch.close();
                                          ChannelFuture closeFuture = ch.closeFuture();
                                          closeFuture.addListener(new ChannelFutureListener() {
                                              @Override
                                              public void operationComplete(ChannelFuture future) throws Exception {
                                                  nioEventLoopGroup.shutdownGracefully();
                                                  threadLocal.set(false); //退出只需要设置为false即可
                                              }
                                          });
                                          break;
                                      default:
                                          System.out.println("无该选项");
                                          break;
                                  }
                              }


                          }
                      });
                  }
              }).connect("localhost",6379);
      }catch (Exception e){
          e.printStackTrace();
          nioEventLoopGroup.shutdownGracefully();
      }

  }

  private static void printInfo()
  {
    System.out.println("请输入以下字符选项：");
    System.out.println("输入a：执行Redis命令");
    System.out.println("输入q：退出");
  }


    /**
     * 协议解析
     * @param redis_cmd 命令
     * @return
     */
    //set myname abc
    //del key
    //get key
  private static synchronized String decodeProtocol(String redis_cmd){
      String delimiter1="*";
      String delimiter2="$";
      String delimiter3="\r\n";
      StringBuffer decodeCmd = new StringBuffer();//使用线程安全的StringBuffer
      List<String> cmd = Arrays.asList(redis_cmd.split(" "));
      decodeCmd.append(delimiter1+cmd.size()+delimiter3);
      cmd.forEach((e)->{

          decodeCmd.append(delimiter2+e.length()+delimiter3);
          decodeCmd.append(e+delimiter3);
      });

      return decodeCmd.toString();
  }

}

```
 
##### Http协议

**http服务端：**
```java
public class HttpServer {
    //http服务器
  public static void main(String[] args) {


      NioEventLoopGroup boss = new NioEventLoopGroup(1);
      NioEventLoopGroup worker = new NioEventLoopGroup(6);
      new ServerBootstrap()
              .group(boss,worker)
              .channel(NioServerSocketChannel.class)
              .childHandler(new ChannelInitializer<NioSocketChannel>() {

                  @Override
                  protected void initChannel(NioSocketChannel ch) throws Exception {

                      ch.pipeline().addLast(new LoggingHandler());
                      //netty自带的http协议转换
                      ch.pipeline().addLast(new HttpServerCodec());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

                              //msg有两种类型
                              //class io.netty.handler.codec.http.DefaultHttpRequest
                              //class io.netty.handler.codec.http.LastHttpContent$1

                              if(msg instanceof HttpRequest){

                                  HttpRequest request=(HttpRequest)msg;

                                  //输出响应DefaultFullHttpResponse(HttpVersion version, HttpResponseStatus status)
                                  DefaultFullHttpResponse response = new DefaultFullHttpResponse(request.protocolVersion(), HttpResponseStatus.OK);

                                  String s="hello 2022";
                                  byte b[]=s.getBytes("utf-8");
                                  //从请求头设置响应数据长度，以免浏览器空转
                                  //content_length是io.netty.handler.codec.http包下的类
                                  response.headers().setInt(CONTENT_LENGTH,b.length);

                                  //输出内容
                                  response.content().writeBytes(b);

                                  ctx.channel().writeAndFlush(response);
                              }

                              super.channelRead(ctx, msg);
                          }
                      });
                  }
              }).bind("localhost",8080);


  }
}

```

![p2.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/p2.png)


**输出结果：**

```java
16:03:05.785 [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486] REGISTERED
16:03:05.785 [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486] ACTIVE
16:03:05.788 [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486] READ: 756B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 47 45 54 20 2f 20 48 54 54 50 2f 31 2e 31 0d 0a |GET / HTTP/1.1..|
|00000010| 48 6f 73 74 3a 20 6c 6f 63 61 6c 68 6f 73 74 3a |Host: localhost:|
|00000020| 38 30 38 30 0d 0a 55 73 65 72 2d 41 67 65 6e 74 |8080..User-Agent|
|00000030| 3a 20 4d 6f 7a 69 6c 6c 61 2f 35 2e 30 20 28 57 |: Mozilla/5.0 (W|
|00000040| 69 6e 64 6f 77 73 20 4e 54 20 31 30 2e 30 3b 20 |indows NT 10.0; |
|00000050| 57 69 6e 36 34 3b 20 78 36 34 3b 20 72 76 3a 39 |Win64; x64; rv:9|
|00000060| 35 2e 30 29 20 47 65 63 6b 6f 2f 32 30 31 30 30 |5.0) Gecko/20100|
|00000070| 31 30 31 20 46 69 72 65 66 6f 78 2f 39 35 2e 30 |101 Firefox/95.0|
|00000080| 0d 0a 41 63 63 65 70 74 3a 20 74 65 78 74 2f 68 |..Accept: text/h|
|00000090| 74 6d 6c 2c 61 70 70 6c 69 63 61 74 69 6f 6e 2f |tml,application/|
|000000a0| 78 68 74 6d 6c 2b 78 6d 6c 2c 61 70 70 6c 69 63 |xhtml+xml,applic|
|000000b0| 61 74 69 6f 6e 2f 78 6d 6c 3b 71 3d 30 2e 39 2c |ation/xml;q=0.9,|
|000000c0| 69 6d 61 67 65 2f 61 76 69 66 2c 69 6d 61 67 65 |image/avif,image|
|000000d0| 2f 77 65 62 70 2c 2a 2f 2a 3b 71 3d 30 2e 38 0d |/webp,*/*;q=0.8.|
|000000e0| 0a 41 63 63 65 70 74 2d 4c 61 6e 67 75 61 67 65 |.Accept-Language|
|000000f0| 3a 20 7a 68 2d 43 4e 2c 7a 68 3b 71 3d 30 2e 38 |: zh-CN,zh;q=0.8|
|00000100| 2c 7a 68 2d 54 57 3b 71 3d 30 2e 37 2c 7a 68 2d |,zh-TW;q=0.7,zh-|
|00000110| 48 4b 3b 71 3d 30 2e 35 2c 65 6e 2d 55 53 3b 71 |HK;q=0.5,en-US;q|
|00000120| 3d 30 2e 33 2c 65 6e 3b 71 3d 30 2e 32 0d 0a 41 |=0.3,en;q=0.2..A|
|00000130| 63 63 65 70 74 2d 45 6e 63 6f 64 69 6e 67 3a 20 |ccept-Encoding: |
|00000140| 67 7a 69 70 2c 20 64 65 66 6c 61 74 65 0d 0a 43 |gzip, deflate..C|
|00000150| 6f 6e 6e 65 63 74 69 6f 6e 3a 20 6b 65 65 70 2d |onnection: keep-|
|00000160| 61 6c 69 76 65 0d 0a 43 6f 6f 6b 69 65 3a 20 48 |alive..Cookie: H|
|00000170| 6d 5f 6c 76 74 5f 62 33 39 33 64 31 35 33 61 65 |m_lvt_b393d153ae|
|00000180| 62 32 36 62 34 36 65 39 34 33 31 66 61 62 61 66 |b26b46e9431fabaf|
|00000190| 30 66 36 31 39 30 3d 31 36 32 31 39 30 35 38 35 |0f6190=162190585|
|000001a0| 30 3b 20 49 64 65 61 2d 33 35 32 63 36 33 39 66 |0; Idea-352c639f|
|000001b0| 3d 31 32 37 64 31 61 65 35 2d 34 34 31 37 2d 34 |=127d1ae5-4417-4|
|000001c0| 61 62 36 2d 61 61 64 33 2d 36 32 36 62 66 38 36 |ab6-aad3-626bf86|
|000001d0| 34 62 62 62 33 3b 20 55 4d 5f 64 69 73 74 69 6e |4bbb3; UM_distin|
|000001e0| 63 74 69 64 3d 31 37 61 61 65 35 65 65 63 33 34 |ctid=17aae5eec34|
|000001f0| 35 31 30 2d 30 39 39 37 35 66 34 36 64 62 65 63 |510-09975f46dbec|
|00000200| 33 38 38 2d 34 63 33 65 32 35 37 61 2d 31 34 34 |388-4c3e257a-144|
|00000210| 30 30 30 2d 31 37 61 61 65 35 65 65 63 33 36 33 |000-17aae5eec363|
|00000220| 61 62 3b 20 43 4e 5a 5a 44 41 54 41 31 32 35 38 |ab; CNZZDATA1258|
|00000230| 35 36 36 39 36 33 3d 31 36 31 32 35 36 38 34 34 |566963=161256844|
|00000240| 38 2d 31 36 32 36 34 32 32 37 30 36 2d 25 37 43 |8-1626422706-%7C|
|00000250| 31 36 32 36 34 32 38 31 35 30 0d 0a 55 70 67 72 |1626428150..Upgr|
|00000260| 61 64 65 2d 49 6e 73 65 63 75 72 65 2d 52 65 71 |ade-Insecure-Req|
|00000270| 75 65 73 74 73 3a 20 31 0d 0a 53 65 63 2d 46 65 |uests: 1..Sec-Fe|
|00000280| 74 63 68 2d 44 65 73 74 3a 20 64 6f 63 75 6d 65 |tch-Dest: docume|
|00000290| 6e 74 0d 0a 53 65 63 2d 46 65 74 63 68 2d 4d 6f |nt..Sec-Fetch-Mo|
|000002a0| 64 65 3a 20 6e 61 76 69 67 61 74 65 0d 0a 53 65 |de: navigate..Se|
|000002b0| 63 2d 46 65 74 63 68 2d 53 69 74 65 3a 20 6e 6f |c-Fetch-Site: no|
|000002c0| 6e 65 0d 0a 53 65 63 2d 46 65 74 63 68 2d 55 73 |ne..Sec-Fetch-Us|
|000002d0| 65 72 3a 20 3f 31 0d 0a 43 61 63 68 65 2d 43 6f |er: ?1..Cache-Co|
|000002e0| 6e 74 72 6f 6c 3a 20 6d 61 78 2d 61 67 65 3d 30 |ntrol: max-age=0|
|000002f0| 0d 0a 0d 0a                                     |....            |
+--------+-------------------------------------------------+----------------+
16:03:05.789 [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486] WRITE: 49B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 48 54 54 50 2f 31 2e 31 20 32 30 30 20 4f 4b 0d |HTTP/1.1 200 OK.|
|00000010| 0a 63 6f 6e 74 65 6e 74 2d 6c 65 6e 67 74 68 3a |.content-length:|
|00000020| 20 31 30 0d 0a 0d 0a 68 65 6c 6c 6f 20 32 30 32 | 10....hello 202|
|00000030| 32                                              |2               |
+--------+-------------------------------------------------+----------------+
16:03:05.789 [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486] FLUSH
16:03:05.789 [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message DefaultHttpRequest(decodeResult: success, version: HTTP/1.1)
GET / HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:95.0) Gecko/20100101 Firefox/95.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: keep-alive
Cookie: Hm_lvt_b393d153aeb26b46e9431fabaf0f6190=1621905850; Idea-352c639f=127d1ae5-4417-4ab6-aad3-626bf864bbb3; UM_distinctid=17aae5eec34510-09975f46dbec388-4c3e257a-144000-17aae5eec363ab; CNZZDATA1258566963=1612568448-1626422706-%7C1626428150
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Cache-Control: max-age=0 that reached at the tail of the pipeline. Please check your pipeline configuration.
16:03:05.790 [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LoggingHandler#0, HttpServerCodec#0, HttpServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486].
16:03:05.790 [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message EmptyLastHttpContent that reached at the tail of the pipeline. Please check your pipeline configuration.
16:03:05.790 [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded message pipeline : [LoggingHandler#0, HttpServerCodec#0, HttpServer$1$1#0, DefaultChannelPipeline$TailContext#0]. Channel : [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486].
16:03:05.790 [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486] READ COMPLETE
16:03:05.809 [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486] READ: 693B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 47 45 54 20 2f 66 61 76 69 63 6f 6e 2e 69 63 6f |GET /favicon.ico|
|00000010| 20 48 54 54 50 2f 31 2e 31 0d 0a 48 6f 73 74 3a | HTTP/1.1..Host:|
|00000020| 20 6c 6f 63 61 6c 68 6f 73 74 3a 38 30 38 30 0d | localhost:8080.|
|00000030| 0a 55 73 65 72 2d 41 67 65 6e 74 3a 20 4d 6f 7a |.User-Agent: Moz|
|00000040| 69 6c 6c 61 2f 35 2e 30 20 28 57 69 6e 64 6f 77 |illa/5.0 (Window|
|00000050| 73 20 4e 54 20 31 30 2e 30 3b 20 57 69 6e 36 34 |s NT 10.0; Win64|
|00000060| 3b 20 78 36 34 3b 20 72 76 3a 39 35 2e 30 29 20 |; x64; rv:95.0) |
|00000070| 47 65 63 6b 6f 2f 32 30 31 30 30 31 30 31 20 46 |Gecko/20100101 F|
|00000080| 69 72 65 66 6f 78 2f 39 35 2e 30 0d 0a 41 63 63 |irefox/95.0..Acc|
|00000090| 65 70 74 3a 20 69 6d 61 67 65 2f 61 76 69 66 2c |ept: image/avif,|
|000000a0| 69 6d 61 67 65 2f 77 65 62 70 2c 2a 2f 2a 0d 0a |image/webp,*/*..|
|000000b0| 41 63 63 65 70 74 2d 4c 61 6e 67 75 61 67 65 3a |Accept-Language:|
|000000c0| 20 7a 68 2d 43 4e 2c 7a 68 3b 71 3d 30 2e 38 2c | zh-CN,zh;q=0.8,|
|000000d0| 7a 68 2d 54 57 3b 71 3d 30 2e 37 2c 7a 68 2d 48 |zh-TW;q=0.7,zh-H|
|000000e0| 4b 3b 71 3d 30 2e 35 2c 65 6e 2d 55 53 3b 71 3d |K;q=0.5,en-US;q=|
|000000f0| 30 2e 33 2c 65 6e 3b 71 3d 30 2e 32 0d 0a 41 63 |0.3,en;q=0.2..Ac|
|00000100| 63 65 70 74 2d 45 6e 63 6f 64 69 6e 67 3a 20 67 |cept-Encoding: g|
|00000110| 7a 69 70 2c 20 64 65 66 6c 61 74 65 0d 0a 43 6f |zip, deflate..Co|
|00000120| 6e 6e 65 63 74 69 6f 6e 3a 20 6b 65 65 70 2d 61 |nnection: keep-a|
|00000130| 6c 69 76 65 0d 0a 52 65 66 65 72 65 72 3a 20 68 |live..Referer: h|
|00000140| 74 74 70 3a 2f 2f 6c 6f 63 61 6c 68 6f 73 74 3a |ttp://localhost:|
|00000150| 38 30 38 30 2f 0d 0a 43 6f 6f 6b 69 65 3a 20 48 |8080/..Cookie: H|
|00000160| 6d 5f 6c 76 74 5f 62 33 39 33 64 31 35 33 61 65 |m_lvt_b393d153ae|
|00000170| 62 32 36 62 34 36 65 39 34 33 31 66 61 62 61 66 |b26b46e9431fabaf|
|00000180| 30 66 36 31 39 30 3d 31 36 32 31 39 30 35 38 35 |0f6190=162190585|
|00000190| 30 3b 20 49 64 65 61 2d 33 35 32 63 36 33 39 66 |0; Idea-352c639f|
|000001a0| 3d 31 32 37 64 31 61 65 35 2d 34 34 31 37 2d 34 |=127d1ae5-4417-4|
|000001b0| 61 62 36 2d 61 61 64 33 2d 36 32 36 62 66 38 36 |ab6-aad3-626bf86|
|000001c0| 34 62 62 62 33 3b 20 55 4d 5f 64 69 73 74 69 6e |4bbb3; UM_distin|
|000001d0| 63 74 69 64 3d 31 37 61 61 65 35 65 65 63 33 34 |ctid=17aae5eec34|
|000001e0| 35 31 30 2d 30 39 39 37 35 66 34 36 64 62 65 63 |510-09975f46dbec|
|000001f0| 33 38 38 2d 34 63 33 65 32 35 37 61 2d 31 34 34 |388-4c3e257a-144|
|00000200| 30 30 30 2d 31 37 61 61 65 35 65 65 63 33 36 33 |000-17aae5eec363|
|00000210| 61 62 3b 20 43 4e 5a 5a 44 41 54 41 31 32 35 38 |ab; CNZZDATA1258|
|00000220| 35 36 36 39 36 33 3d 31 36 31 32 35 36 38 34 34 |566963=161256844|
|00000230| 38 2d 31 36 32 36 34 32 32 37 30 36 2d 25 37 43 |8-1626422706-%7C|
|00000240| 31 36 32 36 34 32 38 31 35 30 0d 0a 53 65 63 2d |1626428150..Sec-|
|00000250| 46 65 74 63 68 2d 44 65 73 74 3a 20 69 6d 61 67 |Fetch-Dest: imag|
|00000260| 65 0d 0a 53 65 63 2d 46 65 74 63 68 2d 4d 6f 64 |e..Sec-Fetch-Mod|
|00000270| 65 3a 20 6e 6f 2d 63 6f 72 73 0d 0a 53 65 63 2d |e: no-cors..Sec-|
|00000280| 46 65 74 63 68 2d 53 69 74 65 3a 20 73 61 6d 65 |Fetch-Site: same|
|00000290| 2d 6f 72 69 67 69 6e 0d 0a 43 61 63 68 65 2d 43 |-origin..Cache-C|
|000002a0| 6f 6e 74 72 6f 6c 3a 20 6d 61 78 2d 61 67 65 3d |ontrol: max-age=|
|000002b0| 30 0d 0a 0d 0a                                  |0....           |
+--------+-------------------------------------------------+----------------+
16:03:05.810 [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486] WRITE: 49B
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 48 54 54 50 2f 31 2e 31 20 32 30 30 20 4f 4b 0d |HTTP/1.1 200 OK.|
|00000010| 0a 63 6f 6e 74 65 6e 74 2d 6c 65 6e 67 74 68 3a |.content-length:|
|00000020| 20 31 30 0d 0a 0d 0a 68 65 6c 6c 6f 20 32 30 32 | 10....hello 202|
|00000030| 32                                              |2               |
+--------+-------------------------------------------------+----------------+
16:03:05.810 [nioEventLoopGroup-3-3] DEBUG io.netty.handler.logging.LoggingHandler - [id: 0x924b66ea, L:/127.0.0.1:8080 - R:/127.0.0.1:52486] FLUSH
16:03:05.810 [nioEventLoopGroup-3-3] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message DefaultHttpRequest(decodeResult: success, version: HTTP/1.1)
GET /favicon.ico HTTP/1.1
Host: localhost:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:95.0) Gecko/20100101 Firefox/95.0
Accept: image/avif,image/webp,*/*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://localhost:8080/
Cookie: Hm_lvt_b393d153aeb26b46e9431fabaf0f6190=1621905850; Idea-352c639f=127d1ae5-4417-4ab6-aad3-626bf864bbb3; UM_distinctid=17aae5eec34510-09975f46dbec388-4c3e257a-144000-17aae5eec363ab; CNZZDATA1258566963=1612568448-1626422706-%7C1626428150
Sec-Fetch-Dest: image
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: same-origin
```


#### 群聊


**服务端：**

```java
private static final Logger log= LoggerFactory.getLogger(NettyServer.class);

   //维护所有channel，key=名称，value为channel对象
    private static Map<String, NioSocketChannel> sessions=new ConcurrentHashMap<>();

    public static Map<String, NioSocketChannel> getSessions() {
        return sessions;
    }
    public static void putSession(String name,NioSocketChannel channel){

        sessions.put(name,channel);

    }

    public static void removeSession(String name){

        sessions.remove(name);
    }

  public static void main(String[] args) {

      NioEventLoopGroup boss = new NioEventLoopGroup(1);
      NioEventLoopGroup worker = new NioEventLoopGroup(6);
      new ServerBootstrap()
              .group(boss,worker)
              .channel(NioServerSocketChannel.class)
              .childHandler(new ChannelInitializer<NioSocketChannel>() {
                  @Override
                  protected void initChannel(NioSocketChannel ch) throws Exception {

//                      ch.pipeline().addLast(new LineBasedFrameDecoder(1024));//配置行解码器

                      ch.pipeline().addLast(new LoggingHandler());
                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {

                              super.exceptionCaught(ctx, cause);
                          }
                          //读消息
                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

                              ByteBuf byteBuf=(ByteBuf)msg;
                              byte b[]=new byte[byteBuf.readableBytes()];
                              byteBuf.readBytes(b);
                              String str=new String(b,"utf8");
                              JSONObject jsonObject = JSONObject.parseObject(str);
                              int state = (int) jsonObject.get("state");
                              String username = (String) jsonObject.get("username");
                              switch (state)
                                  {
                                      case 0: //上线
                                          NettyServer.putSession(username, ch);

                                          if(username.equals("client4")){ //如果是client4用户登录则群发

                                              sessions.forEach((k,v)->{

                                                  ByteBuf buffer = ctx.alloc().buffer(16);
                                                  try {
                                                      buffer.writeBytes("群发hhhh".getBytes("utf8"));
                                                      v.writeAndFlush(buffer);
//                                                      buffer.clear();
                                                  } catch (UnsupportedEncodingException e) {
                                                      e.printStackTrace();
                                                  }
                                              });

                                          }
                                          System.out.println("当前在线人数："+NettyServer.getSessions().size());
                                          break;
                                      case 1: //下线
                                          NettyServer.removeSession(username);
                                          NettyServer.getSessions().forEach((k,v)->{
                                              System.out.println(k);
                                              System.out.println(v.hashCode());
                                          });
                                          System.out.println("当前在线人数："+NettyServer.getSessions().size());
                                          break;
                                      default:
                                          break;
                                  }


                              super.channelRead(ctx, msg);
                          }
                      });

                  }
              }).bind(8080);



  }
```

**客户端1：**

```java
public static void main(String[] args) {
        String name="client1";
      NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
      try{
      new Bootstrap()
              .group(nioEventLoopGroup)
              .channel(NioSocketChannel.class)
              .handler(new ChannelInitializer<Channel>() {
                  @Override
                  protected void initChannel(Channel ch) throws Exception {

                      ch.pipeline().addLast(new LoggingHandler());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {
                              User user = new User();
                              user.setUsername(name);
                              user.setState(0);
                              String jsonString = JSON.toJSONString(user);
                              ByteBuf buffer = ctx.alloc().buffer(16);
                              buffer.writeBytes(jsonString.getBytes("utf8"));
                              ch.writeAndFlush(buffer);
                              super.channelActive(ctx);
                          }

                          @Override
                          public void channelInactive(ChannelHandlerContext ctx) throws Exception {
                              User user = new User();
                              user.setUsername(name);
                              user.setState(1);
                              String jsonString = JSON.toJSONString(user);
                              ByteBuf buffer = ctx.alloc().buffer(16);
                              buffer.writeBytes(jsonString.getBytes("utf8"));
                              ch.writeAndFlush(buffer);
                              super.channelInactive(ctx);
                          }

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              System.out.println(msg);
                              super.channelRead(ctx, msg);
                          }
                      });
                  }
              }).connect("localhost",8080);
      }catch (Exception e){
          e.printStackTrace();
      }

  }
```

**客户端2：**

```java
 public static void main(String[] args) {
        String name="client2";
      NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
      try{
      new Bootstrap()
              .group(nioEventLoopGroup)
              .channel(NioSocketChannel.class)
              .handler(new ChannelInitializer<Channel>() {
                  @Override
                  protected void initChannel(Channel ch) throws Exception {

                      ch.pipeline().addLast(new LoggingHandler());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {
                              User user = new User();
                              user.setUsername(name);
                              user.setState(0);
                              String jsonString = JSON.toJSONString(user);
                              ByteBuf buffer = ctx.alloc().buffer(16);
                              buffer.writeBytes(jsonString.getBytes("utf8"));
                              ch.writeAndFlush(buffer);
                              super.channelActive(ctx);
                          }

                          @Override
                          public void channelInactive(ChannelHandlerContext ctx) throws Exception {
                              User user = new User();
                              user.setUsername(name);
                              user.setState(1);
                              String jsonString = JSON.toJSONString(user);
                              ByteBuf buffer = ctx.alloc().buffer(16);
                              buffer.writeBytes(jsonString.getBytes("utf8"));
                              ch.writeAndFlush(buffer);
                              super.channelInactive(ctx);
                          }

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              System.out.println(msg);
                              super.channelRead(ctx, msg);
                          }
                      });
                  }
              }).connect("localhost",8080);
      }catch (Exception e){
          e.printStackTrace();
      }

  }
```

**客户端3：**

```java
 public static void main(String[] args) {
        String name="client3";
      NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
      try{
      new Bootstrap()
              .group(nioEventLoopGroup)
              .channel(NioSocketChannel.class)
              .handler(new ChannelInitializer<Channel>() {
                  @Override
                  protected void initChannel(Channel ch) throws Exception {

                      ch.pipeline().addLast(new LoggingHandler());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {

                              User user = new User();
                              user.setUsername(name);
                              user.setState(0);
                              String jsonString = JSON.toJSONString(user);
                              ByteBuf buffer = ctx.alloc().buffer(16);
                              buffer.writeBytes(jsonString.getBytes("utf8"));
                              ch.writeAndFlush(buffer);
                              super.channelActive(ctx);
                          }

                          @Override
                          public void channelInactive(ChannelHandlerContext ctx) throws Exception {
                              User user = new User();
                              user.setUsername(name);
                              user.setState(1);
                              String jsonString = JSON.toJSONString(user);
                              ByteBuf buffer = ctx.alloc().buffer(16);
                              buffer.writeBytes(jsonString.getBytes("utf8"));
                              ch.writeAndFlush(buffer);
                              super.channelInactive(ctx);
                          }

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              System.out.println(msg);
                              super.channelRead(ctx, msg);
                          }
                      });
                  }
              }).connect("localhost",8080);
      }catch (Exception e){
          e.printStackTrace();
      }

  }
```

**客户端4：**

```java
public static void main(String[] args) {
        String name="client4";
      NioEventLoopGroup nioEventLoopGroup = new NioEventLoopGroup();
      try{
      new Bootstrap()
              .group(nioEventLoopGroup)
              .channel(NioSocketChannel.class)
              .handler(new ChannelInitializer<Channel>() {
                  @Override
                  protected void initChannel(Channel ch) throws Exception {

                      ch.pipeline().addLast(new LoggingHandler());

                      ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){

                          @Override
                          public void channelActive(ChannelHandlerContext ctx) throws Exception {


                              User user = new User();
                              user.setUsername(name);
                              user.setState(0);
                              String jsonString = JSON.toJSONString(user);
                              ByteBuf buffer = ctx.alloc().buffer(16);
                              buffer.writeBytes(jsonString.getBytes("utf8"));
                              ch.writeAndFlush(buffer);
                              super.channelActive(ctx);
                          }

                          @Override
                          public void channelInactive(ChannelHandlerContext ctx) throws Exception {
                              User user = new User();
                              user.setUsername(name);
                              user.setState(1);
                              String jsonString = JSON.toJSONString(user);
                              ByteBuf buffer = ctx.alloc().buffer(16);
                              buffer.writeBytes(jsonString.getBytes("utf8"));
                              ch.writeAndFlush(buffer);
                              super.channelInactive(ctx);
                          }

                          @Override
                          public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                              System.out.println(msg);
                              super.channelRead(ctx, msg);
                          }
                      });
                  }
              }).connect("localhost",8080);
      }catch (Exception e){
          e.printStackTrace();
      }

  }
```

**实体类：**

```java
public class User implements Serializable {

    private String username;
    private int state; //用户状态，0在线，1下线

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public int getState() {
        return state;
    }

    public void setState(int state) {
        this.state = state;
    }

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", state=" + state +
                '}';
    }
}
```

## JVM


### JVM的定义

**Java Virtual Machine（Java虚拟机），JAVA程序的运行环境（JAVA二进制字节码的运行环境）**

### JVM带来的好处

* 一次编写，到处运行
* 垃圾回收机制
* 数组下标越界检查（C语言是没有的）

### JVM、JRE、JDK的区别

包含关系：**JDK>JRE>JVM**

![jvm-01.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/jvm-01.png)


### Java内存结构(JVM内存结构)

**记得区别Java内存结构和Java内存模型，Java内存模型是虚构的，而Java内存结构是真实存在的**
![jvm-02.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/jvm-02.png)


#### 程序计数器 

> 作用

**记录下一条JVM执行的指令的地址**

> 特点

* 线程私有
  * CPU会为每个线程分配时间片，假如当前线程时间片用完之后就会执行另外一个线程的代码
  * 每个线程都有一个程序计数器，CPU会不断重复上面的顺序去执行代码，**由程序计数器去记录每个线程应该执行哪一句代码**。
* 不存在内存溢出

#### 虚拟机栈

**定义**

* 线程私有
* 每个线程运行需要的内存空间称为虚拟机栈
* 每个栈由多个栈帧组成，栈帧是由调用方法产生的(入栈)，调用完方法后自动销毁(出栈)
* 每个线程创建的虚拟机栈都**只有**一个活动栈帧，对应着当前正在执行的方法


##### 垃圾回收是否涉及栈内存

**不涉及**。因为虚拟机栈是由一个个栈帧组成，当调用方法时**栈帧入栈**，调用完该方法后该栈帧出栈，即内存释放了。所以不需要
垃圾回收器去回收栈内存。

##### 栈内存越大是否越好？

**不是**。因为物理内存是固定的一个数值，栈内存增大，好处是可以接受更多次递归调用或者方法调用，但是可执行的线程就会变少。因为我们上面说了
当线程执行代码时就会创建一个虚拟机栈，**假如我们物理内存有1000MB和每一个栈内存10MB，这种情况计算得可以支持的线程数是100个**，
反之如果我们增大栈内存，**物理内存还是1000MB而每一个栈内存变成100MB，这种情况下我们能够支持的线程数变成了10个。大幅减少了可支持线程线程数**。

##### 方法内的局部变量是否安全

* 如果方法外不能使用该局部变量，那么就是**线程安全**的，反之则是不安全

##### 栈溢出

**异常信息：Java.lang.stackOverflowError**

**原因：**

* 递归没有终止条件，或者是永远无法达到递归终止条件，则会导致方法调用产生**栈帧过多**，最终发生栈溢出异常。（**较为常见**） 
* 某个方法太过于庞大，导致**栈帧多大**，最终发生栈溢出。（**不常见**）

##### 排查CPU占用过高--重要

**第一步在控制台输入：top,然后找到了占用cpu过高的进程**

![jvm-03.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/jvm-03.png)

**第二步输入：top -Hp cpu占用多高的进程id(也就是上面top命令查看到的进程id) ，然后就可以找到cpu占用高的线程id**

![jvm-04.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/jvm-04.png)

**第三步输入：jstack 线程id**

**上面我们找到的线程id是3294。转成16进制，得0xCDE，根据这个16进制数找到对应的位置就可以看到信息了。**



#### 本地方法栈

一些带有**native关键字**的方法就会调用本地的C/C++函数，因为Java不能和系统底层交互，所以需要这些语言的借助。

#### 堆

通过**new**关键字创建的对象就会被放到堆内存。

**特点：**

* 线程共享。堆内存的对象需要考虑线程安全问题
* 有垃圾回收机制


**堆内存溢出**

**java.lang.OutofMemoryError**，简称OOM


##### 堆内存诊断工具

**jps**

```text
D:\java code\netty-study>jps
10276 Launcher
18804 demo  #目标进程id
14236 Jps
```
**1：jmap**

> JDK8之前

```text
jmap -heap 进程id
```

> JDK8之后

```text
jhsdb jmap --heap --pid 进程id
```

**输出**

```text
Attaching to process ID 18804, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 10.0.2+13

using thread-local object allocation.
Garbage-First (G1) GC with 8 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 6402605056 (6106.0MB)
   NewSize                  = 1363144 (1.2999954223632812MB)
   MaxNewSize               = 3840933888 (3663.0MB)
   OldSize                  = 5452592 (5.1999969482421875MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 1048576 (1.0MB)

Heap Usage:
G1 Heap:
   regions  = 6106
   capacity = 6402605056 (6106.0MB)
   used     = 4194304 (4.0MB)
   free     = 6398410752 (6102.0MB)
   0.06550933508024893% used
G1 Young Generation:
Eden Space:
   regions  = 4
   capacity = 27262976 (26.0MB)
   used     = 4194304 (4.0MB)
   free     = 23068672 (22.0MB)
   15.384615384615385% used
Survivor Space:
   regions  = 0
   capacity = 0 (0.0MB)
   used     = 0 (0.0MB)
   free     = 0 (0.0MB)
   0.0% used
G1 Old Generation:
   regions  = 0
   capacity = 373293056 (356.0MB)
   used     = 0 (0.0MB)
   free     = 373293056 (356.0MB)
   0.0% used

5360 interned Strings occupying 400600 bytes.

```

**2：jconsole**

**3：jvirsualvm**

**4：阿里巴巴arthas**


#### 方法区

**JDK1.8之前和JDK1.8之后的方法区结构：**

![jvm-05.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/jvm-05.png)

**方法区的实现：**

* JDK1.8之前，也就是jdk1.6、jdk1.7这些版本方法区采用的是**永久代**
* JDK1.8之后，也就是jdk1.8、jdk1.9这些版本方法区采用的是**元空间**

**常量池**

二进制字节码的组成：类的基本信息、常量池、类的方法定义、jvm指令

##### 通过反编译来查看类的信息

**1：先用javac编译成class**

```shell script
javac Demo1.java
```

**2：再用javap进行反编译字节码文件**

```shell script
javap -v Demo1.class
```

**3：输出反编译后的字节码信息**

```text
Classfile /D:/java code/jvm/src/com/jvm/demo1/Demo1.class
  Last modified 2022年2月21日; size 429 bytes
  MD5 checksum dcc0c07c66f64b64b606d0b6566c3c9c
  Compiled from "Demo1.java"
public class com.jvm.demo1.Demo1
  minor version: 0
  major version: 54
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #5                          // com/jvm/demo1/Demo1
  super_class: #6                         // java/lang/Object
  interfaces: 0, fields: 0, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
   #2 = String             #16            // hello
   #3 = Fieldref           #17.#18        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Methodref          #19.#20        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #21            // com/jvm/demo1/Demo1
   #6 = Class              #22            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               Demo1.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = Utf8               hello
  #17 = Class              #23            // java/lang/System
  #18 = NameAndType        #24:#25        // out:Ljava/io/PrintStream;
  #19 = Class              #26            // java/io/PrintStream
  #20 = NameAndType        #27:#28        // println:(Ljava/lang/String;)V
  #21 = Utf8               com/jvm/demo1/Demo1
  #22 = Utf8               java/lang/Object
  #23 = Utf8               java/lang/System
  #24 = Utf8               out
  #25 = Utf8               Ljava/io/PrintStream;
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               println
  #28 = Utf8               (Ljava/lang/String;)V
{
  public com.jvm.demo1.Demo1();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: ldc           #2                  // String hello
         2: astore_1
         3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         6: aload_1
         7: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        10: return
      LineNumberTable:
        line 7: 0
        line 9: 3
        line 11: 10
}
SourceFile: "Demo1.java"

```

**真正编译的方法内容：**

```text
Code:
      stack=2, locals=2, args_size=1
         0: ldc           #2                  // String hello
         2: astore_1
         3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         6: aload_1
         7: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        10: return
```

**这些#数字代表地址，需要去常量池里面找。。。。**


**常量池内容：**

```text
Constant pool:
   #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
   #2 = String             #16            // hello
   #3 = Fieldref           #17.#18        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Methodref          #19.#20        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #21            // com/jvm/demo1/Demo1
   #6 = Class              #22            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               Demo1.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = Utf8               hello
  #17 = Class              #23            // java/lang/System
  #18 = NameAndType        #24:#25        // out:Ljava/io/PrintStream;
  #19 = Class              #26            // java/io/PrintStream
  #20 = NameAndType        #27:#28        // println:(Ljava/lang/String;)V
  #21 = Utf8               com/jvm/demo1/Demo1
  #22 = Utf8               java/lang/Object
  #23 = Utf8               java/lang/System
  #24 = Utf8               out
  #25 = Utf8               Ljava/io/PrintStream;
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               println
  #28 = Utf8               (Ljava/lang/String;)V
```

**运行时常量池**

* **常量池**的**值是一个地址**，不是真正的值
```text
#2 = String             #16 
```
* **运行时常量池**当该**类被加载**时，它的常量池信息（也就是上面展示的）就**会变成真正的值**。


#### 常量池和字符串常量池StringTable区别

* StringTable底层是**HashTable**，是**线程安全**的
* 常量池的值是一个地址，只有运行时才会变成真正的值
* 可以利用字符串常量池StringTable的特性来**避免重复创建String对象**
* 字符串**变量**的**加法拼接**底层是new了一个StringBuilder再new一个String对象进行拼接
* 字符串**常量**的**加法拼接**原理是**编译器优化**。
* 可以使用**intern方法**主动将StringTable还没有的字符串放入StringTable。
* StringTable和堆的字符串都是对象






## Vue2框架-2.9.6

**该Vue2入门笔记所参考的是：Vue.js官方文档、尤雨溪讲解视频(中文翻译版)、以及诸多CSDN笔记、知乎等，仅供入门参考，不作任何深入研究！**

### 起步


#### 编译器选择

* IDEA（不推荐），IDEA开发Vue应用的代码提示不太友好
* Hbuilder （不推荐），Hbuilder速度慢，HbuilderX是用C++进行重写，速度更快
* HbuilderX （推荐），Vue的合作伙伴，代码提示很好，官方也推荐
* vs code (推荐)，但没用过


#### CDN安装Vue

```html
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
```

**或者**

```html
<!-- 生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
```

#### 第一个Vue程序


```html
<div id="vue-app">
    {{content}}
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>

<script>

    var vue = new Vue({

        el:'#vue-app', //绑定元素
        data:{   //传入到绑定元素的数据
            content:'hello world！！！'
        }

    });

</script>
```

#### v-bind

**v-bind的作用是绑定属性，例如v-bind:src，就是绑定了src属性，我们可以在vue对象里面更改即可**

绑定属性语法有**两种**

* v-bind:src
* :src

**上面这两种方式等价**


```html

<html lang="en" xmlns:v="http://www.w3.org/1999/xhtml" xmlns:v-bind="http://www.w3.org/1999/xhtml">

<div id="vue-app">
    <span v-bind:title="currentTime">vue</span>
</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>

<script>

    var vue=new Vue({

        el:'#vue-app',
        data:{
            currentTime:'页面加载于 ' + new Date().toLocaleString()
        }

    });

</script>
```

然后在浏览器控制台输入vue.currentTime='v-bind触发了'，就会发现内容被更改了


#### v-text

**v-text和{{}}非常相似，其实他们两个作用也是差不多的。**

```html
<div id="vue-app1">
    <!-- v-text取值-->
    <h2 v-text="content"></h2>
    <!-- {{}}取值-->
    <h3>文本：{{content}}</h3>

</div>

<div id="vue-app2">
    <!-- v-text取值-->
    <h2 v-text="arr"></h2>
    <!-- {{}}取值-->
    <h3>文本：{{arr}}</h3>
	
	<!-- 取数组指定下标元素 -->
	<h3>{{arr[2]}}</h3>
	
	 <h2 v-text="arr[1]"></h2>

</div>

<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>

<script>

    var vue1=new Vue({

        el:'#vue-app1',
        data:{
            content:'我是内容'
        }

    });
	
	var vue2=new Vue({
		
		el:'#vue-app2',
		data:{
			arr:['1','2','3','4','5','6'] //传入数组
			
		}
		
		
	});
	
</script>
```



#### v-html

**v-text和v-html虽然看起来差不多，但是还是有很大的不同。**

* v-text传入的数据**不会转义成html语句**
* v-html传入的数据**会转义成html语句展示**

**例子：**

```html
        <div id="vue-app">
			<span>---------v-text</span>
			<br />
			<span v-text="content">
				
			</span>
			
			<br />
			<span>---------v-html</span>
			<br />
			
			<span v-html="content">
				
			</span>
			
		</div>
		
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		<script type="text/javascript">
		
			var vue=new Vue({
				
				el:'#vue-app',
				data:{
					content:'<a href=\"https://gitee.com/youzhengjie\">个人主页</a>'
				}
				
			});
		
		</script>
```

#### 计数器案例：v-on/@

**定义vue的事件方法必须在vue对象里面的methods定义，而不能在vue对象外定义**

**methods的事件语法规则：**

* 方法名:function(也可以写方法参数){js语法}


**绑定单击事件方法,两种方法等价**

* v-on:click="sub()"
* @click="add()"

> 案例

```html
        <div id="vue-app">
			<!-- 绑定事件 -->
			<input type="button" value="-" v-on:click="sub()">
			<span>{{num}}</span>
			<input type="button" value="+" @click="add()">
			
		</div>
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		<script>
		
		   var vue =new Vue({
				
				el:'#vue-app',
				data:{
					num:1
				},
				//定义vue的事件方法必须在vue对象里面的methods定义，而不能在vue对象外定义。。。。
				methods:{
					sub:function(){
						this.num--; 
						console.log(this); //这里的this等价于进入到data里面了
					},
					add:function(){
						this.num++;
						console.log(this.num);
					}
				}
			   
		   });
		
		</script>
```


#### v-show

**v-show="布尔值"，如果布尔值为true说明展示，反之为false则隐藏，底层是通过修改css样式进行隐藏的**


```html
         <div id="vue-app">
			
			<h3 v-show="v1">v1</h3>
			<h3 v-show="v2">v2</h3>
			<h3 v-show="3>=5">v3</h3>
			<h3 v-show="5>3">v4</h3>
		</div>
		
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		
		<script>
		
			var vue=new Vue({
				
				el:'#vue-app',
				data:{
					v1:true,
					v2:false
				}
				
			});
	
		</script>
```

#### v-if

```html
        <div id="vue-app">
			
			<h3 v-if="res==1">
				111
			</h3>
			<h3 v-else-if="res==2">
				222
			</h3>
			
			<h3 v-else-if="res==3">
				333
			</h3>
			
			<h3 v-else>
				666
			</h3>
			
		</div>
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		<script>
		
			var vue=new Vue({
				
				el:'#vue-app',
				data:{					
					res:3
				}
				
			});
		
		</script>
```

#### v-for

```html
        <div id="vue-app">
			
			<div v-for="(item,index) in arr">
				
				{{item}}->{{index}}
				
			</div>
			
			
		</div>
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		
		<script>
		
			var vue=new Vue({
				
				el:'#vue-app',
				data:{
					
					arr:['aaa','bbb','ccc','ddd','eee']
				}
				
			});
		
		</script>
```



#### 切换图片案例

```html
        <div id="vue-app">
			
			<!-- 因为切换图片是切换src，所以要用v-bind:src或者:src去绑定src属性 -->
			
			<input type="button" value="<" @click="left()" v-show="index!=0">
			<img v-bind:src="imgs[index]" alt="" style="width: 200px;height: 200px;">
			<input type="button" value=">" v-on:click="right()" v-show="index!=imgs.length-1">
		</div>
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		<script>
		
		   var vue = new Vue({
			   
			   el:'#vue-app',
			   data:{
				   // 定义一个存放图片的数组
				   imgs:['http://5b0988e595225.cdn.sohucs.com/images/20170828/5ebb6bb4797e4d4faa848c530acfd016.jpeg',
				         'https://pic.qqtn.com/up/2019-1/2019010208201525732.jpg',
						 'https://ss0.bdstatic.com/70cFuHSh_Q1YnxGkpoWK1HF6hhy/it/u=1901564855,3168536127&fm=26&gp=0.jpg',
						 'http://n.sinaimg.cn/sinacn/w640h595/20180218/a939-fyrswmu0801569.jpg',
						 'http://pic.wodingche.com/carimg/febgjfifz.jpeg'
						],
				   index:0
			   },
			   methods:{
				   
				   left:function(){
					   console.log('left');
					   this.index--;
				   },
				   right:function(){
					   console.log('right')
					   this.index++;
				   }
			   }
			   
		   });
		
		</script>
```


#### v-model

**v-model可以实现双向绑定。**

```html
        <div id="vue-app">
			
			<!-- t1绑定了text字段，如果修改t1的内容，则vue对象的text也会被修改，反之修改vue对象的text字段t1也会被修改，这就是双向绑定 -->
			<input id="t1" type="text" v-model="text">
			
			<!-- 展示vue对象的text值，看看有没有实现双向绑定 -->
			<h3>{{text}}</h3>
			
		</div>
		
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		<script>
		
		   var vue=new Vue({
			   
			  el:'#vue-app',
			  data:{
				  
				text:'默认文本'
				  
			  }
			   
		   });
		
		
		</script>
```


### Vue生命周期函数


```html
      <div id="vue-app">
			
			<span id="t1">{{tx}}</span>
			
			
		</div>
		
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		<script>
		
		// VUE生命周期钩子函数
			var vue=new Vue({
				
				el:'#vue-app',
				data:{
					tx:'数据加载了'
				},
				//创建vue对象前调用
				beforeCreate:()=>{
					
					console.log('beforeCreate')
					
				},
				//创建vue对象后立刻调用，此时data还没有赋值到html
				created:()=>{
					
					console.log(document.getElementById('t1').innerText); //返回：{{tx}}
				},
				//data赋值到html页面后触发。
				mounted:()=>{
					
					console.log(document.getElementById('t1').innerText);//返回：数据加载了
				},
				//销毁vue对象后触发
				destroyed: () => {
					
					console.log('vue对象被销毁了')
				}
				
			});
			
			vue.$destroy(); //销毁vue对象，触发destroyed生命周期钩子函数
		
		
		</script>
```


### 定义data的3种方式

**这三种方法效果其实是差不多一样的，就是语法不同。**



```html
        <div id="vue-app1">
			
			<span>{{content}}</span>
			
		</div>
		
		<div id="vue-app2">
			
			<span>{{content}}</span>
			
		</div>
		
		<div id="vue-app3">
			
			<span>{{content}}</span>
			
		</div>
		
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		
		<script>
		
		// 方式一：推荐
			var vue1=new Vue({
				
				el:'#vue-app1',
				data:{		
					content:'vue-app1'	
				}
				
			});
		
		//方式二：不喜欢
			var vue2=new Vue({
				
				el:'#vue-app2',
				data:function(){
					return{
						content:'vue-app2'
					}
				}
				
			});
			
		
		//方式三：推荐
			var vue3=new Vue({
				
				el:'#vue-app3',
				data(){
					
					return{
						content:'vue-app3'
					}
				}
				
			});
		
		
		
		</script>
```


### axios异步通信

**axious实际上就是ajax的替代品。**

#### 安装axios

> CDN安装

```html
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

或者

```html
<script src="https://cdn.staticfile.org/axios/0.21.0/axios.min.js"></script>
```

> npm安装

```shell script
$ npm install axios
```

> github下载

**https://github.com/axios/axios**


#### 使用axios

##### get

```html
<div id="vue-app">
			{{json}}
			<input type="button" @click="getJson()" v-bind:value="btnName">
		</div>
		
		 
		
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		<!-- axios异步通信 -->
		<script src="https://cdn.staticfile.org/axios/0.21.0/axios.min.js"></script>
		
		
		<script>
		
			var vue =new Vue({
				
				el:'#vue-app',
				// 等价于data:{}
				data(){
					return {
						
						json:'null',
						btnName:'getJson'
						
					}
				},
				methods:{
					
					// 1：方式一：this可以获取，推荐用
					getJson:function(){
						// axios异步调用
						
						//********保存当前this对象，因为axios内部访问的this不是vue对象的this*********
						var nthis=this; 
						
						
						axios
						.get('./axios-demo.json?username=1')
						.then(function(response){
							nthis.json=response.data.sites[1].name;//获取axios请求过来的json
						})
						.catch(function(err){
							console.log(err)
						});
						
					}
					
					//方式二：this获取不到,说明getJson:function(){}和getJson:()=>{}是有区别的
					
					// getJson:()=>{
					// 	// axios异步调用
						
					// 	//********保存当前this对象，因为axios内部访问的this不是vue对象的this*********
					// 	var nthis=this; 
						
						
					// 	axios
					// 	.get('./axios-demo.json?username=1')
					// 	.then(function(response){
					// 		nthis.json=response.data.sites[1].name;//获取axios请求过来的json
					// 	})
					// 	.catch(function(err){
					// 		console.log(err)
					// 	});
						
					// }
					
				},
				mounted() {
					
					//********保存当前this对象，因为axios内部访问的this不是vue对象的this*********
					var nthis=this;
					
					
					axios
					.get('./axios-demo.json?username=1')
					.then(function(response){
						console.log(nthis)
					})
					.catch(function(err){
						console.log(err)
					});
					
				}
				
			});

		</script>
```

**axios-demo.json文件:**

```json
{
    "name":"网站",
    "num":3,
    "sites": [
        { "name":"Google", "info":[ "Android", "Google 搜索", "Google 翻译" ] },
        { "name":"Runoob", "info":[ "菜鸟教程", "菜鸟工具", "菜鸟微信" ] },
        { "name":"Taobao", "info":[ "淘宝", "网购" ] }
    ]
}

```


##### ()=>和function()区别


**()=>和function()定义方法内的this对象是不同的，很多情况还是用function()好一点**


##### post

```html
axios.post('/user',{
    username:'root',
    password:123456
})
.then(function(response){
    console.log(response)
}
)
.catch(function(err){
    console.log(err)
}
);
```



### Vue Component

**组件是可复用的Vue实例， 说白了就是一组可以重复使用的模板， 跟JSTL的自定义标签、Thymeleal的th:fragment等框架有着异曲同工之妙，通常一个应用会以一棵嵌套的组件树的形式来组织：**

#### 第一个组件

```html
        <div id="vue-app">
			<!-- 自定义标签就是组件 -->
			<my-content></my-content>
		</div>
		
		
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		
		<script>
		
			//定义组件。。
			//组件必须放在new Vue绑定的el元素内，不然无法生效
			Vue.component('my-content',{
				
				template:'<div>\
				<ul>\
				<li>Java</li>\
				<li>Redis</li>\
				<li>Docker</li>\
				</ul>\
				</div>'
			});
		
			var vue=new Vue({
				
				el:'#vue-app'
				,
				comments:['my-content'] //可选：也可以这样写
				
			});
		
		
		
		</script>
```


#### 使用props属性传递参数

**注意：默认规则下props属性里的值不能为大写**

```html
        <div id="vue-app">
			
			<!-- 使用组件 ,绑定循环对象-->
			<my-content v-for="(item,index) in tx" v-bind:proptx="item"></my-content>
			
			<!-- 使用组件，绑定多个值 -->
			<my-foot v-bind:us="username" :pwd="password"></my-foot>
		</div>
		
		
		<!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
		
		
		<script>
			
			//使用props把vue对象的参数传到Vue.component组件里面,然后配合v-bind:prop='接收的值'
			
			Vue.component('my-content',{
				
				props:['proptx'],
				template:'<h3>{{proptx}}</h3>'
				
			});
			
			Vue.component('my-foot',{
				
				props:['us','pwd'],
				template:'<h4>{{us}}->{{pwd}}</h4>'
				
			});
			
			
			var vue =new Vue({
				
				el:'#vue-app',
				
				data() {
					
					return{
						
						tx:[1,2,3],
						username:'admin',
						password:'123666'
						
					}
					
				}
				
			});
		
		
		</script>
```

### Vue-cli脚手架

> 安装nodejs

```text
cmd检查本机是否有nodejs：node -v
如果没有就下载：
下载网址：http://nodejs.cn/download/
```

> 安装npm

```text
cmd下输入npm -v，查看是否能够正确打印出版本号即可！
如果没有安装则：npm install
```
> 安装git

```text
Git：https://git-scm.com/doenloads
镜像：https://npm.taobao.org/mirrors/git-for-windows/
```

> 安装cnpm

```text
# -g 就是全局安装
npm install cnpm -g

# 或使用如下语句解决npm速度慢的问题
npm install --registry=https://registry.npm.taobao.org
```

> 安装webpack

```text
npm install webpack -g  或者 npm install -g webpack

检查webpack是否安装成功
webpack -v
```

> 安装vue-cli

```text
全局安装：
npm install --global vue-cli

检查是否安装成功
vue -V
vue list
```

#### 第一个Vue-cli

```text
1：随便创建一个文件夹：D:\vue\vue-demo1
2：进入这个文件夹：cd D:\vue\vue-demo1
3: 初始化：vue init webpack 自己起一个vue项目名，例如：vue init webpack myvue1
4：初始化的过程中一路都选择*no*即可
```

**安装vue-cli过程说明：**
* Project name：项目名称，默认**回车**即可
* Project description：项目描述，默认**回车**即可
* Author：项目作者，默认**回车**即可
* Install vue-router：是否安装vue-router路由，选择**n**不安装（**后期需要再手动添加**）
* Use ESLint to lint your code:是否使用ESLint做代码检查，选择**n**不安装（**后期需要再手动添加**)
* Set up unit tests:单元测试相关，选择**n**不安装（**后期需要再手动添加**）
* Setupe2etests with Nightwatch：单元测试相关，选择**n**不安装（**后期需要再手动添加**）
* Should we run npm install for you after the,project has been created:创建完成后直接初始化，选择**n**，我们手动执行；运行结果！

**然后就会生成一个标准的Vue项目：**
![vue1.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/vue1.png)


**运行刚刚生成的Vue项目：**

```text
1：cd vue项目名 ，例如：cd myvue1
2：npm run dev 运行
```

![vue2.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/vue2.png)

**访问生成的地址：**

![vue3.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/vue3.png)

**这就是我们的第一个Vue-cli程序了。**


##### 执行vue-cli出现报错

![vue4.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/vue4.png)

> 解决办法

```shell script
npm audit fix  #进行修复即可
````


### webpack打包工具

**webpack是一款模块加载器兼打包工具， 它能把各种资源， 如JS、JSX、ES 6、SASS、LESS、图片等都作为模块来处理和使用。**

#### 安装webpack

```shell script
npm install webpack -g
npm install webpack-cli -g
```

```shell script
webpack -v
webpack-cli -v
```

#### 使用webpack

**创建名称为webpack.config.js的配置文件。例如：**

![vue5.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/vue5.png)

> 编写webpack打包的简单配置

```js
const path = require('path');

module.exports = {
  entry: './src/main.js', //打包入口
  output: {
    path: path.resolve(__dirname, 'dist'), //输出路径
    filename: 'bundle.js' //相当于./dist/bundle.js
  }
}
```

**webpack配置项：**

* entry：入口文件,指定WebPack用哪个文件作为项目的入口
* output：输出,指定WebPack把处理完成的文件放置到指定路径
* module：模块,用于处理各种类型的文件
* plugins：插件,如：热更新、代码重用等
* resolve：设置路径指向
* watch：监听,用于设置文件改动后直接打包

**执行命令即可打包：**

```shell script
webpack
```

> 动态打包，监听

```shell script
webpack --watch
```

### 展示Vue内容

**vue文件其实就是组件，组件名为export default中的name，所以想要展示vue组件的内容，有如下两种方法：**

**方式一：**
![vue6.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/vue6.png)

**方式二：**
![vue7.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/vue7.png)

**不然vue的内容将不会展示，除非用vue-router**

**main.js**

```js
//程序主入口
import Vue from "vue"; //导入vue
import Home from "./components/Home";//导入Home组件

//使用vue

new Vue({

  //******这里main.js的el名称必须是#app
  el:'#app',
  //注册组件,必须要
  components:{
    Home
  },
  template: '<Home></Home>'
});
```

**Home.vue**

```vue
<template>
    <div id="app">
<!--      接收传值 -->
      <h3>{{tx}}</h3>
    </div>
</template>

<script>
    export default {
        name: "Home", //vue组件名。import Home from xxx
        data(){ //相当于new Vue里面的data属性，给当前vue组件传值
          return{
            tx:'hello vue'
          }
        }
    }
</script>

<style scoped>

</style>

```

### vue的注意点

* **程序主入口main.js的el名称必须是#app，否则会报错**


### Vue组件嵌套Vue组件

#### 实战

**main.js**

```js
//程序主入口
import Vue from "vue"; //导入vue
import Home from "./components/Home";//导入Home组件
//使用vue

new Vue({

  //******这里main.js的el名称必须是#app
  el:'#app',
  //注册组件,必须要
  components:{
    Home
  },
  template: '<Home></Home>'
});

```

**Home.vue**

```vue
<template>
    <div id="app">
<!--      接收传值 -->
      <h3>{{tx}}</h3>
      <MyContent></MyContent>
      <MyFoot></MyFoot>
    </div>
</template>

<script>
  //需要什么组件就导入什么组件，并且注册
  //方式一：
  // import MyContent from "@/components/MyContent";
  //方式二：
  import MyContent from "./MyContent";

  import MyFoot from "./MyFoot";
    export default {
        name: "Home", //vue组件名。import Home from xxx
        data(){ //相当于new Vue里面的data属性，给当前vue组件传值
          return{
            tx:'hello vue'
          }
        },
      //注册组件
      components: {
          MyContent,MyFoot
      }
    }
</script>

<style scoped>

</style>

```

**MyContent.vue**

```vue
<template>
    <div id="ct">
      <h3>我是内容</h3>
    </div>
</template>

<script>
    export default {
        name: "MyContent"
    }
</script>

<style scoped>

</style>
```

**MyFoot.vue**

```vue
<template>
  <h3>this is foot</h3>
</template>

<script>
    export default {
        name: "MyFoot"
    }
</script>

<style scoped>

</style>
```

### vue-router

**vue-Router是Vue.js官方的路由管理器。它和Vue.js的核心深度集成， 让构建单页面应用变得易如反掌。包含的功能有：**

* 嵌套的路由/视图表
* 模块化的、基于组件的路由配置
* 路由参数、查询、通配符
* 基于Vue js过渡系统的视图过渡效果
* 细粒度的导航控制
* 带有自动激活的CSS class的链接
* HTML5 历史模式或hash模式， 在IE 9中自动降级
* 自定义的滚动行为

#### 使用vue-router3.0

##### 安装

```shell script
npm install vue-router --save-dev
```

##### vue-router报错

**因为默认下载的vue-router是vue-router4，我们要修改到vue-router3才能使用。比如3.5.3版本就可以**

![vue8.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/vue8.png)

#### 路由实战

**项目结构：**

![vue9.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/vue9.png)


**main.js:**

```js
import Vue from "vue";

import App from "./App";

import router from "./router";

new Vue({

  el:'#app',
  router,
  render: h => h(App)
})
```

**App.vue:**


```vue
<template>

  <div id="app">

<!--    路由链接-->
    <router-link to="/main">首页</router-link>
    <router-link to="/content">内容</router-link>
    <router-link to="/other">其他</router-link>

<!--    路由展示组件-->
    <router-view></router-view>

    <h3>-------------------</h3>
<!--    命名路由视图-->
    <router-view name="otherview"></router-view>
  </div>

</template>

<script>
    export default {
        name: "App"
    }
</script>

<style scoped>

</style>
```

**路由核心配置文件,index.js:**

```js
import Vue from 'vue'
//导入路由插件
import VueRouter from 'vue-router'
//导入上面定义的组件
import Content from '../components/Content'
import Main from '../components/Main'
import Other from "../components/Other";

//安装路由
Vue.use(VueRouter);

//配置路由
export default new VueRouter({
  routes: [
    {
      //路由路径
      path: '/content',
      //路由名称
      name: 'content',
      //跳转到组件
      component: Content
    }, {
      //路由路径
      path: '/main',
      //路由名称
      name: 'main',
      //跳转到组件
      component: Main
    }, {

      path: '/other',
      components: {
        // default: Content, //默认如果当访问这个路径没有指定视图的name则会到这个组件
        otherview: Other //命名视图格式==>视图的name:响应组件名
      }

    }
  ]
});

```

**Content.vue:**

```vue
<template>
    <h3>content</h3>
</template>

<script>
    export default {
        name: "Content"
    }
</script>

<style scoped>

</style>

```


**Main.vue:**

```vue
<template>
    <h3>main</h3>
</template>

<script>
    export default {
        name: "Main"
    }
</script>

<style scoped>

</style>

```


**Other.vue:**

```vue
<template>
    <h3>其他</h3>
</template>

<script>
    export default {
        name: "Other"
    }
</script>

<style scoped>

</style>

```

### vue+element-ui

#### 安装element-ui

```shell script
#安装element-ui
npm i element-ui -S
#安装依赖
npm install
# npm安装SASS加载器
npm install sass-loader node-sass --save-dev


# 如果不行就用cnpm安装SASS加载器
cnpm install sass-loader node-sass --save-dev
```

**package.json：**

```json
{
  "name": "myvue2",
  "version": "1.0.0",
  "description": "A Vue.js project",
  "author": "ms666 <1550324080@qq.com>",
  "private": true,
  "scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "build": "node build/build.js"
  },
  "dependencies": {
    "element-ui": "^2.15.6",
    "vue": "^2.5.2"
  },
  "devDependencies": {
    "autoprefixer": "^7.1.2",
    "babel-core": "^6.22.1",
    "babel-helper-vue-jsx-merge-props": "^2.0.3",
    "babel-loader": "^7.1.1",
    "babel-plugin-syntax-jsx": "^6.18.0",
    "babel-plugin-transform-runtime": "^6.22.0",
    "babel-plugin-transform-vue-jsx": "^3.5.0",
    "babel-preset-env": "^1.3.2",
    "babel-preset-stage-2": "^6.22.0",
    "chalk": "^2.0.1",
    "copy-webpack-plugin": "^4.0.1",
    "css-loader": "^0.28.0",
    "extract-text-webpack-plugin": "^3.0.0",
    "file-loader": "^1.1.4",
    "friendly-errors-webpack-plugin": "^1.6.1",
    "html-webpack-plugin": "^2.30.1",
    "node-notifier": "^5.1.2",
    "node-sass": "^7.0.1",
    "optimize-css-assets-webpack-plugin": "^3.2.0",
    "ora": "^1.2.0",
    "portfinder": "^1.0.13",
    "postcss-import": "^11.0.0",
    "postcss-loader": "^2.0.8",
    "postcss-url": "^7.2.1",
    "rimraf": "^2.6.0",
    "sass-loader": "^12.6.0",
    "semver": "^5.3.0",
    "shelljs": "^0.8.5",
    "uglifyjs-webpack-plugin": "^1.1.1",
    "url-loader": "^0.5.8",
    "vue-loader": "^13.3.0",
    "vue-router": "^3.5.3",
    "vue-style-loader": "^3.0.1",
    "vue-template-compiler": "^2.5.2",
    "webpack": "^3.6.0",
    "webpack-bundle-analyzer": "^2.9.0",
    "webpack-dev-server": "^2.9.1",
    "webpack-merge": "^4.1.0"
  },
  "engines": {
    "node": ">= 6.0.0",
    "npm": ">= 3.0.0"
  },
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not ie <= 8"
  ]
}

```

#### 使用element-ui

> 引入element-ui

**在项目的入口文件main.js引入以下即可。**

```js
import Vue from 'vue'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
Vue.use(ElementUI)
```

> 实战

**main.js**

```js
import Vue from 'vue'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

import App from "./App";

//路由配置
import router from "./router";

Vue.use(router)
//使用elementui插件
Vue.use(ElementUI)

new Vue({

  el:'#app',
  router,
  render: h => h(App)
})

```

**App.vue**

```vue
<template>

  <div id="app">

    <router-link to="/activity">新增活动</router-link>
    <router-link to="/info">数据信息</router-link>

<!--    /activity专用路由视图-->
    <router-view name="act"></router-view>

    <!--    /info专用路由视图-->
    <router-view name="in"></router-view>

  </div>

</template>

<script>
  export default {
    name: "App"
  }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>

```

**路由index.js**

```js
import Vue from 'vue'
//导入路由插件
import VueRouter from 'vue-router'
//导入上面定义的组件
import Activity from "../views/Activity";
import Info from "../views/Info";

//安装路由
Vue.use(VueRouter);

//配置路由
export default new VueRouter({
  routes: [
    {
      //路由路径
      path: '/activity',
      //路由名称
      name: 'activity',
      //跳转到组件
      components: {

        act:Activity

      }
    },
    {
      //路由路径
      path: '/info',
      //路由名称
      name: 'info',
      //跳转到组件
      components: {

        in:Info

      }
    }
  ]
});

```

**Activity.vue**

```vue
<template>

  <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
    <el-form-item label="活动名称" prop="name">
      <el-input v-model="ruleForm.name"></el-input>
    </el-form-item>
    <el-form-item label="活动区域" prop="region">
      <el-select v-model="ruleForm.region" placeholder="请选择活动区域">
        <el-option label="区域一" value="shanghai"></el-option>
        <el-option label="区域二" value="beijing"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="活动时间" required>
      <el-col :span="11">
        <el-form-item prop="date1">
          <el-date-picker type="date" placeholder="选择日期" v-model="ruleForm.date1" style="width: 100%;"></el-date-picker>
        </el-form-item>
      </el-col>
      <el-col class="line" :span="2">-</el-col>
      <el-col :span="11">
        <el-form-item prop="date2">
          <el-time-picker placeholder="选择时间" v-model="ruleForm.date2" style="width: 100%;"></el-time-picker>
        </el-form-item>
      </el-col>
    </el-form-item>
    <el-form-item label="即时配送" prop="delivery">
      <el-switch v-model="ruleForm.delivery"></el-switch>
    </el-form-item>
    <el-form-item label="活动性质" prop="type">
      <el-checkbox-group v-model="ruleForm.type">
        <el-checkbox label="美食/餐厅线上活动" name="type"></el-checkbox>
        <el-checkbox label="地推活动" name="type"></el-checkbox>
        <el-checkbox label="线下主题活动" name="type"></el-checkbox>
        <el-checkbox label="单纯品牌曝光" name="type"></el-checkbox>
      </el-checkbox-group>
    </el-form-item>
    <el-form-item label="特殊资源" prop="resource">
      <el-radio-group v-model="ruleForm.resource">
        <el-radio label="线上品牌商赞助"></el-radio>
        <el-radio label="线下场地免费"></el-radio>
      </el-radio-group>
    </el-form-item>
    <el-form-item label="活动形式" prop="desc">
      <el-input type="textarea" v-model="ruleForm.desc"></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="submitForm('ruleForm')">立即创建</el-button>
      <el-button @click="resetForm('ruleForm')">重置</el-button>
    </el-form-item>
  </el-form>

</template>

<script>
  export default {
    name:'Activity',
    data() {
      return {
        ruleForm: {
          name: '',
          region: '',
          date1: '',
          date2: '',
          delivery: false,
          type: [],
          resource: '',
          desc: ''
        },
        rules: {
          name: [
            { required: true, message: '请输入活动名称', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ],
          region: [
            { required: true, message: '请选择活动区域', trigger: 'change' }
          ],
          date1: [
            { type: 'date', required: true, message: '请选择日期', trigger: 'change' }
          ],
          date2: [
            { type: 'date', required: true, message: '请选择时间', trigger: 'change' }
          ],
          type: [
            { type: 'array', required: true, message: '请至少选择一个活动性质', trigger: 'change' }
          ],
          resource: [
            { required: true, message: '请选择活动资源', trigger: 'change' }
          ],
          desc: [
            { required: true, message: '请填写活动形式', trigger: 'blur' }
          ]
        }
      };
    },
    methods: {
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            alert('submit!');
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      resetForm(formName) {
        this.$refs[formName].resetFields();
      }
    }
  }
</script>

<style scoped>

</style>

```

**Info.vue**

```vue
<style>
  .el-table .warning-row {
    background: oldlace;
  }

  .el-table .success-row {
    background: #f0f9eb;
  }
</style>

<template>
  <el-table
    :data="tableData"
    style="width: 100%"
    :row-class-name="tableRowClassName">
    <el-table-column
      prop="date"
      label="日期"
      width="180">
    </el-table-column>
    <el-table-column
      prop="name"
      label="姓名"
      width="180">
    </el-table-column>
    <el-table-column
      prop="address"
      label="地址">
    </el-table-column>
  </el-table>
</template>

<script>
  export default {
    name: 'Info',
    methods: {
      tableRowClassName({row, rowIndex}) {
        if (rowIndex === 1) {
          return 'warning-row';
        } else if (rowIndex === 3) {
          return 'success-row';
        }
        return '';
      }
    },
    data() {
      return {
        tableData: [{
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄',
        }, {
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }, {
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄',
        }, {
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }]
      }
    }
  }
</script>

<style scoped>

</style>

```

### 路由嵌套


#### 嵌套路由注意的问题

**嵌套路由必须只有一个root，否则会报错**

```text
  - Component template should contain exactly one root element. If you are using v-if on multiple elements, use v-else-if to chain them instead.


 @ ./src/views/Info.vue 12:0-360
 @ ./src/router/index.js
 @ ./src/main.js
 @ multi (webpack)-dev-server/client?http://localhost:8080 webpack/hot/dev-server ./src/main.js
```

> 问题产生原因

```vue
<template>

<!--  嵌套路由必须只有一个root，下面这个div就是root-->

  
    <router-link to="/chird/chirdContent">展示嵌套路由内容</router-link>
    <el-table
      :data="tableData"
      style="width: 100%"
      :row-class-name="tableRowClassName">
      <el-table-column
        prop="date"
        label="日期"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="姓名"
        width="180">
      </el-table-column>
      <el-table-column
        prop="address"
        label="地址">
      </el-table-column>
    </el-table>

<!--  嵌套路由视图-->
  <router-view name="cc"></router-view>

</template>
```

> 解决办法

**解决办法就是用一个div包括他们全部**

```vue
<template>

<!--  嵌套路由必须只有一个root，下面这个div就是root-->
  <div>
    <router-link to="/chird/chirdContent">展示嵌套路由内容</router-link>
    <el-table
      :data="tableData"
      style="width: 100%"
      :row-class-name="tableRowClassName">
      <el-table-column
        prop="date"
        label="日期"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="姓名"
        width="180">
      </el-table-column>
      <el-table-column
        prop="address"
        label="地址">
      </el-table-column>
    </el-table>
  <!--  嵌套路由视图-->
  <router-view name="cc"></router-view>

  </div>
</template>
```


#### 实战

**路由配置index.js**

```js
import Vue from 'vue'
//导入路由插件
import VueRouter from 'vue-router'
//导入上面定义的组件
import Activity from "../views/Activity";
import Info from "../views/Info";

//路由嵌套组件
import ChirdContent from "../views/chirdrenViews/ChirdContent";

//安装路由
Vue.use(VueRouter);

//配置路由
export default new VueRouter({
  routes: [
    {
      //路由路径
      path: '/activity',
      //路由名称
      name: 'activity',
      //跳转到组件
      components: {

        act:Activity

      }
    },
    {
      //路由路径
      path: '/info',
      //路由名称
      name: 'info',
      //跳转到组件
      components: {

        in:Info

      },
      //配置路由嵌套
      children: [
        {
          path: '/chird/chirdContent',
          components: {
            cc:ChirdContent //ChirdContent组件嵌套在info这个vue组件中
          }
        }
      ]
    }
  ]
});

```

**被嵌套组件：**

```vue
<style>
  .el-table .warning-row {
    background: oldlace;
  }

  .el-table .success-row {
    background: #f0f9eb;
  }
</style>

<template>

<!--  嵌套路由必须只有一个root，下面这个div就是root-->
  <div>
    <router-link to="/chird/chirdContent">展示嵌套路由内容</router-link>
    <el-table
      :data="tableData"
      style="width: 100%"
      :row-class-name="tableRowClassName">
      <el-table-column
        prop="date"
        label="日期"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="姓名"
        width="180">
      </el-table-column>
      <el-table-column
        prop="address"
        label="地址">
      </el-table-column>
    </el-table>
  <!--  嵌套路由视图-->
  <router-view name="cc"></router-view>

  </div>


</template>

<script>
  export default {
    name: 'Info',
    methods: {
      tableRowClassName({row, rowIndex}) {
        if (rowIndex === 1) {
          return 'warning-row';
        } else if (rowIndex === 3) {
          return 'success-row';
        }
        return '';
      }
    },
    data() {
      return {
        tableData: [{
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄',
        }, {
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }, {
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄',
        }, {
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }]
      }
    }
  }
</script>

<style scoped>

</style>

```

**嵌套组件**

```vue
<template>
    <h3>路由嵌套内容</h3>
</template>

<script>
    export default {
        name: "ChirdContent"
    }
</script>

<style scoped>

</style>

```

**其他基本和上面的实战一致。**


### 参数传递

**路由index.js**

```js
{
      //路由路径
      path: '/home/:id/:username', //绑定参数id
      //路由名称
      name: 'myhome',
      //跳转到组件
      components: {
        hm:Home
      }
    }
```

**App.vue**

```vue
<!--    name是路由名称-->
<router-link :to="{name:'myhome',params:{id:20,username:'admin'}}">绑定参数方式1</router-link>
<router-view name="hm"></router-view>
```

**Home.vue**

```vue
<template>

  <div>
    <!--  获取绑定参数-->
    <h3>{{$route.params.id}}</h3>

    <h3>{{$route.params.username}}</h3>

    <h3>{{$route.path}}</h3>

    <h3>{{$route.params}}</h3>
  </div>


</template>

<script>
    export default {
        name: "Home"
    }
</script>

<style scoped>

</style>

```

### 重定向

重定向到一个地址

**路由index.js**

```js

//重定向
    {
      //路由路径
      path: '/activity',
      //路由名称
      name: 'activity',
      //跳转到组件
      components: {

        act:Activity

      }
    },
//重定向
    {
      path: '/toActivity',
      redirect: '/activity'
    }
```

**App.vue**

```vue
 <router-link to="/toActivity">重定向</router-link>
```

### 路由模式

**路由模式有两种:**

* hash：路径带 # 符号，如 http://localhost/#/login
* history：路径不带 # 符号，如 http://localhost/login

```js
//配置路由
export default new VueRouter({
  mode:'history',
  // mode:'hash', //默认模式
  routes: [
    {
      //路由路径
      path: '/activity',
      //路由名称
      name: 'activity',
      //跳转到组件
      components: {

        act:Activity

      }
    }
  ]
});
```

### 404

**NotFound.vue**

```vue
<template>
    <h3>404未找到页面</h3>
</template>

<script>
    export default {
        name: "NotFound"
    }
</script>

<style scoped>

</style>
```

**路由index.js**

```js
import NotFound from "../views/NotFound";
//配置404页面
    {
      path:'*',
      component: NotFound
    }
```

```vue
<router-view></router-view>
```
### 路由钩子

* beforeRouteEnter: 在进入路由前执行
* beforeRouteLeave: 在离开路由前执行

**Home.vue:**
```vue
<script>
    export default {
        name: "Home",
        beforeRouteEnter:((to, from, next) => {
          console.log('进入路由之前');
          next();
        }),
        beforeRouteLeave:((to, from, next) => {
          console.log('离开路由之前')
          next();
        })
    }
</script>
```

**参数说明：**
* to：路由将要跳转的路径信息
* from：路径跳转前的路径信息
* next：路由的控制参数
* next() 跳入下一个页面
* next(’/path’) 改变路由的跳转方向，使其跳到另一个路由
* next(false) 返回原来的页面
* next((vm)=>{}) 仅在 beforeRouteEnter 中可用，vm 是组件实例

### axios异步请求

> 1:安装axios

```shell script
npm install --save axios vue-axios
```

> 2:main.js引入axios

```vue
import axios from 'axios'
import VueAxios from 'vue-axios'
Vue.use(VueAxios, axios)
```

> 3:准备测试数据

**只有我们的 static 目录下的文件是可以被访问到的，所以我们就把静态文件放入该目录下。**
**数据和之前用的json数据一样 需要的去上述axios例子里**

![vue10.png](https://gitee.com/youzhengjie/Java-Study/raw/master/doc/images/vue10.png)

> 4：使用

**Home.vue**

```vue
<script>
    export default {
        name: "Home",
        beforeRouteEnter:((to, from, next) => {
          console.log('进入路由之前');
          next(vm => {
            //进入路由之前执行getData方法
            vm.getData()
          });
        }),
        beforeRouteLeave:((to, from, next) => {
          console.log('离开路由之前')
          next();
        }),
      //axios
      methods: {
        getData: function () {
          this.axios
            .get('http://localhost:8080/static/mock/data.json')
            .then(function (response) {
              console.log(response)
            })
        }
      }
    }
</script>
```
