## OOM

### SQL in 过多

> https://mp.weixin.qq.com/s/g5Y47cQ25KbVjzHhZcjN7g

面对 OOM 问题如果代码不是有明显的问题，下面几个JVM参数相当有用，尤其是在容器化之后。

```shell
-XX:+HeapDumpOnOutOfMemoryError -XX:ErrorFile=/logs/oom_dump/xxx.log -XX:HeapDumpPath=/logs/oom_dump/xxx.hprof
```

另外提一个参数也很有用，正常来说如果程序出现 OOM 之后，就是有代码存在内存泄漏的风险，这个时候即使能对外提供服务，其实也是有风险的，可能造成更多的请求有问题，所以该参数非常有必要，可以让 K8S 快速的再拉起来一个实例。

```shell
-XX:+ExitOnOutOfMemoryError
```

另外，针对这两个非常类似的问题，对于 SQL 语句，如果监测到没有`where`条件的全表查询应该默认增加一个合适的`limit`作为限制，防止这种问题拖垮整个系统。



## CPU 

### 线程池Bug

> https://mp.weixin.qq.com/s/EqOTtYO8K1AteO3ADnX-aw

```java
public static void main(String[] args){
    ScheduledExecutorService e = Executors.newScheduledThreadPool(0);
    e.schedule(() -> {
        System.out.println("业务逻辑");
    }, 60, TimeUnit.SECONDS);
    e.shutdown();
}
```



### MySQL 进程飙升 900%

> https://mp.weixin.qq.com/s/MZam941snuuV5AXlLDEj2A

并发量大并且大量 SQL 性能低的情况下，比如字段是**没有建立索引**，则会导致快速 CPU 飙升，如果还**开启了慢日志记录**，会导致性能更加恶化

**定位过程：**

- 使用 top 命令观察，确定是 mysqld 导致还是其他原因
- 如果是 mysqld 导致的，show processlist，查看 session 情况，确定是不是有消耗资源的 sql 在运行
- 找出消耗高的 sql，看看执行计划是否准确， index 是否缺失，或者实在是数据量太大造成



### Java 进程飙升 900%

> https://mp.weixin.qq.com/s/MZam941snuuV5AXlLDEj2A

一旦高并发场景，要么走到了死循环，要么就是在做大量的 GC

**定位过程：**

CPU飙升问题定位的一般步骤是：

- 首先通过 top 指令查看当前占用 CPU 较高的进程 PID

- 查看当前进程消耗资源的线程 PID：**top -Hp** PID

- 通过 print 命令将线程 PID 转为 16 进制，根据该 16 进制值去打印的堆栈日志内查询，查看该线程所驻留的方法位置

- 通过 jstack 命令，查看栈信息，定位到线程对应的具体代码

- 分析代码解决问题

**处理过程：**

- 如果是空循环，或者空自旋

处理方式：可以使用 Thread.sleep 或者 加锁，让线程适当的阻塞

- 在循环的代码逻辑中，创建大量的新对象导致频繁 GC。比如，从 mysql 查出了大量的数据，比如 100W 以上等等。

处理方式：可以减少对象的创建数量或者可以考虑使用对象池

- 其他的一些造成 CPU 飙升的场景，比如  selector 空轮训导致 CPU 飙升 

处理方式：参考 Netty 源码，无效的事件查询到了一定的次数，进行 selector 重建



### ConcurrentHashMap 死循环

在 JDK1.8 版本中，ConcurrentHashMap 中存在死循环 bug，下面是 bug 重现代码：

```java
package com.hiwe.demo.cache;
import java.util.concurrent.ConcurrentHashMap;

public class TestCache {
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>(16);
        map.computeIfAbsent(
                "AaAa",
                key -> {
                    return map.computeIfAbsent(
                            "BBBB",
                            key2 -> 42);
                }
        );
        System.out.println("程序完成");
    }
    
    or
    
    public static void main(String[] args) {
        Map map =new ConcurrentHashMap();
        map.computeIfAbsent("a", key -> {
            map.put("a", "v2");
            return"v1";
        });
    }    
}
```

执行以上代码，程序会进入死循环状态，导致 CPU 使用率暴涨。

- bug 原因

因为 "AaAa" 和 “BBBB” 的 hash 值相同，会定位到用一个 bucket 中，这样就形成了 CAS 嵌套，产生死循环问题。具体的可以看源码分析

- 解决

禁止在向 ConcurrentHashMap 中嵌套执行 computeIfAbsent/putIfAbsent 操作

```java
V value = map.get(k);
if (value == null) {
    V newValue = computeValue(k);  // 这里对computeValue(k)的重复调用不敏感
    value = map.putIfAbsent(k, newValue);
   if (value == null) {
        return newValue;
   }

   return value;
}
```























