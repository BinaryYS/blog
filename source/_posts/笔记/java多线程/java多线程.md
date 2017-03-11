---
title: java多线程核心技术
date: 2017-02-06 16:10:05
tags: [java,thread]
categories: 笔记
---
## Java多线程技能
### 停止线程
虽然interrupt有停止、中止的意思，但是不能直接停止当前运行的线程，<!-- more -->需要通过调用interrupted或者isInterrupted的状态来判断
*interrupted*:测试当前线程是否处于中断状态，具有清除状态标识将其置为false的功能
*isInterrupted*:测试当前线程是否处于中断状态，不改变状态值
``` sql
  public class Interrupt extends Thread{
      public void run(){
          for (int i=0; i<500000; i++){
              super.run();
              if (this.isInterrupted()){
                  System.out.println("线程已经停止！退出");
                  break;
              }
              System.out.println("i=:" + (i+1));
          }
      }
  }
  
  --测试
  public class Interrupted_test {
  
      public static void main(String[] args) {
  
          try {
              Interrupt interrupt = new Interrupt();
              interrupt.setName("A");
              interrupt.start();
              Thread.sleep(2000);
              interrupt.interrupt();
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
```

#### 异常法
 通过捕获异常interruptedException，停止线程
``` sql
public class Interrupt extends Thread{
    public void run(){
        try {
            for (int i=0; i<500000; i++){
                super.run();
                if (this.isInterrupted()){
                    System.out.println("线程已经停止！退出");
                    throw new InterruptedException();
                }
                System.out.println("i=:" + (i+1));
            }
        System.out.println("我还要继续执行呢，狠狠！");
        }catch (InterruptedException e) {
            System.out.println("I am interruptedException , i am catched by run! wu wu ....");
            e.printStackTrace();
        }

    }
}
```

#### 在线程沉睡中停止
在线程sleep时调用interrupted
``` sql
public class SleepInterrupt extends Thread{

    public void run() {
    try{
        System.out.println("run begin--> oo oo");
        Thread.sleep(2000);
        System.out.println("run end--> ......");
    } catch(InterruptedException e) {
        System.out.println("I am dead in sleeping ......");
        e.printStackTrace();
    }

    }
}
```

#### 暴力停止stop
用stop的方法停止线程的方式不推荐使用，这是一种不安全的停止方式,暴力停止可能会导致一些清理操作无法执行。

#### return停止
线程run()方法中直接return便退出线程。

### 暂停、恢复线程
*suspend()*: 暂停线程
*resume()*: 复线程
这种方法极易造成公共同步对象的独占，阻止其他线程访问公共同步方法;同时该方法也容易造成数据不同步的问题。
``` sql
public class PrintLock extends Thread {
    private long i = 0;
    public void run(){
        while (true) {
           // if (this.isInterrupted()) return;
            i++;
           //System.out.println("i=:"+i);<!--试试-->
        }
    }
}

--测试
public class PrintLock_test {
    public static void main(String[] args) {
        try {
            PrintLock printLock = new PrintLock();
            printLock.setName("binary");
            printLock.start();
            Thread.sleep(1000);
            printLock.suspend();
            System.out.println("main end!");

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
当线程获取print()的同步锁未释放时，主线程无法使用println(), println()源码如下：
``` sql
 public void println(String x) {
        synchronized (this) {
            print(x);
            newLine();
        }
    }
