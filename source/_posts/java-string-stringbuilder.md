---
title: java中String、StringBuffer、StringBuilder之前的区别
date: 2022-05-13 15:38:07
tags: 技术杂谈
---

## 使用场景

String、StringBuffer、StringBuilder都可以应用于字符串的处理，但是在内部实现上还是有区别的。

- 在String内部可以发现有这样的代码`private final char value[];`是用来存储字符用的，final修饰的就表明一旦初始化就无法更改，那么也就导致对String的操作始终都是产生新的String对象，这样的话，频繁更改String字符串，会导致大量的内存分配，从而给GC造成很大压力，影响程序的性能
- StringBuffer/StringBuilder内部默认都是创建容量为16的`char[] value;`数组，当使用append追加字符串的时候，会首先判断添加的字符串长度是否已经超过数组剩余的长度，如果没有则直接在末尾追加，并计算剩余的可用长度，如果超过，则会进行扩容的操作，重新计算新的长度（`int newCapacity = (value.length << 1) + 2;`）并创建一个新的char[]数组，将原来的数组copy过去（`value = Arrays.copyOf(value,newCapacity(minimumCapacity));`）,通过源代码分析，可以发现StringBuffer与StringBuilder本质上的区别就是：StringBuffer支持多线程的，内部使用同步操作的机制，而StringBuilder是线程不安全的，只能使用在单线程的场景

通过对比分析，我们可以知道如果是字符串频繁的修改操作，建议使用StringBuffer/StringBuilder，而如果是在多线程的应用场景的话，选择StringBuffer，单线程就选择StringBuilder，还有一点我们必须要注意，StringBuffer/StringBuilder默认创建的时候指定的容量大小为16，我们可以根据实际的应用场景指定初始化的大小，以避免频繁的扩容操作，从而影响程序的性能

## 性能比较

我们可以简单地比较一下使用String、StringBuffer与StringBuilder的性能究竟如何，可以通过获取程序的执行时间（当然我们也可以使用jmh-core去更详细地比较相关的性能指标，这个在后续的文章当中专门介绍一下）

定义一个接口`StringTestService`

```java
package com.sherman.service;

public interface StringTestService {
    void testString(int loopCount);

    void testStringBuffer(int loopCount);

    void testStringBuilder(int loopCount);

    void testStringBufferBySpecifiedCapacity(int loopCount);

    void testStringBuilderBySpecifiedCapacity(int loopCount);
}
```

实现接口：

```java
package com.sherman.service.impl;

import com.sherman.service.StringTestService;
import org.springframework.stereotype.Service;

@Service
public class StringTestServiceImpl implements StringTestService {
    @Override
    public void testString(int loopCount) {
        long start = System.currentTimeMillis();
        String str = "";
        for (int i = 0; i < loopCount; i++) {
            str += String.format("loop：%s", i);
        }
        long end = System.currentTimeMillis();
        System.out.println(String.format("testString spend total time: %d ms", end - start));
    }

    @Override
    public void testStringBuffer(int loopCount) {
        long start = System.currentTimeMillis();
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < loopCount; i++) {
            sb.append(String.format("loop：%s", i));
        }
        long end = System.currentTimeMillis();
        System.out.println(String.format("testStringBuffer spend total time: %d ms", end - start));
    }

    @Override
    public void testStringBuilder(int loopCount) {
        long start = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < loopCount; i++) {
            sb.append(String.format("loop：%s", i));
        }

        long end = System.currentTimeMillis();
        System.out.println(String.format("testStringBuilder spend total time: %d ms", end - start));
    }

    @Override
    public void testStringBufferBySpecifiedCapacity(int loopCount) {
        long start = System.currentTimeMillis();
        StringBuffer sb = new StringBuffer(loopCount);
        for (int i = 0; i < loopCount; i++) {
            sb.append(String.format("loop：%s", i));
        }
        long end = System.currentTimeMillis();
        System.out.println(String.format("testStringBufferBySpecifiedCapacity spend total time: %d ms", end - start));
    }

    @Override
    public void testStringBuilderBySpecifiedCapacity(int loopCount) {
        long start = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder(loopCount);
        for (int i = 0; i < loopCount; i++) {
            sb.append(String.format("loop：%s", i));
        }

        long end = System.currentTimeMillis();
        System.out.println(String.format("testStringBuilderBySpecifiedCapacity spend total time: %d ms", end - start));
    }
}
```

可以发现实现的服务里面有很多重复的代码，针对这种情况，我们可以使用aop的方式进行重构，有兴趣的同学也可以自己去尝试一下，我也打算放在单独的文章当中介绍如何通过aop的方式实现日志的记录。暂且先忍受一下，重点是比较执行时间。

```java
package com.sherman;

import com.sherman.service.StringTestService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Hello world!
 *
 */
@SpringBootApplication
public class App implements CommandLineRunner
{
    @Autowired
    private StringTestService stringTestService;

    public static void main( String[] args )
    {
        SpringApplication.run(App.class, args);
        System.out.println( "Hello World!" );
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println("程序的真正入口地址");
        int loop = 1000000;
        stringTestService.testString(loop);
        stringTestService.testStringBuffer(loop);
        stringTestService.testStringBuilder(loop);
        stringTestService.testStringBufferBySpecifiedCapacity(loop);
        stringTestService.testStringBuilderBySpecifiedCapacity(loop);}
}

```

程序的执行结果如下：

