> 本文来自廖雪峰，郎涯进行简单排版与补充



## DAO

- Spring 提供了 `JdbcDaoSupport` 来便于我们实现 DAO 模式

- 可以基于泛型实现更通用、更简洁的 DAO 模式

编写数据访问层的时候，可以使用 DAO 模式。DAO 即 Data Access Object 的缩写，它没有什么神秘之处，实现起来基本如下：

```java
public class UserDao {
    @Autowired
    JdbcTemplate jdbcTemplate;

    User getById(long id) {
        ...
    }

    List<User> getUsers(int page) {
        ...
    }

    User createUser(User user) {
        ...
    }

    User updateUser(User user) {
        ...
    }

    void deleteUser(User user) {
        ...
    }
}
```



Spring 提供了一个 `JdbcDaoSupport` 类，用于简化 DAO 的实现。这个 `JdbcDaoSupport`没什么复杂的，核心代码就是持有一个 `JdbcTemplate`：

```java
public abstract class JdbcDaoSupport extends DaoSupport {

    private JdbcTemplate jdbcTemplate;

    public final void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        initTemplateConfig();
    }

    public final JdbcTemplate getJdbcTemplate() {
        return this.jdbcTemplate;
    }

    ...
}
```

它的意图是子类直接从 `JdbcDaoSupport` 继承后，可以随时调用 `getJdbcTemplate()` 获得 `JdbcTemplate` 的实例。那么问题来了：因为 `JdbcDaoSupport` 的 `jdbcTemplate `字段没有标记 `@Autowired`，所以，子类想要注入`JdbcTemplate`，还得自己想个办法：

可以编写一个 `AbstractDao`，专门负责注入 `JdbcTemplate`：

> 如果使用传统的 XML 配置，并不需要编写`@Autowired JdbcTemplate jdbcTemplate`

```java
public abstract class AbstractDao extends JdbcDaoSupport {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @PostConstruct
    public void init() {
        super.setJdbcTemplate(jdbcTemplate);
    }
}
```

这样，子类的代码就非常干净，可以直接调用`getJdbcTemplate()`：

```java
@Component
@Transactional
public class UserDao extends AbstractDao {
    public User getById(long id) {
        return getJdbcTemplate().queryForObject(
                "SELECT * FROM users WHERE id = ?",
                new BeanPropertyRowMapper<>(User.class),
                id
        );
    }
    ...
}
```



倘若肯再多写一点样板代码，就可以把 `AbstractDao` 改成泛型，并实现 `getById()`，`getAll()`，`deleteById()`这样的通用方法：

```java
public abstract class AbstractDao<T> extends JdbcDaoSupport {
    private String table;
    private Class<T> entityClass;
    private RowMapper<T> rowMapper;

    public AbstractDao() {
        // 获取当前类型的泛型类型:
        this.entityClass = getParameterizedType();
        this.table = this.entityClass.getSimpleName().toLowerCase() + "s";
        this.rowMapper = new BeanPropertyRowMapper<>(entityClass);
    }

    public T getById(long id) {
        return getJdbcTemplate().queryForObject("SELECT * FROM " + table + " WHERE id = ?", this.rowMapper, id);
    }

    public List<T> getAll(int pageIndex) {
        int limit = 100;
        int offset = limit * (pageIndex - 1);
        return getJdbcTemplate().query("SELECT * FROM " + table + " LIMIT ? OFFSET ?",
                new Object[] { limit, offset },
                this.rowMapper);
    }

    public void deleteById(long id) {
        getJdbcTemplate().update("DELETE FROM " + table + " WHERE id = ?", id);
    }
    ...
}
```

这样，每个子类就自动获得了这些通用方法：

```java
@Component
@Transactional
public class UserDao extends AbstractDao<User> {
    // 已经有了:
    // User getById(long)
    // List<User> getAll(int)
    // void deleteById(long)
}

@Component
@Transactional
public class BookDao extends AbstractDao<Book> {
    // 已经有了:
    // Book getById(long)
    // List<Book> getAll(int)
    // void deleteById(long)
}
```

> 可见，DAO 模式就是一个简单的数据访问模式，是否使用 DAO，根据实际情况决定，因为很多时候，直接在 Service 层操作数据库也是完全没有问题的



