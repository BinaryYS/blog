---
title: java多线程核心技术
date: 2017-02-06 16:10:05
tags: [java,thread]
categories: 笔记
---
## interrupted & isInterrupted
虽然interrupt有停止、中止的意思，但是不能直接停止当前运行的线程，<!-- more -->需要通过调用interrupted或者isInterrupted的状态来判断
*interrupted*:测试当前线程是否处于中断状态，具有清除状态标识将其置为false的功能
*isInterrupted*:测试当前线程是否处于中断状态，不改变状态值
``` bash
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
### 线程停止的方法
#### 异常法
 通过捕获异常interruptedException，停止线程
``` bash
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
``` bash
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
用stop的方法停止线程的方式不推荐使用，这是一种不安全的停止方式。





