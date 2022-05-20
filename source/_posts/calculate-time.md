---
title: Java中获取执行时间的几种方式
date: 2022-05-16 22:58:12
tags: 技术杂谈
---

## 应用场景

有的时候，我们需要查看某一段代码的性能如何，最为简单的方式，可以通过计算该段代码执行的耗时，来进行简单的判断，那么我们在java中可以通过以下几种方式获取程序的执行耗时。

## 代码示例

1. 通过 `System.currentTimeMillis()`方法可以获取当前时间的毫秒数据，那么就可以在开始执行的地方记录一个当前时间的毫秒数值，程序执行结束的时候获取一个当前时间的毫秒数值，取两个时间的差值即为程序的耗时，代码如下：

   ```java
    /**
     * 通过 <code>System.currentTimeMillis()</code>获取执行的时长
     * @throws InterruptedException
     */
   public void getExecuteTimeByCurrentTimeMillis() throws InterruptedException {
       long start = System.currentTimeMillis();
       Thread.sleep(1000);
       long end = System.currentTimeMillis();
       System.out.println(String.format("Total Time：%d ms", end - start));
   }
   ```

2. 通过`System.nanoTime()`方法，该方法和`System.currentTimeMillis()`类似，只是返回的是纳秒。

   ```java
   /**
    * 通过<code>System.nanoTime()</code>获取执行时长，该方法返回的单位是纳秒
    * 
    * @throws InterruptedException
    */
   public void getExecuteTimeByNanoTime() throws InterruptedException {
       long start = System.nanoTime();
       Thread.sleep(1000);
       long end = System.nanoTime();
       System.out.println(String.format("Total Time：%d ms", (end - start) / 1000000));
   }
   ```

3. 通过`java.util.Date`初始化一个时间类型的对象，在程序的开始地方用于表示开始时间，程序结束时再初始化一个时间对象，然后计算两个时间的差值，也就是执行的时长。

   ```java
   /**
    * 通过java.util.Date初始化开始和结束时间，计算两个时间的差值得出执行时间
    * 
    * @throws InterruptedException
    */
   public void getExecuteTimeByDate() throws InterruptedException {
       Date startDate = new Date();
       Thread.sleep(1000);
       Date endDate = new Date();
       System.out.println(String.format("Total time：%d ms", endDate.getTime() - startDate.getTime()));
   }
   ```

4. 通过`commons.lang3`中的`StopWatch`，需要引入如下的依赖

   ```xml
   <dependency>
         <groupId>org.apache.commons</groupId>
         <artifactId>commons-lang3</artifactId>
         <version>3.12.0</version>
   </dependency>
   ```

   ```java
   /**
    * 通过引入apache 的 commons.lang3包，使用StopWatch获取执行时间
    * 
    * @throws InterruptedException
    */
   public void getExecuteByStopWatch1() throws InterruptedException {
       StopWatch stopWatch = new StopWatch();
       stopWatch.start();
       Thread.sleep(1000);
       stopWatch.stop();
       System.out.println(String.format("Total time：%d ms", stopWatch.getTime()));
   }
   ```

5. 通过`com.google.guava`中`Stopwatch`，需要引入如下的依赖

   ```xml
   <dependency>
         <groupId>com.google.guava</groupId>
         <artifactId>guava</artifactId>
         <version>31.1-jre</version>
         <!-- <type>bundle</type> -->
   </dependency>
   ```

   ```java
   /**
    * 引入com.google.guava中的Guava包，使用Stopwatch获取执行时间
    * @throws InterruptedException
    */
   public void getExecuteByStopWatch2() throws InterruptedException {
       Stopwatch stopwatch = Stopwatch.createStarted();
       Thread.sleep(1000);
       stopwatch.stop();
       System.out.println(String.format("Total time：%d ms", stopwatch.elapsed(TimeUnit.MILLISECONDS)));
   }
   ```

6. 通过spring中的`StopWatch`获取执行时间，我这里是使用的spring boot搭建的控制台程序

   ```java
   /**
    * 通过spring中的StopWatch获取执行时间
    * @throws InterruptedException
    */
   public void getExecuteByStopWatch3() throws InterruptedException {
       // TODO Auto-generated method stub
       org.springframework.util.StopWatch stopWatch = new org.springframework.util.StopWatch();
       stopWatch.start();
       Thread.sleep(1000);
       stopWatch.stop();
       System.out.println(String.format("Total time：%d ms", stopWatch.getTotalTimeMillis()));
   }
   ```

通过以上6种方式都可以获取到程序的执行耗时，不知道大家是否还有其他的方式呢？欢迎在评论区讨论。
