### 传统部署模式

#### Session 模式

Session 模式是预分配资源的，也就是提前根据指定的资源参数初始化**一个 Flink 集群**，并常驻在 YARN 系统中，拥有固定数量的 JobManager 和 TaskManager（注意 JobManager 只有一个）。提交到这个集群的作业可以直接运行，免去每次分配资源的 overhead。但是 Session 的资源总量有限，多个作业之间又不是隔离的，故可能会造成资源的争用；如果有一个 TaskManager 宕机，它上面承载着的所有作业也都会失败。另外启动的作业越多，JobManager 的负载也就越大。所以 Session 模式一般用来部署那些对**延迟非常敏感但运行时长较短**的作业。



#### Per-Job 模式

顾名思义在 Per-Job 模式下，每个提交到 YARN 上的作业会各自形成**单独的 Flink 集群**，拥有专属的 JobManager 和TaskManager。可见以 Per-Job 模式提交作业的启动延迟可能会较高，但是作业之间的资源完全隔离，一个作业的TaskManager 失败不会影响其他作业的运行，JobManager 的负载也是分散开来的，不存在单点问题。当作业运行完成，与它关联的集群也就被销毁，资源被释放。所以，Per-Job 模式一般用来部署那些**长时间运行**的作业。



#### 存在的问题

上文所述 Session 模式和 Per-Job 模式可以用如下的简图表示，其中红色、蓝色和绿色的图形代表不同的作业。

![195230-743909a210f3d23a](https://img-note.langyastudio.com/202210271457947.webp?x-oss-process=style/watermark)

Deployer 代表向 YARN 集群发起部署请求的节点，一般来讲在生产环境中，也总有这样一个节点作为所有作业的提交入口（即客户端）。在 main() 方法开始执行直到 env.execute() 方法之前，客户端也需要做一些工作，即：

- 获取作业所需的依赖项
- 通过执行环境分析并取得逻辑计划，即 StreamGraph→JobGraph
- 将依赖项和 JobGraph 上传到集群中

只有在这些都完成之后，才会通过 env.execute() 方法触发 Flink 运行时真正地开始执行作业。试想如果所有用户都在Deployer 上提交作业，较大的依赖会消耗更多的带宽，而较复杂的作业逻辑翻译成 JobGraph 也需要吃掉更多的 CPU 和内存，**客户端的资源反而会成为瓶颈**——不管 Session 还是 Per-Job 模式都存在此问题。为了解决它，社区在传统部署模式的基础上实现了 Application 模式。



### Application 模式

![195230-35e05a5d309d953c](https://img-note.langyastudio.com/202210271458470.webp?x-oss-process=style/watermark)

可见，原本需要客户端做的三件事被转移到了 JobManager 里，也就是说 main() 方法在集群中执行（入口点位于ApplicationClusterEntryPoint），Deployer 只需要负责发起部署请求了。另外，如果一个 main() 方法中有多个env.execute()/executeAsync() 调用，在 Application 模式下，这些作业会被视为属于同一个应用，在同一个集群中执行（如果在 Per-Job 模式下，就会启动多个集群）。可见 **Application 模式本质上是 Session 和 Per-Job 模式的折中**。