```
### yield
 线程调用yield()让出cpu，之后再获取cpu继续执行，期间间隔的时间不确定。

### 线程的优先级
线程优先级有10个等级，1-10，设置优先级setPriority()。优先级较高的线程得到的cpu资源较多，也就是说cpu优先执行优先级较高的线程中对象中的任务。
#### 优先级的继承性
线程的优先级与启动的线程的优先级一样。
#### 优先级具有规则性
#### 优先级具有随机性
### 守护线程
Java线程中有两种线程，一种用户线程，另一种为守护线程。守护线程为非守护线程服务，当进程中没有非守护线程的时候，守护线程就会自动销毁，典型的如垃圾回收线程GC。只要当前的JVM虚拟机中存在任何一个非守护线程实例，守护线程就会一直工作。setDaemon()将线程设置为守护线程。

## 对象及变量的并发访问
### synchronized 方法
非线程安全就是在多个线程对同一个对象的一个实例变量进行并发访问时出现“脏读”的现象，读到的数据是被修改过的，线程安全就是获取的实例变量的值是同步处理的，避免脏读现象。
#### 实例变量的线程安全

方法内的变量是线程安全的，然而多个线程并发访问同一个对象的实例变量就是非线程安全的，此时会出现脏读。
``` sql
 public class synch {
     //private int num = 0;  非线程安全
      public void addI(String name) {
       int num = 0;
         if(name.eq("a")){
              num = 100;
              try
             { Thread.sleep(1000);}
             catch(InterruptedException e){
                     e.printStackTrace();
             }
         else{
             num = 200;
             }
         }
      }
 }
 
```

这种情况就是在addI()方法加上synchronized锁。

#### 多个对象多个锁
synchronized只能锁住同一个对象实例的一个方法，同一对象的多个实例不受约束，通过synchronized锁住的方法只能顺序访问。

#### 脏读
``` sql
public class MyObject {

    private String name = "A";
    private String pwd = "AA";

    synchronized public void setValue(String _name, String _pwd){
        this.name = _name;
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.pwd = _pwd;

    }

   /*synchronized*/ public void getValue(){
        System.out.println(Thread.currentThread().getName()+"name:"+this.name+"&"+"pwd:"+this.pwd);
    }
}


/**
 * Created by song.yang on 2017/3/2：17:00.
 * <p>
 * e-mail:song.yang@msxf.com
 */

public class MyThread extends Thread {
    private MyObject mo;
    public MyThread(MyObject _mo) {
        this.mo = _mo;
    }

    public void run(){
        mo.setValue("B", "BB");
    }

}


public class MyTest {
    public static void main(String[] args) {
        MyObject mo = new MyObject();
        MyThread mt = new MyThread(mo);
        mt.start();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        mo.getValue();
    }


}
```
线程调用对象的同步方法获得对象锁，其他线程不能获取对象的锁，即其他线程不能调用该对象的所有同步方法，但是其他线程可以调用该对象的非同步方法。

#### synchronized 锁重入

一个线程获取对象锁之后，是可以连续请求以连续获得该对象的锁，即在一个synchronized修饰的方法块内再次调用该对象的其他同步方法是可以的。

#### synchronized 锁释放
出现异常，直接自动释放锁

#### 同步不具有继承性
synchronized方法不能继承，在子类方法中添加synchronized修饰才能实现同步。
### synchronized同步语句块
使用synchronized容易出现长时间等待的现象
#### 锁非this对象
非this锁不会与this锁争抢对象锁，这样可以大大提高执行效率，不会发生长时间等待。
``` sql
/**
 * Created by song.yang on 2017/3/7：14:02.
 * <p>
 * e-mail:song.yang@msxf.com
 */

public class MyService {
    public MyList addMyService(MyList ml,String str){
        try {
            synchronized(ml){
            if (ml.getSize()<1){
                Thread.sleep(2000);
                ml.addList(str);
            }
            }
        }catch (InterruptedException e) {
            e.printStackTrace();
        }
        return ml;
    }
}
```

####静态同步synchronized方法与synchronized（class）代码块
synchronized加到静态方法上，锁住的是class;synchronized加到非静态方法上，锁住的是对象。而锁住的是class锁可以对类的所有实例对象起作用
#### 数据类型String的常量池特性
JVM中具有String常量池缓存的功能，对象不具有
``` sql
/**
 * Created by song.yang on 2017/3/9：13:47.
 * <p>
 * e-mail:song.yang@msxf.com
 */

