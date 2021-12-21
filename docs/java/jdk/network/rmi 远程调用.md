> 本文来自JavaGuide，郎涯进行简单排版与补充



Java的RMI（Remote Method Invocation）远程调用是指，一个JVM中的代码可以通过网络实现远程调用另一个JVM的某个方法：

- RMI通过自动生成stub和skeleton实现网络调用，客户端只需要查找服务并获得接口实例，服务器端只需要编写实现类并注册为服务；

- RMI的序列化和反序列化可能会造成安全漏洞，因此调用双方必须是内网互相信任的机器，不要把1099端口暴露在公网上作为对外服务。

```ascii
┌ ─ ─ ─ ─ ─ ─ ─ ─ ┐         ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
  ┌─────────────┐                                 ┌─────────────┐
│ │   Service   │ │         │                     │   Service   │ │
  └─────────────┘                                 └─────────────┘
│        ▲        │         │                            ▲        │
         │                                               │
│        │        │         │                            │        │
  ┌─────────────┐   Network   ┌───────────────┐   ┌─────────────┐
│ │ Client Stub ├─┼─────────┼>│Server Skeleton│──>│Service Impl │ │
  └─────────────┘             └───────────────┘   └─────────────┘
└ ─ ─ ─ ─ ─ ─ ─ ─ ┘         └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

> Java的RMI调用机制决定了双方必须是Java程序，其他语言很难调用Java的RMI。
>
> 如果要使用不同语言进行RPC调用，可以选择更通用的协议，例如[gRPC](https://grpc.io/)。



实现一个最简单的RMI：服务器会提供一个`WorldClock`服务，允许客户端获取指定时区的时间

1、服务器和客户端必须共享同一个接口。我们定义一个`WorldClock`接口，代码如下：

```java
public interface WorldClock extends Remote {
    LocalDateTime getLocalDateTime(String zoneId) throws RemoteException;
}
```

RMI规定此接口必须派生自`java.rmi.Remote`，并在每个方法声明抛出`RemoteException`



2、编写服务器的实现类

```java
public class WorldClockService implements WorldClock {
    @Override
    public LocalDateTime getLocalDateTime(String zoneId) throws RemoteException {
        return LocalDateTime.now(ZoneId.of(zoneId)).withNano(0);
    }
}
```



编写的服务以RMI的形式暴露在网络上，客户端才能调用：

```java
public class Server {
    public static void main(String[] args) throws RemoteException {
        System.out.println("create World clock remote service...");
        
        // 实例化一个WorldClock:
        WorldClock worldClock = new WorldClockService();
        // 将此服务转换为远程服务接口:
        WorldClock skeleton = (WorldClock) UnicastRemoteObject.exportObject(worldClock, 0);
        
        // 将RMI服务注册到1099端口:
        Registry registry = LocateRegistry.createRegistry(1099);
        // 注册此服务，服务名为"WorldClock":
        registry.rebind("WorldClock", skeleton);
    }
}
```



3、编写客户端代码

```java
public class Client {
    public static void main(String[] args) throws RemoteException, NotBoundException {
        // 连接到服务器localhost，端口1099:
        Registry registry = LocateRegistry.getRegistry("localhost", 1099);
        // 查找名称为"WorldClock"的服务并强制转型为WorldClock接口:
        WorldClock worldClock = (WorldClock) registry.lookup("WorldClock");
        
        // 正常调用接口方法:
        LocalDateTime now = worldClock.getLocalDateTime("Asia/Shanghai");
        // 打印调用结果:
        System.out.println(now);
    }
}
```

> Thrift、Dubbo 远程调用框架