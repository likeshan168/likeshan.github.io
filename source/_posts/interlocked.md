---
title: Interlocked中CompareExchange的用法
date: 2022-05-05 23:09:44
tags: 技术杂谈
---

## CompareExchange使用说明

方法签名：public static int CompareExchange(ref int location1, int value, int comparand);  location1与comparand进行比较，如果相等，则用value替换location1的值，并返回location1被替换之前的值，例如：

```C#
int a = 0;
int b = Interlocked.CompareExchange(ref a, 1, 0);
Console.WriteLine($"a is {a}, b is {b}");
```

输出结果：a is 1, b is 0

此方法是原子操作，意味着比较和替换值的操作是线程安全的，那么这就可以使用在多线程当中。

## CompareExchange示例

```C#
//模拟多线程操作
var calculationHelper = new CalculationHelper();
int ordinaryValue = 0;
ManualResetEventSlim manualResetEvent = new ManualResetEventSlim(false);

Thread thread1 = new Thread(new ThreadStart(Test));
thread1.Name = "线程1";
thread1.Start();

Thread thread2 = new Thread(new ThreadStart(Test));
thread2.Name = "线程2";
thread2.Start();

manualResetEvent.Set();//多个等待的线程可以执行

thread1.Join();
thread2.Join();

Console.WriteLine($"通过线程安全的方式计算结果：{calculationHelper.Total}，非线程安全计算出的结果为：{ordinaryValue}");

void Test()
{
    manualResetEvent.Wait();//等待信号
    for (int i = 1; i <= 10000; i++)
    {
        ordinaryValue += i;
        calculationHelper.AddTotal(i);
    }
}

/// <summary>
/// 计算帮助类
/// </summary>
public class CalculationHelper
{
    private int total = 0;
    public int Total { get { return total; } }

    /// <summary>
    /// 累加
    /// </summary>
    /// <param name="value">需要加的值</param>
    /// <returns></returns>
    public int AddTotal(int value)
    {
        if (value == 0)
        {
            return value;
        }
        int localValue, compuetedValue;
        do
        {
            localValue = total;
            compuetedValue = localValue + value;
        } while (localValue != Interlocked.CompareExchange(ref total, compuetedValue, localValue));//说明计算成功了

        return compuetedValue;
    }
}
```

可以多次运行，观察结果，发现线程安全的方法返回的结果都是固定的，而非线程安全返回的值是变化的，如下是运行两次的结果

`通过线程安全的方式计算出的结果：100010000，非线程安全计算出的结果为：97383348`

`通过线程安全的方式计算出的结果：100010000，非线程安全计算出的结果为：93608564`

