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
非线程安全就是在多个线程对同一个对象的一个实例变量进行并发访问时出现“脏读”的现象，



