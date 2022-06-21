>  本文来自捡田螺的小男孩，郎涯进行简单排版与补充

平时我们写代码呢，多数情况都是**流水线式**写代码，基本就可以实现业务逻辑了。**如何在写代码中找到乐趣呢**，我觉得，最好的方式就是：**使用设计模式优化自己的业务代码**。今天跟大家聊聊日常工作中，我都使用过哪些设计模式。

![工作中常用到哪些设计模式](https://img-note.langyastudio.com/202206162223066.webp?x-oss-process=style/watermark)



## 策略模式

### 业务场景

假设有这样的业务场景，大数据系统把文件推送过来，根据不同类型采取**不同的解析**方式。多数的小伙伴就会写出以下的代码：

```java
if(type=="A"){
   //按照A格式解析
 
}else if(type=="B"){
    //按B格式解析
}else{
    //按照默认格式解析
}
```

这个代码可能会存在哪些**问题呢**？

- 如果分支变多，这里的代码就会变得**臃肿，难以维护，可读性低**。
- 如果你需要接入一种新的解析类型，那只能在**原有代码上修改**。

说得专业一点的话，就是以上代码，违背了面向对象编程的**开闭原则**以及**单一原则**。

- **开闭原则**（对于扩展是开放的，但是对于修改是封闭的）：增加或者删除某个逻辑，都需要修改到原来代码
- **单一原则**（规定一个类应该只有一个发生变化的原因）：修改任何类型的分支逻辑代码，都需要改动当前类的代码。

如果你的代码就是酱紫：有多个`if...else`等条件分支，并且每个条件分支，可以封装起来替换的，我们就可以使用**策略模式**来优化。



### 策略模式定义

**策略模式**定义了算法族，分别封装起来，让它们之间可以相互替换，此模式让算法的变化独立于使用算法的的客户。这个策略模式的定义是不是有点抽象呢？那我们来看点通俗易懂的比喻：

> 假设你跟不同性格类型的小姐姐约会，要用不同的策略，有的请电影比较好，有的则去吃小吃效果不错，有的去逛街买买买最合适。当然，目的都是为了得到小姐姐的芳心，请看电影、吃小吃、逛街就是不同的策略。

策略模式针对一组算法，将每一个算法封装到具有共同接口的独立的类中，从而使得它们可以相互替换。



### 策略模式使用

策略模式怎么使用呢？酱紫实现的：

- 一个接口或者抽象类，里面两个方法（一个方法匹配类型，一个可替换的逻辑实现方法）
- 不同策略的差异化实现(就是说，不同策略的实现类)
- 使用策略模式



#### 一个接口，两个方法

```java
public interface IFileStrategy {
    
    //属于哪种文件解析类型
    FileTypeResolveEnum gainFileType();
    
    //封装的公用算法（具体的解析方法）
    void resolve(Object objectparam);
}
```



#### 不同策略的差异化实现

A 类型策略具体实现

```java
@Component
public class AFileResolve implements IFileStrategy {
    
    @Override
    public FileTypeResolveEnum gainFileType() {
        return FileTypeResolveEnum.File_A_RESOLVE;
    }

    @Override
    public void resolve(Object objectparam) {
      logger.info("A 类型解析文件，参数：{}",objectparam);
      //A类型解析具体逻辑
    }
}
```

B 类型策略具体实现

```java
@Component
public class BFileResolve implements IFileStrategy {
   
    @Override
    public FileTypeResolveEnum gainFileType() {
        return FileTypeResolveEnum.File_B_RESOLVE;
    }


    @Override
    public void resolve(Object objectparam) {
      logger.info("B 类型解析文件，参数：{}",objectparam);
      //B类型解析具体逻辑
    }
}
```

默认类型策略具体实现

```java
@Component
public class DefaultFileResolve implements IFileStrategy {

    @Override
    public FileTypeResolveEnum gainFileType() {
        return FileTypeResolveEnum.File_DEFAULT_RESOLVE;
    }

    @Override
    public void resolve(Object objectparam) {
      logger.info("默认类型解析文件，参数：{}",objectparam);
      //默认类型解析具体逻辑
    }
}
```



#### 使用策略模式

如何使用呢？我们借助`spring`的生命周期，使用`ApplicationContextAware`接口，把对用的策略，初始化到`map`里面。然后对外提供`resolveFile`方法即可。

```java
/**
 *  @author 公众号：捡田螺的小男孩
 */
@Component
public class StrategyUseService implements ApplicationContextAware{
  
    private Map<FileTypeResolveEnum, IFileStrategy> iFileStrategyMap = new ConcurrentHashMap<>();
    public void resolveFile(FileTypeResolveEnum fileTypeResolveEnum, Object objectParam) {
        IFileStrategy iFileStrategy = iFileStrategyMap.get(fileTypeResolveEnum);
        if (iFileStrategy != null) {
            iFileStrategy.resolve(objectParam);
        }
    }

    //把不同策略放到map
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        Map<String, IFileStrategy> tmepMap = applicationContext.getBeansOfType(IFileStrategy.class);
        tmepMap.values().forEach(strategyService -> iFileStrategyMap.put(strategyService.gainFileType(), strategyService));
    }
}
```



## 责任链模式

### 业务场景

我们来看一个常见的业务场景，下订单。下订单接口，基本的逻辑，一般有参数非空校验、安全校验、黑名单校验、规则拦截等等。很多伙伴会使用异常来实现：

```java
public class Order {

    public void checkNullParam(Object param){
      //参数非空校验
      throw new RuntimeException();
    }
    public void checkSecurity(){
      //安全校验
      throw new RuntimeException();
    }
    public void checkBackList(){
        //黑名单校验
        throw new RuntimeException();
    }
    public void checkRule(){
        //规则拦截
        throw new RuntimeException();
    }

    public static void main(String[] args) {
        Order order= new Order();
        try{
            order.checkNullParam();
            order.checkSecurity ();
            order.checkBackList();
            order2.checkRule();
            System.out.println("order success");
        }catch (RuntimeException e){
            System.out.println("order fail");
        }
    }
}
```

这段代码使用了**异常**来做逻辑条件判断，如果后续逻辑越来越复杂的话，会出现一些问题：如异常只能返回异常信息，不能返回更多的字段，这时候需要**自定义异常类**。

并且，阿里开发手册规定：**禁止用异常做逻辑判断**。

> 【强制】 异常不要用来做流程控制，条件控制。

说明：异常设计的初衷是解决程序运行中的各种意外情况，且异常的处理效率比条件判断方式要低很多。

如何优化这段代码呢？可以考虑**责任链模式**



### 责任链模式定义

当你想要让一个**以上的对象**有机会能够处理某个请求的时候，就使用**责任链模式**。

> 责任链模式为请求创建了一个接收者对象的链。执行链上有多个对象节点，每个对象节点都有机会（条件匹配）处理请求事务，如果某个对象节点处理完了，就可以根据实际业务需求传递给下一个节点继续处理或者返回处理完毕。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。

责任链模式实际上是一种处理请求的模式，它让多个处理器（对象节点）都有机会处理该请求，直到其中某个处理成功为止。责任链模式把多个处理器串成链，然后让请求在链上传递：

![责任链模式](https://img-note.langyastudio.com/202206162223194.webp?x-oss-process=style/watermark)

打个比喻：

> 假设你晚上去上选修课，为了可以走点走，坐到了最后一排。来到教室，发现前面坐了好几个漂亮的小姐姐，于是你找张纸条，写上：“你好, 可以做我的女朋友吗？如果不愿意请向前传”。纸条就一个接一个的传上去了，后来传到第一排的那个妹子手上，她把纸条交给老师，听说老师40多岁未婚...



### 责任链模式使用

责任链模式怎么使用呢？

- 一个接口或者抽象类
- 每个对象差异化处理
- 对象链（数组）初始化（连起来）

#### 一个接口或者抽象类

这个接口或者抽象类，需要：

- 有一个指向责任下一个对象的属性
- 一个设置下一个对象的set方法
- 给子类对象差异化实现的方法（如以下代码的doFilter方法）

```java
/**
 *  关注公众号：捡田螺的小男孩
 */
public abstract class AbstractHandler {

    //责任链中的下一个对象
    private AbstractHandler nextHandler;

    /**
     * 责任链的下一个对象
     */
    public void setNextHandler(AbstractHandler nextHandler){
        this.nextHandler = nextHandler;
    }

    /**
     * 具体参数拦截逻辑,给子类去实现
     */
    public void filter(Request request, Response response) {
        doFilter(request, response);
        if (getNextHandler() != null) {
            getNextHandler().filter(request, response);
        }
    }

    public AbstractHandler getNextHandler() {
        return nextHandler;
    }

     abstract void doFilter(Request filterRequest, Response response);

}
```



#### 每个对象差异化处理

责任链上，每个对象的**差异化**处理，如本小节的业务场景，就有参数校验对象、安全校验对象、黑名单校验对象、规则拦截对象

```java
/**
 * 参数校验对象
 **/
@Component
@Order(1) //顺序排第1，最先校验
public class CheckParamFilterObject extends AbstractHandler {

    @Override
    public void doFilter(Request request, Response response) {
        System.out.println("非空参数检查");
    }
}

/**
 *  安全校验对象
 */
@Component
@Order(2) //校验顺序排第2
public class CheckSecurityFilterObject extends AbstractHandler {

    @Override
    public void doFilter(Request request, Response response) {
        //invoke Security check
        System.out.println("安全调用校验");
    }
}
/**
 *  黑名单校验对象
 */
@Component
@Order(3) //校验顺序排第3
public class CheckBlackFilterObject extends AbstractHandler {

    @Override
    public void doFilter(Request request, Response response) {
        //invoke black list check
        System.out.println("校验黑名单");
    }
}

/**
 *  规则拦截对象
 */
@Component
@Order(4) //校验顺序排第4
public class CheckRuleFilterObject extends AbstractHandler {

    @Override
    public void doFilter(Request request, Response response) {
        //check rule
        System.out.println("check rule");
    }
}
```



#### 对象链连起来（初始化）&& 使用

```java
@Component("ChainPatternDemo")
public class ChainPatternDemo {

    //自动注入各个责任链的对象
    @Autowired
    private List<AbstractHandler> abstractHandleList;

    private AbstractHandler abstractHandler;

    //spring注入后自动执行，责任链的对象连接起来
    @PostConstruct
    public void initializeChainFilter(){

        for(int i = 0;i<abstractHandleList.size();i++){
            if(i == 0){
                abstractHandler = abstractHandleList.get(0);
            }else{
                AbstractHandler currentHander = abstractHandleList.get(i - 1);
                AbstractHandler nextHander = abstractHandleList.get(i);
                currentHander.setNextHandler(nextHander);
            }
        }
    }

    //直接调用这个方法使用
    public Response exec(Request request, Response response) {
        abstractHandler.filter(request, response);
        return response;
    }

    public AbstractHandler getAbstractHandler() {
        return abstractHandler;
    }

    public void setAbstractHandler(AbstractHandler abstractHandler) {
        this.abstractHandler = abstractHandler;
    }
}
```

运行结果如下：

```java
非空参数检查
安全调用校验
校验黑名单
check rule
复制代码
```



## 模板方法模式

### 业务场景

假设我们有这么一个业务场景：内部系统不同商户，调用我们系统接口，去跟外部第三方系统交互（http方式）。走类似这么一个流程，如下：

![img](https://img-note.langyastudio.com/202206162224727.webp?x-oss-process=style/watermark)

一个请求都会经历这几个流程：

- 查询商户信息
- 对请求报文加签
- 发送http请求出去
- 对返回的报文验签

这里，有的商户可能是走代理出去的，有的是走直连。假设当前有A，B商户接入，不少伙伴可能这么实现，伪代码如下：

```java
// 商户A处理句柄
CompanyAHandler implements RequestHandler {
   Resp hander(req){
   //查询商户信息
   queryMerchantInfo();
   //加签
   signature();
   //http请求（A商户假设走的是代理）
   httpRequestbyProxy()
   //验签
   verify();
   }
}
// 商户B处理句柄
CompanyBHandler implements RequestHandler {
   Resp hander(Rreq){
   //查询商户信息
   queryMerchantInfo();
   //加签
   signature();
   // http请求（B商户不走代理，直连）
   httpRequestbyDirect();
   // 验签
   verify(); 
   }
}
```

假设新加一个C商户接入，你需要再实现一套这样的代码。显然，这样代码就**重复**了，**一些通用的方法，却在每一个子类都重新写了这一方法**。

如何优化呢？可以使用**模板方法模式**。



### 模板方法模式定义

定义一个操作中的算法的骨架流程，而将一些步骤延迟到子类中，使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。它的核心思想就是：定义一个操作的一系列步骤，对于某些暂时确定不下来的步骤，就留给子类去实现，这样不同的子类就可以定义出不同的步骤。

打个通俗的比喻：

> 模式举例：追女朋友要先“牵手”，再“拥抱”，再“接吻”， 再“拍拍..额..手”。至于具体你用左手还是右手牵，无所谓，但是整个过程，定了一个流程模板，按照模板来就行。



### 模板方法使用

- 一个抽象类，定义骨架流程（抽象方法放一起）
- 确定的共同方法步骤，放到抽象类（去除抽象方法标记）
- 不确定的步骤，给子类去差异化实现

我们继续那以上的举例的业务流程例子，来一起用 模板方法优化一下哈：



#### 一个抽象类，定义骨架流程

因为一个个请求经过的流程为一下步骤：

- 查询商户信息
- 对请求报文加签
- 发送http请求出去
- 对返回的报文验签

所以我们就可以定义一个抽象类，包含请求流程的几个方法，方法首先都定义为抽象方法哈：

```java
/**
 * 抽象类定义骨架流程（查询商户信息，加签，http请求，验签）
 */
abstract class AbstractMerchantService  { 

      //查询商户信息
      abstract queryMerchantInfo();
      //加签
      abstract signature();
      //http 请求
      abstract httpRequest();
      // 验签
      abstract verifySinature();
 
}
```



#### 确定的共同方法步骤，放到抽象类

```java
abstract class AbstractMerchantService  { 

     //模板方法流程
     Resp handlerTempPlate(req){
           //查询商户信息
           queryMerchantInfo();
           //加签
           signature();
           //http 请求
           httpRequest();
           // 验签
           verifySinature();
     }
      // Http是否走代理（提供给子类实现）
      abstract boolean isRequestByProxy();
}
```



#### 不确定的步骤，给子类去差异化实现

因为是否走代理流程是**不确定**的，所以给子类去实现。

商户A的请求实现：

```java
CompanyAServiceImpl extends AbstractMerchantService{
    Resp hander(req){
      return handlerTempPlate(req);
    }
    //走http代理的
    boolean isRequestByProxy(){
       return true;
    }
}
```

商户B的请求实现：

```java
CompanyBServiceImpl extends AbstractMerchantService{
    Resp hander(req){
      return handlerTempPlate(req);
    }
    //公司B是不走代理的
    boolean isRequestByProxy(){
       return false;
    }
}
```



## 观察者模式

### 业务场景

登陆注册应该是最常见的业务场景了。就拿**注册**来说事，我们经常会遇到类似的场景，就是用户注册成功后，我们给用户发一条消息，又或者发个邮件等等，因此经常有如下的代码：

```java
void register(User user){
  insertRegisterUser（user）;
  sendIMMessage();
  sendEmail()；
}
```

这块代码会有什么问题呢？ 如果产品又加需求：现在注册成功的用户，再给用户发一条短信通知。于是你又得改register方法的代码了。。。这是不是违反了**开闭原则**啦。

```java
void register(User user){
  insertRegisterUser（user）;
  sendIMMessage();
  sendMobileMessage（）;
  sendEmail()；
}
```

并且，如果调**发短信的接口失败**了，是不是又影响到用户注册了？！这时候，是不是得加个异步方法给**通知消息**才好。。。

实际上，我们可以使用观察者模式优化。



### 观察者模式定义

> 观察者模式定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被完成业务的更新。

观察者模式属于行为模式，一个对象（被观察者）的状态发生改变，所有的依赖对象（观察者对象）都将得到通知，进行广播通知。它的主要成员就是**观察者和被观察者**。

- 被观察者（Observerable）：目标对象，状态发生变化时，将通知所有的观察者。
- 观察者（observer）：接受被观察者的状态变化通知，执行预先定义的业务。

**使用场景：** 完成某件事情后，异步通知场景。如，登陆成功，发个IM消息等等。



### 观察者模式使用

观察者模式实现的话，还是比较简单的。

- 一个被观察者的类Observerable ;
- 多个观察者Observer ；
- 观察者的差异化实现
- 经典观察者模式封装：EventBus实战



#### 一个被观察者的类Observerable 和 多个观察者Observer

```java
public class Observerable {
   
   private List<Observer> observers 
      = new ArrayList<Observer>();
   private int state;
 
   public int getState() {
      return state;
   }
 
   public void setState(int state) {
      notifyAllObservers();
   }
 
   //添加观察者
   public void addServer(Observer observer){
      observers.add(observer);      
   }
   
   //移除观察者
   public void removeServer(Observer observer){
      observers.remove(observer);      
   }
   //通知
   public void notifyAllObservers(int state){
      if(state!=1){
          System.out.println(“不是通知的状态”);
         return ;
      }
   
      for (Observer observer : observers) {
         observer.doEvent();
      }
   }  
}
```



#### 观察者的差异化实现

```java
 //观察者
 interface Observer {  
    void doEvent();  
}  
//Im消息
IMMessageObserver implements Observer{
    void doEvent（）{
       System.out.println("发送IM消息");
    }
}

//手机短信
MobileNoObserver implements Observer{
    void doEvent（）{
       System.out.println("发送短信消息");
    }
}
//EmailNo
EmailObserver implements Observer{
    void doEvent（）{
       System.out.println("发送email消息");
    }
}
```



#### EventBus实战

自己搞一套观察者模式的代码，还是有点小麻烦。实际上，`Guava EventBus `就封装好了，它 提供一套基于注解的事件总线，api可以灵活的使用，爽歪歪。

我们来看下`EventBus`的实战代码哈，首先可以声明一个EventBusCenter类，它类似于以上被观察者那种角色`Observerable`。

```java
public class EventBusCenter {

    private static EventBus eventBus = new EventBus();

    private EventBusCenter() {
    }

    public static EventBus getInstance() {
        return eventBus;
    }
     //添加观察者
    public static void register(Object obj) {
        eventBus.register(obj);
    }
    //移除观察者
    public static void unregister(Object obj) {
        eventBus.unregister(obj);
    }
    //把消息推给观察者
    public static void post(Object obj) {
        eventBus.post(obj);
    }
}
```

然后再声明观察者`EventListener`

```java
public class EventListener {

    @Subscribe //加了订阅，这里标记这个方法是事件处理方法  
    public void handle(NotifyEvent notifyEvent) {
        System.out.println("发送IM消息" + notifyEvent.getImNo());
        System.out.println("发送短信消息" + notifyEvent.getMobileNo());
        System.out.println("发送Email消息" + notifyEvent.getEmailNo());
    }
}

//通知事件类
public class NotifyEvent  {

    private String mobileNo;

    private String emailNo;

    private String imNo;

    public NotifyEvent(String mobileNo, String emailNo, String imNo) {
        this.mobileNo = mobileNo;
        this.emailNo = emailNo;
        this.imNo = imNo;
    }
 }
```

使用demo测试：

```java
public class EventBusDemoTest {

    public static void main(String[] args) {

        EventListener eventListener = new EventListener();
        EventBusCenter.register(eventListener);
        EventBusCenter.post(new NotifyEvent("13372817283", "123@qq.com", "666"));
        }
}
```

运行结果：

```
发送IM消息666
发送短信消息13372817283
发送Email消息123@qq.com
复制代码
```



## 工厂模式

### 业务场景

工厂模式一般配合策略模式一起使用。用来去优化大量的`if...else...`或`switch...case...`条件语句。

我们就取第一小节中策略模式那个例子吧。根据不同的文件解析类型，创建不同的解析对象

```java
 IFileStrategy getFileStrategy(FileTypeResolveEnum fileType){
     IFileStrategy  fileStrategy ;
     if(fileType=FileTypeResolveEnum.File_A_RESOLVE){
       fileStrategy = new AFileResolve();
     }else if(fileType=FileTypeResolveEnum.File_A_RESOLV){
       fileStrategy = new BFileResolve();
     }else{
       fileStrategy = new DefaultFileResolve();
     }
     return fileStrategy;
 }
```

其实这就是**工厂模式**，定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。

策略模式的例子，没有使用上一段代码，而是借助spring的特性，搞了一个工厂模式，哈哈，小伙伴们可以回去那个例子细品一下，我把代码再搬下来，小伙伴们再品一下吧：

```java
/**
 *  @author 公众号：捡田螺的小男孩
 */
@Component
public class StrategyUseService implements ApplicationContextAware{

    private Map<FileTypeResolveEnum, IFileStrategy> iFileStrategyMap = new ConcurrentHashMap<>();

    //把所有的文件类型解析的对象，放到map，需要使用时，信手拈来即可。这就是工厂模式的一种体现啦
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        Map<String, IFileStrategy> tmepMap = applicationContext.getBeansOfType(IFileStrategy.class);
        tmepMap.values().forEach(strategyService -> iFileStrategyMap.put(strategyService.gainFileType(), strategyService));
    }
}
```



### 使用工厂模式

定义工厂模式也是比较简单的:

- 一个工厂接口，提供一个创建不同对象的方法
- 其子类实现工厂接口，构造不同对象
- 使用工厂模式

#### 一个工厂接口

```java
interface IFileResolveFactory{
   void resolve();
}
```



#### 不同子类实现工厂接口

```java
class AFileResolve implements IFileResolveFactory{
   void resolve(){
      System.out.println("文件A类型解析");
   }
}

class BFileResolve implements IFileResolveFactory{
   void resolve(){
      System.out.println("文件B类型解析");
   }
}

class DefaultFileResolve implements IFileResolveFactory{
   void resolve(){
      System.out.println("默认文件类型解析");
   }
}
```



### 使用工厂模式

```java
//构造不同的工厂对象
IFileResolveFactory fileResolveFactory;
if(fileType=“A”){
    fileResolveFactory = new AFileResolve();
}else if(fileType=“B”){
    fileResolveFactory = new BFileResolve();
 }else{
    fileResolveFactory = new DefaultFileResolve();
}

fileResolveFactory.resolve();
```

一般情况下，对于工厂模式，你不会看到以上的代码。工厂模式会跟配合其他设计模式如策略模式一起出现的。



## 单例模式

### 6.1 业务场景

单例模式，**保证一个类仅有一个实例**，并提供一个访问它的全局访问点。 I/O与数据库的连接,一般就用单例模式实现de的。Windows里面的Task Manager（任务管理器）也是很典型的单例模式。

来看一个单例模式的例子

```java
/**
 *  公众号：捡田螺的小男孩
 */
public class LanHanSingleton {

    private static LanHanSingleton instance;

    private LanHanSingleton(){

    }

    public static LanHanSingleton getInstance(){
        if (instance == null) {
            instance = new LanHanSingleton();
        }
        return instance;
    }

}
```

以上的例子，就是**懒汉式**的单例实现。实例在需要用到的时候，才去创建，就比较懒。如果有则返回，没有则新建，需要加下 `synchronized `关键字，要不然可能存在**线性安全问题**。



### 单例模式的经典写法

其实单例模式还有有好几种实现方式，如饿汉模式，双重校验锁，静态内部类，枚举等实现方式。

#### 饿汉模式

```java
public class EHanSingleton {

   private static EHanSingleton instance = new EHanSingleton();
   
   private EHanSingleton(){      
   }

   public static EHanSingleton getInstance() {
       return instance;
   }
   
}
```

饿汉模式，它**比较饥饿、比较勤奋**，实例在初始化的时候就已经建好了，不管你后面有没有用到，都先新建好实例再说。这个就没有线程安全的问题，但是呢，浪费内存空间呀。



#### 双重校验锁

```java
public class DoubleCheckSingleton {

   private static DoubleCheckSingleton instance;

   private DoubleCheckSingleton() { }
   
   public static DoubleCheckSingleton getInstance(){
       if (instance == null) {
           synchronized (DoubleCheckSingleton.class) {
               if (instance == null) {
                   instance = new DoubleCheckSingleton();
               }
           }
       }
       return instance;
   }
}
```

双重校验锁实现的单例模式，综合了懒汉式和饿汉式两者的优缺点。以上代码例子中，在synchronized关键字内外都加了一层  `if `条件判断，这样既保证了线程安全，又比直接上锁提高了执行效率，还节省了内存空间。



#### 静态内部类

```java
public class InnerClassSingleton {

   private static class InnerClassSingletonHolder{
       private static final InnerClassSingleton INSTANCE = new InnerClassSingleton();
   }

   private InnerClassSingleton(){}
   
   public static final InnerClassSingleton getInstance(){
       return InnerClassSingletonHolder.INSTANCE;
   }
}
```

静态内部类的实现方式，效果有点类似双重校验锁。但这种方式只适用于静态域场景，双重校验锁方式可在实例域需要延迟初始化时使用。



#### 枚举

```java
public enum SingletonEnum {

    INSTANCE;
    public SingletonEnum getInstance(){
        return INSTANCE;
    }
}
```

枚举实现的单例，代码简洁清晰。并且它还自动支持序列化机制，绝对防止多次实例化。