public class Service {
    public  void print(String object){
        synchronized(object){
            while (true){
                System.out.println(Thread.currentThread().getName());
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
### Volatile
 volatile是强制从公共堆栈中获取变量值，而不是从线程私有数据栈中获取变量值
#### volatile与synchronized的比较
关键字volatile是线程同步的轻量级实现，性能比synchronized好，volatile只能修数变量，而synchronized可以修饰方法和代码块，随着JDK版本的提升，synchronized的性能得到提升；多线程访问volatile不会发生阻塞，而多线程访问synchronized会发生阻塞；volatile可以保证数据的可见性，但不能保证数据的原子性，synchronized可以保证数据的原子性，也可以间接保证数据的可见性，因为它将会同步私有内存数据域和公共内存数据域的变量值；volatile可以保证多个线程访问变量的可见性，synchronized保证多个线程访问资源的同步性。volatile只能保证线程从内存中加载数据时是最新的，也就是读数据的时候是最新的，多个线程访问同一个实例变量还是要加锁保证同步。

## 线程间通信
### 共享实例变量实现通信
### 通知/等待机制
wait/notify通知等待机制，二者都要在同步方法中，都需要获得对象锁。
*wait():* 使调用wait的线程释放共享资源的锁，进入等待队列，直到再次被唤醒
*notify():* 随机唤醒等待队列中等待同一个资源的线程，被选中的线程退出等待队列，进入可执行状态
*notifyall():* 唤醒等待队列中所有等待同一资源的线程，全部进入可运行状态，此时优先级最高的线程最先运行，也可能随机执行，具体取决于JVM的实现机制
wait()执行之后会自动释放锁，notify()不会自动释放锁，只有在其所在的同步块执行完之后才释放锁
#### wait(long)
 在等待一段时间之内没被唤醒则自动唤醒
 
### join()
join()是让所属的线程t正常执行，其他线程进入阻塞队列，在t执行完成之后，阻塞队列中的线程才能继续执行。
*join(long):* 设置等待时间，底层实现是wait(long), 能够自动释放当前锁，之后其他线程可以获取该对象的同步方法
*sleep(long):* 与join(long)不同的是sleep(long)不能自动释放锁

### ThreadLocal
ThreadLocal为线程绑定私有变量

## Lock的使用
### 使用ReentrantLock类
ReentrantLock与synchronized作用类似，有lock()和nulock();

## Timer
## 单例模式与多线程
### 立即加载/饿汉模式
立即加载就是在使用类的时候已经创建完毕，常见的方法就是new实例化，而立即加载有着急、急迫的意思，所以称为饿汉模式
### 延迟加载/懒汉模式
延迟加载就是调用get()方法时候才创建实例，常见的情况就是直接在get()方法中实例化，而延迟加载就是在从中文的语境的来看有缓慢和不急迫的意思，故称为懒汉模式。
延迟加载根本不能实现安全单例，只能在get()方法加同步synchronized锁才可以,但是该方法效率不高，也可以改成synchronized代码块的方法，但是同样效率低
#### 使用DCL双检查锁机制
``` sql
/**
 * Created by song.yang on 2017/3/11：14:52.
 * <p>
 * e-mail:song.yang@msxf.com
 */

public class MyObject {

    private volatile static MyObject myObject;
    private MyObject(){

    }

    public static MyObject getInstance(){
        if (myObject != null){

        }else {
            try {
                Thread.sleep(1000);
                synchronized (MyObject.class){
                    if (myObject == null){
                        myObject = new MyObject();
                    }
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        return myObject;
    }
}
```
#### 使用静态内置类实现锁机制
``` sql
/**
 * Created by song.yang on 2017/3/11：15:17.
 * <p>
 * e-mail:song.yang@msxf.com
 */

public class StaticClass {
    private static final Logger logger = LoggerFactory.getLogger(StaticClass.class);
    private static class StaticHandler{
        private static StaticClass staticClass = new StaticClass();
    }

    private StaticClass(){
    }

    public static StaticClass getInstance(){
        logger.info("静态内置类实现单例模式。。。。");
        return  StaticHandler.staticClass;
    }
}
```

### 序列化和反序列化的单例模式实现
静态内置类能够达到单例模式的效果，但是在实现序列化的时候，如果按照默认的的方式则结果还是多例的
``` sql
/**
 * Created by song.yang on 2017/3/11：15:44.
 * <p>
 * e-mail:song.yang@msxf.com
 */

public class MyObject implements Serializable{
    private static final Logger logger = LoggerFactory.getLogger(MyObject.class);
    private final static long serialVersionUID = -1L;
    private static class ObjectClass{
        private static final MyObject myObject = new MyObject();
    }
    private MyObject(){

    }

    public static MyObject getInstance(){
        return ObjectClass.myObject;
    }
/**
 那么这个readResolve()方法是从哪来的，为什么加上之后就能返回同一实例了呢？
 找到ObjectInputStream类的
 * Reads and returns "ordinary" (i.e., not a String, Class,
 * ObjectStreamClass, array, or enum constant) object, or null if object's
 * class is unresolvable (in which case a ClassNotFoundException will be
 * associated with object's handle).  Sets passHandle to object's assigned
 * handle.

private Object readOrdinaryObject(boolean unshared)
        throws IOException
{
    if (bin.readByte() != TC_OBJECT) {
        throw new InternalError();
    }

    ObjectStreamClass desc = readClassDesc(false);
    desc.checkDeserialize();

    Class<?> cl = desc.forClass();
    if (cl == String.class || cl == Class.class
            || cl == ObjectStreamClass.class) {
        throw new InvalidClassException("invalid class descriptor");
    }

    Object obj;
    try {
        obj = desc.isInstantiable() ? desc.newInstance() : null;
    } catch (Exception ex) {
        throw (IOException) new InvalidClassException(
                desc.forClass().getName(),
                "unable to create instance").initCause(ex);
    }

    passHandle = handles.assign(unshared ? unsharedMarker : obj);
    ClassNotFoundException resolveEx = desc.getResolveException();
    if (resolveEx != null) {
        handles.markException(passHandle, resolveEx);
    }

    if (desc.isExternalizable()) {
        readExternalData((Externalizable) obj, desc);
    } else {
        readSerialData(obj, desc);
    }

    handles.finish(passHandle);

    if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
    {
        Object rep = desc.invokeReadResolve(obj);
        if (unshared && rep.getClass().isArray()) {
            rep = cloneArray(rep);
        }
        if (rep != obj) {
            handles.setObject(passHandle, obj = rep);
        }
    }

    return obj;
}
 */
     protected Object readResolve(){
        System.out.println("调用了readResolve方法！");
        return  ObjectClass.myObject;
    }
}
```

### 使用static代码块实现单例模式
static代码块中的代码代码在使用类的时候已经执行了
``` sql
/**
 * Created by song.yang on 2017/3/11：16:50.
 * <p>
 * e-mail:song.yang@msxf.com
 */

public class StaticSingleton {
    private static StaticSingleton instance = null;
    private StaticSingleton(){

    }
    static {
        instance = new StaticSingleton();
    }
    public static StaticSingleton getInstance(){
        return instance;
    }
}
```
### 使用enum枚举数据类型实现单例模式
在使用枚举数据类型时，*构造方法*会被自动调用，可以根据这个特性实现单例模式

## 拾遗增补
### 线程状态
调用线程的getState(), 线程状态new,runnable,running,terminated,timed_waiting,blocked,waiting,

### 线程组
### 线程对象关联线程组：1级关联
父对象与子对象
### 线程 对象关联线程组：多级关联
父对象与子对象以及孙子对象

### 线程具有有序性

### SimpleDateFormate非线程安全
SimpleDateFormate类是非线程安全的，