## JDBC

Spring 为了简化数据库访问，主要做了以下几点工作：

- 提供了简化的访问 JDBC 的模板类，不必手动释放资源
- 提供了一个统一的 DAO 类以实现 Data Access Object 模式
- 把 `SQLException` 封装为 `DataAccessException`，这个异常是一个 `RuntimeException`，并且让我们能区分SQL异常的原因，例如，`DuplicateKeyException` 表示违反了一个唯一约束
- 能方便地集成 Hibernate、JPA 和 MyBatis 这些数据库访问框架



**只是对 JDBC 操作的一个简单封装，它的目的是尽量减少手动编写`try(resource) {...}`的代码**

- Spring提供了 `JdbcTemplate` 来简化 JDBC 操作

- 使用 `JdbcTemplate` 时，根据需要优先选择高级方法

- 任何 JDBC 操作都可以使用保底的 `execute(ConnectionCallback)` 方法



在 Spring 使用 JDBC，首先我们通过 IoC 容器创建并管理一个`DataSource`实例，然后 Spring 提供了一个`JdbcTemplate`，可以方便地让我们操作 JDBC，因此通常情况下，我们会实例化一个`JdbcTemplate`。顾名思义，这个类主要使用了[Template模式](https://www.liaoxuefeng.com/wiki/1252599548343744/1281319636041762)。

引入依赖：

```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.2.0.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>javax.annotation</groupId>
        <artifactId>javax.annotation-api</artifactId>
        <version>1.3.2</version>
    </dependency>    
```

在 AppConfig 中，我们需要创建以下几个必须的Bean：

```java
@Configuration
@ComponentScan
@PropertySource("jdbc.properties")
public class AppConfig {

    @Value("${jdbc.url}")
    String jdbcUrl;

    @Value("${jdbc.username}")
    String jdbcUsername;

    @Value("${jdbc.password}")
    String jdbcPassword;

    @Bean
    DataSource createDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(jdbcUrl);
        config.setUsername(jdbcUsername);
        config.setPassword(jdbcPassword);
        config.addDataSourceProperty("autoCommit", "true");
        config.addDataSourceProperty("connectionTimeout", "5");
        config.addDataSourceProperty("idleTimeout", "60");
        return new HikariDataSource(config);
    }

    @Bean
    JdbcTemplate createJdbcTemplate(@Autowired DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

写一个配置文件`jdbc.properties`：

```ini
# 数据库文件名为testdb:
jdbc.url=jdbc:mysql://localhost:3306/testdb
jdbc.username=root
jdbc.password=xxx
```

只需要在需要访问数据库的Bean中，注入`JdbcTemplate`即可：

```java
@Component
public class UserService {
    @Autowired
    JdbcTemplate jdbcTemplate;
    ...
}
```



### JdbcTemplate用法

首先我们看`T execute(ConnectionCallback<T> action)`方法，它提供了Jdbc的`Connection`供我们使用：

**允许获取Connection，然后做任何基于Connection的操作**

```java
public User getUserById(long id) {
    // 注意传入的是ConnectionCallback:
    return jdbcTemplate.execute((Connection conn) -> {
        // 可以直接使用conn实例，不要释放它，回调结束后JdbcTemplate自动释放:
        // 在内部手动创建的PreparedStatement、ResultSet必须用try(...)释放:
        try (var ps = conn.prepareStatement("SELECT * FROM users WHERE id = ?")) {
            ps.setObject(1, id);
            try (var rs = ps.executeQuery()) {
                if (rs.next()) {
                    return new User( // new User object:
                            rs.getLong("id"), // id
                            rs.getString("email"), // email
                            rs.getString("password"), // password
                            rs.getString("name")); // name
                }
                throw new RuntimeException("user not found by id.");
            }
        }
    });
}
```

再看`T execute(String sql, PreparedStatementCallback<T> action)`的用法：

```java
public User getUserByName(String name) {
    // 需要传入SQL语句，以及PreparedStatementCallback:
    return jdbcTemplate.execute("SELECT * FROM users WHERE name = ?", (PreparedStatement ps) -> {
        // PreparedStatement实例已经由JdbcTemplate创建，并在回调后自动释放:
        ps.setObject(1, name);
        try (var rs = ps.executeQuery()) {
            if (rs.next()) {
                return new User( // new User object:
                        rs.getLong("id"), // id
                        rs.getString("email"), // email
                        rs.getString("password"), // password
                        rs.getString("name")); // name
            }
            throw new RuntimeException("user not found by id.");
        }
    });
}
```



最后，我们看`T queryForObject(String sql, Object[] args, RowMapper<T> rowMapper)`方法：

```java
public User getUserByEmail(String email) {
    // 传入SQL，参数和RowMapper实例:
    return jdbcTemplate.queryForObject("SELECT * FROM users WHERE email = ?", new Object[] { email },
            (ResultSet rs, int rowNum) -> {
                // 将ResultSet的当前行映射为一个JavaBean:
                return new User( // new User object:
                        rs.getLong("id"), // id
                        rs.getString("email"), // email
                        rs.getString("password"), // password
                        rs.getString("name")); // name
            });
}
```

在`queryForObject()`方法中，传入SQL以及SQL参数后，`JdbcTemplate`会自动创建`PreparedStatement`，自动执行查询并返回`ResultSet`，我们提供的`RowMapper`需要做的事情就是把`ResultSet`的当前行映射成一个JavaBean并返回。整个过程中，使用`Connection`、`PreparedStatement`和`ResultSet`都不需要我们手动管理。



如果我们期望返回多行记录，而不是一行，可以用`query()`方法：

```java
public List<User> getUsers(int pageIndex) {
    int limit = 100;
    int offset = limit * (pageIndex - 1);
    return jdbcTemplate.query("SELECT * FROM users LIMIT ? OFFSET ?", new Object[] { limit, offset },
            new BeanPropertyRowMapper<>(User.class));
}
```

如果数据库表的结构恰好和 JavaBean 的属性名称一致，那么**`BeanPropertyRowMapper`**就可以直接把一行记录按列名转换为 JavaBean。



如果我们执行的不是查询，而是插入、更新和删除操作，那么需要使用`update()`方法：

```java
public void updateUser(User user) {
    // 传入SQL，SQL参数，返回更新的行数:
    if (1 != jdbcTemplate.update("UPDATE user SET name = ? WHERE id=?", user.getName(), user.getId())) {
        throw new RuntimeException("User not found by id");
    }
}
```



只有一种`INSERT`操作比较特殊，那就是如果某一列是自增列（例如自增主键），通常，我们需要获取插入后的自增值。`JdbcTemplate`提供了一个`KeyHolder`来简化这一操作：

```java
public User register(String email, String password, String name) {
    // 创建一个KeyHolder:
    KeyHolder holder = new GeneratedKeyHolder();
    if (1 != jdbcTemplate.update(
        // 参数1:PreparedStatementCreator
        (conn) -> {
            // 创建PreparedStatement时，必须指定RETURN_GENERATED_KEYS:
            var ps = conn.prepareStatement("INSERT INTO users(email,password,name) VALUES(?,?,?)",
                    Statement.RETURN_GENERATED_KEYS);
            ps.setObject(1, email);
            ps.setObject(2, password);
            ps.setObject(3, name);
            return ps;
        },
        // 参数2:KeyHolder
        holder)
    ) {
        throw new RuntimeException("Insert failed.");
    }
    // 从KeyHolder中获取返回的自增值:
    return new User(holder.getKey().longValue(), email, password, name);
}
```



我们总结一下`JdbcTemplate`的用法，那就是：

- 针对简单查询，优选`query()`和`queryForObject()`，因为只需提供SQL语句、参数和`RowMapper`
- 针对更新操作，优选`update()`，因为只需提供SQL语句和参数
- 任何复杂的操作，最终也可以通过`execute(ConnectionCallback)`实现，因为拿到`Connection`就可以做任何JDBC操作



## MyBatis

Hibernate / JPA  — 这类 ORM 干的主要工作就是把 ResultSet 的每一行变成 Java Bean，或者把 Java Bean 自动转换到 INSERT 或 UPDATE 语句的参数中，从而实现 ORM

| JDBC       | Hibernate      | JPA                  | MyBatis           |
| :--------- | :------------- | :------------------- | :---------------- |
| DataSource | SessionFactory | EntityManagerFactory | SqlSessionFactory |
| Connection | Session        | EntityManager        | SqlSession        |

> Spring Data JPA 是在 JPA 之上所做的更高级别的抽象，使对数据库的操作更简单、更语义化。



介于全自动 ORM 如 Hibernate 和手写全部如 JdbcTemplate 之间，还有一种半自动的 ORM，它只负责把 ResultSet 自动映射到 Java Bean，或者自动填充 Java Bean 参数，但仍需自己写出SQL。[MyBatis ](https://mybatis.org/)就是这样一种半自动化 ORM 框架，**需要手写 SQL 语句**，没有自动加载一对多或多对一关系的功能。

它和 ORM 框架相比，主要有几点差别：

- 查询后需要手动提供 Mapper 实例以便把 ResultSet 的每一行变为 Java 对象

- 增删改操作所需的参数列表，需要手动传入，即把 User 实例变为[user.id, user.name, user.email]这样的列表，比较麻烦

 	

引入MyBatis

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.5.6</version>
</dependency>
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>2.0.6</version>
</dependency>
```

使用 MyBatis 的核心就是创建 `SqlSessionFactory`，这里我们需要创建的是`SqlSessionFactoryBean`：

```java
@Bean
SqlSessionFactoryBean createSqlSessionFactoryBean(@Autowired DataSource dataSource) {
    var sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSource);
    return sqlSessionFactoryBean;
}
```

因为 MyBatis 可以直接使用 Spring 管理的声明式事务，因此，创建事务管理器和使用 JDBC 是一样的：

```java
@Bean
PlatformTransactionManager createTxManager(@Autowired DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}
```



### 注解SQL

和 Hibernate 不同的是，MyBatis 使用 Mapper 来实现映射，而且 Mapper 必须是接口。我们以 User 类为例，在 User类和 users 表之间映射的 UserMapper 编写如下：

```java
public interface UserMapper {
	@Select("SELECT * FROM users WHERE id = #{id}")
	User getById(@Param("id") long id);
}
```

注意：这里的 Mapper 不是 JdbcTemplate 的 RowMapper 的概念，它是定义访问 users 表的接口方法。比如我们定义了一个 `User getById(long)` 的主键查询方法，不仅要定义接口方法本身，还要明确写出查询的SQL，这里用注解`@Select`标记。SQL语句的任何参数，都与方法参数按名称对应。例如，方法参数 id 的名字通过注解`@Param()`标记为`id`，则 SQL 语句里将来替换的占位符就是`#{id}`。



如果有多个参数，那么每个参数命名后直接在SQL中写出对应的占位符即可：

```java
@Select("SELECT * FROM users LIMIT #{offset}, #{maxResults}")
List<User> getAll(@Param("offset") int offset, @Param("maxResults") int maxResults);
```

注意：MyBatis 执行查询后，将根据方法的返回类型自动把 ResultSet 的每一行转换为 User 实例，转换规则当然是按列名和属性名对应。如果列名和属性名不同，最简单的方式是编写 SELECT 语句的别名：

```sql
-- 列名是created_time，属性名是createdAt:
SELECT id, name, email, created_time AS createdAt FROM users
```



执行 INSERT 语句就稍微麻烦点，因为我们希望传入 User 实例，因此定义的方法接口与 `@Insert` 注解如下：

```java
@Insert("INSERT INTO users (email, password, name, createdAt) VALUES (#{user.email}, #{user.password}, #{user.name}, #{user.createdAt})")
void insert(@Param("user") User user);
```

上述方法传入的参数名称是 `user`，参数类型是 User 类，在 SQL 中引用的时候，以`#{obj.property}`的方式写占位符。和 Hibernate 这样的全自动化 ORM 相比，MyBatis 必须写出完整的 INSERT 语句。

如果 `users` 表的 id 是自增主键，那么，我们在 SQL 中不传入 id，但希望获取插入后的主键，需要再加一个 `@Options` 注解：

```java
@Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "id")
@Insert("INSERT INTO users (email, password, name, createdAt) VALUES (#{user.email}, #{user.password}, #{user.name}, #{user.createdAt})")
void insert(@Param("user") User user);
```

`keyProperty` 和 `keyColumn`分别指出 JavaBean 的属性和数据库的主键列名。



执行 UPDATE 和 DELETE 语句相对比较简单，我们定义方法如下：

```java
@Update("UPDATE users SET name = #{user.name}, createdAt = #{user.createdAt} WHERE id = #{user.id}")
void update(@Param("user") User user);

@Delete("DELETE FROM users WHERE id = #{id}")
void deleteById(@Param("id") long id);
```



有了 `UserMapper` 接口，还需要对应的实现类才能真正执行这些数据库操作的方法。虽然可以自己写实现类，但我们除了编写 `UserMapper` 接口外，还有 `BookMapper`、`BonusMapper`……一个一个写太麻烦，因此，MyBatis 提供了一个`MapperFactoryBean` 来自动创建所有 Mapper 的实现类。可以用一个简单的注解来启用它：

```java
@MapperScan("com.itranswarp.learnjava.mapper")
...其他注解...
public class AppConfig {
    ...
}
```

有了`@MapperScan`，就可以让 MyBatis 自动扫描指定包的所有 Mapper 并创建实现类。在真正的业务逻辑中，我们可以直接注入：

```java
@Component
@Transactional
public class UserService {
    // 注入UserMapper:
    @Autowired
    UserMapper userMapper;

    public User getUserById(long id) {
        // 调用Mapper方法:
        User user = userMapper.getById(id);
        if (user == null) {
            throw new RuntimeException("User not found by id.");
        }
        
        return user;
    }
}
```

可见，业务逻辑主要就是通过 `XxxMapper` 定义的数据库方法来访问数据库。



### XML配置

上述在 Spring 中集成 MyBatis 的方式，我们只需要用到注解，并没有任何 XML 配置文件。MyBatis 也允许使用 XML 配置映射关系和 SQL 语句，例如，更新 `User` 时根据属性值构造动态SQL：

```java
<update id="updateUser">
  UPDATE users SET
  <set>
    <if test="user.name != null"> name = #{user.name} </if>
    <if test="user.hobby != null"> hobby = #{user.hobby} </if>
    <if test="user.summary != null"> summary = #{user.summary} </if>
  </set>
  WHERE id = #{user.id}
</update>
```

编写 XML 配置的优点是可以组装出动态 SQL，并且把所有 SQL 操作集中在一起。缺点是配置起来太繁琐，调用方法时如果想查看 SQL 还需要定位到 XML 配置中。这里我们不介绍 XML 的配置方式，需要了解的童鞋请自行阅读 [官方文档](https://mybatis.org/mybatis-3/zh/configuration.html)。

使用 MyBatis 最大的问题是所有 SQL 都需要全部手写，优点是执行的 SQL 就是我们自己写的 SQL，对 SQL 进行优化非常简单，也可以编写任意复杂的 SQL，或者使用数据库的特定语法，但切换数据库可能就不太容易。好消息是大部分项目并没有切换数据库的需求，完全可以针对某个数据库编写尽可能优化的SQL。



## Spring Data Respository

**Spring Data Repository** 为访问不同的数据库提供统一的抽象，它极大地减少了数据访问层的样板代码。Spring Data Repository 抽象的核心是 org.springframework.data.repository. Repository<T,ID>，T 代表它处理的实体的类型，ID 代表实体的唯一标识。它的主要子接口是 CrudRepository，定义了新增、查询、更新和删除的功能接口。PagingAndSortingRepository 是 CrudRepository 的子接口，定义了分页和排序的功能接口。

针对不同的数据库，有特定的子接口抽象，如 JpaRepository、ElasticsearchRepository 等。

可以通过继承上面的接口来定义实体的 Repository 开启配置（@EnableJpaRepositories）注解后，实体的Repository会被Spring Data注册成一个Bean，我们可以使用这个Bean进行数据访问操作。

用注解将相应数据库的领域模型标识为实体，如JPA使用的是@Entity，Elasticsearch使用的是@Document。



## Transactional

> **声明式事务有一个局限，那就是他的最小粒度要作用在方法上**！所以大家在用的时候要格外格外注意大事务的问题，尽量避免在事务中做一些无关数据库的操作，比如 RPC 远程调用、文件解析等，都是血泪的教训啊！！

Spring 提供的声明式事务极大地方便了在数据库中使用事务，正确使用声明式事务的关键在于确定好事务边界，理解事务传播级别。

Spring 提供了一个 `PlatformTransactionManager` 来表示事务管理器，所有的事务都由它负责管理。而事务由`TransactionStatus `表示。如果手写事务代码，使用`try...catch`如下：

```java
TransactionStatus tx = null;
try {
    // 开启事务:
    tx = txManager.getTransaction(new DefaultTransactionDefinition());
    
    // 相关JDBC操作:
    jdbcTemplate.update("...");
    jdbcTemplate.update("...");
    
    // 提交事务:
    txManager.commit(tx);
} catch (RuntimeException e) {
    // 回滚事务:
    txManager.rollback(tx);
    throw e;
}
```

Spring 为了同时支持 JDBC 和 JTA （分布式事务）两种事务模型，就抽象出 `PlatformTransactionManager`。

因为我们的代码只需要 JDBC 事务，因此在 `AppConfig` 中，需要再定义一个 `PlatformTransactionManager` 对应的 Bean，它的实际类型是 `DataSourceTransactionManager`：

```java
@Configuration
@ComponentScan
@PropertySource("jdbc.properties")
public class AppConfig {
    ...
    @Bean
    PlatformTransactionManager createTxManager(@Autowired DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```



**声明式事务**

使用编程的方式使用 Spring 事务仍然比较繁琐，更好的方式是通过声明式事务来实现。使用声明式事务非常简单，除了在 `AppConfig` 中追加一个上述定义的 `PlatformTransactionManager`外，再加一个 `@EnableTransactionManagement`就可以启用声明式事务：

```java
@Configuration
@ComponentScan
@EnableTransactionManagement // 启用声明式
@PropertySource("jdbc.properties")
public class AppConfig {
    ...
}
```

然后，对需要事务支持的方法，加一个 `@Transactional` 注解：

```java
@Component
public class UserService {
    // 此public方法自动具有事务支持:
    @Transactional
    public User register(String email, String password, String name) {
       ...
    }
}
```

或者更简单一点，直接在 Bean 的 `class` 处加上，表示所有 `public` 方法都具有事务支持：

```java
@Component
@Transactional
public class UserService {
    ...
}
```

>  Spring 对一个声明式事务的方法，如何开启事务支持？原理仍然是 **AOP 代理**



### 回滚事务

默认情况下，如果发生了 `RuntimeException`，Spring 的声明式事务将自动回滚。 在一个事务方法中，如果程序判断需要回滚事务，只需抛出 `RuntimeException`，例如：

```java
@Transactional
public buyProducts(long productId, int num) {
    ...
    if (store < num) {
        // 库存不够，购买失败:
        throw new IllegalArgumentException("No enough products");
    }
    ...
}
```



**如果要针对 Checked Exception 回滚事务，需要在 `@Transactional`注解中写出来：**

```java
@Transactional(rollbackFor = {RuntimeException.class, IOException.class})
public buyProducts(long productId, int num) throws IOException {
    ...
}
```

上述代码表示在抛出 `RuntimeException` 或 `IOException` 时，事务将回滚。



为了简化代码，**我们强烈建议业务异常体系从 `RuntimeException` 派生**，这样就不必声明任何特殊异常即可让 Spring 的声明式事务正常工作：

```java
public class BusinessException extends RuntimeException {
    ...
}

public class LoginException extends BusinessException {
    ...
}

public class PaymentException extends BusinessException {
    ...
}
```



### 事务边界

在使用事务的时候，明确事务边界非常重要。对于声明式事务，例如，一个负责给用户增加积分的`addBonus()`方法：

```java
@Component
public class BonusService {
    @Transactional
    public void addBonus(long userId, int bonus) { // 事务开始
       ...
    } // 事务结束
}
```

它的事务边界就是`addBonus()`方法开始和结束。

在现实世界中，问题总是要复杂一点点。用户注册后，能自动获得100积分，因此，实际代码如下：

```java
@Component
public class UserService {
    @Autowired
    BonusService bonusService;

    @Transactional
    public User register(String email, String password, String name) {
        // 插入用户记录:
        User user = jdbcTemplate.insert("...");
        
        // 增加100积分:
        bonusService.addBonus(user.id, 100);
    }
}
```

现在问题来了：调用方（比如`RegisterController`）调用 `UserService.register()` 这个事务方法，它在内部又调用了 `BonusService.addBonus()` 这个事务方法，一共有几个事务？如果 `addBonus()` 抛出了异常需要回滚事务，`register()` 方法的事务是否也要回滚？



### 事务传播

要解决上面的问题，我们首先要定义事务的传播模型。

Spring 的声明式事务为事务传播定义了几个级别，默认传播级别就是 REQUIRED，它的意思是，如果当前没有事务，就创建一个新事务，**如果当前有事务，就加入到当前事务中执行**。

我们观察 `UserService.register()` 方法，它在 `RegisterController` 中执行，因为 `RegisterController` 没有事务，因此，`UserService.register()` 方法会自动创建一个新事务。

在 `UserService.register()` 方法内部，调用 `BonusService.addBonus()` 方法时，因为 `BonusService.addBonus()` 检测到当前已经有事务了，因此，它会加入到当前事务中执行。

因此，整个业务流程的事务边界就清晰了：它只有一个事务，并且范围就是 `UserService.register()` 方法。



#### **传播级别**

- `SUPPORTS`：表示如果有事务，就加入到当前事务，如果没有，那也不开启事务执行。这种传播级别可用于查询方法，因为SELECT语句既可以在事务内执行，也可以不需要事务

- `MANDATORY`：表示必须要存在当前事务并加入执行，否则将抛出异常。这种传播级别可用于核心更新逻辑，比如用户余额变更，它总是被其他事务方法调用，不能直接由非事务方法调用

- `REQUIRES_NEW`：表示不管当前有没有事务，都必须开启一个新的事务执行。如果当前已经有事务，那么当前事务会挂起，等新事务完成后，再恢复执行

- `NOT_SUPPORTED`：表示不支持事务，如果当前有事务，那么当前事务会挂起，等这个方法执行完成后，再恢复执行

- `NEVER`：和`NOT_SUPPORTED`相比，它不但不支持事务，而且在监测到当前有事务时，会抛出异常拒绝执行

- `NESTED`：表示如果当前有事务，则开启一个嵌套级别事务，如果当前没有事务，则开启一个新事务

> 上面这么多种事务的传播级别，其实默认的 `REQUIRED` 已经满足绝大部分需求，`SUPPORTS` 和 `REQUIRES_NEW` 在少数情况下会用到，其他基本不会用到，因为把事务搞得越复杂，不仅逻辑跟着复杂，而且速度也会越慢



定义事务的传播级别也是写在 `@Transactional` 注解里的：

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public Product createProduct() {
    ...
}
```



### Spring是如何传播事务的

Spring 使用声明式事务，最终也是通过执行 JDBC 事务来实现功能的，那么一个事务方法，如何获知当前是否存在事务？

答案是 [使用ThreadLocal](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581251653666)。Spring 总是把 JDBC 相关的 `Connection` 和 `TransactionStatus` 实例绑定到`ThreadLocal`。如果一个事务方法从 `ThreadLocal` 未取到事务，那么它会打开一个新的 JDBC 连接，同时开启一个新的事务，否则，它就直接使用从 `ThreadLocal` 获取的 JDBC 连接以及 `TransactionStatus`。



因此，事务能正确传播的前提是，方法调用是在一个线程内才行。如果像下面这样写：

```java
@Transactional
public User register(String email, String password, String name) { // BEGIN TX-A
    User user = jdbcTemplate.insert("...");
    new Thread(() -> {
        // BEGIN TX-B:
        bonusService.addBonus(user.id, 100);
        // END TX-B
    }).start();
} // END TX-A
```

在另一个线程中调用 `BonusService.addBonus()`，它根本获取不到当前事务，因此，`UserService.register()`和`BonusService.addBonus()`两个方法，将分别开启两个完全独立的事务。



**事务只能在当前线程传播，无法跨线程传播**

那如果我们想实现跨线程传播事务呢？原理很简单，就是要想办法把当前线程绑定到  `ThreadLocal` 的 `Connection` 和`TransactionStatus` 实例传递给新线程，但实现起来非常复杂，根据异常回滚更加复杂，不推荐自己去实现。
