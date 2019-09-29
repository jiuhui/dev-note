# spring对数据库事务的支持

## 1. spring事务管理器

PlatformTransactionManager的源码如下：

```java
package org.springframework.transaction;

import org.springframework.lang.Nullable;

/**这是spring事务基础设施中的中心接口，Application可以直接使用，但是它不是主要作为API，
	通常，application会工作用无论是TransactionTemplate还是声明式事务通过AOP提供的。
 * This is the central interface in Spring's transaction infrastructure.
 * Applications can use this directly, but it is not primarily meant as API:
 * Typically, applications will work with either TransactionTemplate or
 * declarative transaction demarcation through AOP.
 * 对于实现者来说，建议实现AbstractPlatformTransactionManager类，它预先实现定义的传播行为
 	和负责事务同步处理。子类必须为底层事务的特定状态实现模板方法，例如：begin、suspend、resume、commit。
 * <p>For implementors, it is recommended to derive from the provided
 * {@link org.springframework.transaction.support.AbstractPlatformTransactionManager}
 * class, which pre-implements the defined propagation behavior and takes care
 * of transaction synchronization handling. Subclasses have to implement
 * template methods for specific states of the underlying transaction,
 * for example: begin, suspend, resume, commit.
 * 默认的实现有DataSoureceTransactionManager和JtaTransactionManager可以作为参考
 * <p>The default implementations of this strategy interface are
 * {@link org.springframework.transaction.jta.JtaTransactionManager} and
 * {@link org.springframework.jdbc.datasource.DataSourceTransactionManager},
 * which can serve as an implementation guide for other transaction strategies.
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @since 16.05.2003
 * @see org.springframework.transaction.support.TransactionTemplate
 * @see org.springframework.transaction.interceptor.TransactionInterceptor
 * @see org.springframework.transaction.interceptor.TransactionProxyFactoryBean
 */
public interface PlatformTransactionManager {

	/**
	 * Return a currently active transaction or create a new one, according to
	 * the specified propagation behavior.
	 * <p>Note that parameters like isolation level or timeout will only be applied
	 * to new transactions, and thus be ignored when participating in active ones.
	 * <p>Furthermore, not all transaction definition settings will be supported
	 * by every transaction manager: A proper transaction manager implementation
	 * should throw an exception when unsupported settings are encountered.
	 * <p>An exception to the above rule is the read-only flag, which should be
	 * ignored if no explicit read-only mode is supported. Essentially, the
	 * read-only flag is just a hint for potential optimization.
	 * @param definition TransactionDefinition instance (can be {@code null} for defaults),
	 * describing propagation behavior, isolation level, timeout etc.
	 * @return transaction status object representing the new or current transaction
	 * @throws TransactionException in case of lookup, creation, or system errors
	 * @throws IllegalTransactionStateException if the given transaction definition
	 * cannot be executed (for example, if a currently active transaction is in
	 * conflict with the specified propagation behavior)
	 * @see TransactionDefinition#getPropagationBehavior
	 * @see TransactionDefinition#getIsolationLevel
	 * @see TransactionDefinition#getTimeout
	 * @see TransactionDefinition#isReadOnly
	 */
	TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

	/**
	 * Commit the given transaction, with regard to its status. If the transaction
	 * has been marked rollback-only programmatically, perform a rollback.
	 * <p>If the transaction wasn't a new one, omit the commit for proper
	 * participation in the surrounding transaction. If a previous transaction
	 * has been suspended to be able to create a new one, resume the previous
	 * transaction after committing the new one.
	 * <p>Note that when the commit call completes, no matter if normally or
	 * throwing an exception, the transaction must be fully completed and
	 * cleaned up. No rollback call should be expected in such a case.
	 * <p>If this method throws an exception other than a TransactionException,
	 * then some before-commit error caused the commit attempt to fail. For
	 * example, an O/R Mapping tool might have tried to flush changes to the
	 * database right before commit, with the resulting DataAccessException
	 * causing the transaction to fail. The original exception will be
	 * propagated to the caller of this commit method in such a case.
	 * @param status object returned by the {@code getTransaction} method
	 * @throws UnexpectedRollbackException in case of an unexpected rollback
	 * that the transaction coordinator initiated
	 * @throws HeuristicCompletionException in case of a transaction failure
	 * caused by a heuristic decision on the side of the transaction coordinator
	 * @throws TransactionSystemException in case of commit or system errors
	 * (typically caused by fundamental resource failures)
	 * @throws IllegalTransactionStateException if the given transaction
	 * is already completed (that is, committed or rolled back)
	 * @see TransactionStatus#setRollbackOnly
	 */
	void commit(TransactionStatus status) throws TransactionException;

	/**
	 * Perform a rollback of the given transaction.
	 * <p>If the transaction wasn't a new one, just set it rollback-only for proper
	 * participation in the surrounding transaction. If a previous transaction
	 * has been suspended to be able to create a new one, resume the previous
	 * transaction after rolling back the new one.
	 * <p><b>Do not call rollback on a transaction if commit threw an exception.</b>
	 * The transaction will already have been completed and cleaned up when commit
	 * returns, even in case of a commit exception. Consequently, a rollback call
	 * after commit failure will lead to an IllegalTransactionStateException.
	 * @param status object returned by the {@code getTransaction} method
	 * @throws TransactionSystemException in case of rollback or system errors
	 * (typically caused by fundamental resource failures)
	 * @throws IllegalTransactionStateException if the given transaction
	 * is already completed (that is, committed or rolled back)
	 */
	void rollback(TransactionStatus status) throws TransactionException;

}

```

所以，Spring并不直接管理事务，而是提供了多种事务管理器，他们将事务管理委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。通过事务管理器PlatformTransactionManager，Spring为各个平台如JDBC、Hibernate等提供了对应的事务管理器，但是具体的实现依赖各自平台的事务实现。

## 2、spring事务定义及状态描述

  从事务管理器PlatformTransactionManager中可以看出，spring完成事务管理还需2个关键元素：事务定义TransactionDefinition及事务状态TransactionStatus描述。

```java
public interface TransactionDefinition {
    int getPropagationBehavior(); //传播行为，默认PROPAGATION_REQUIRED
    int getIsolationLevel();  //隔离级别，默认数据库默认级别，如mysql为可重复读
    int getTimeout();
    boolean isReadOnly();  //是否只读，查询操作可以设置为true
    String getName();
}
```



Spring中最为常用的事务管理器为org.springframework.jdbc.datasource.DataSourceTransactionManager。Spring基于DataSourceTransactionManager提供了两种方式来支持数据库事务处理，编程式和声明式。

### 2.1 事务传播行为

有两个含有事务的方法A和B，B调用A时，A是启用新事务还是延续B的事务，这便是事务传播行为，Spring提供了7种事务传播行为：

PROPAGATION_REQUIRED：当前方法必须运行在事务中，如果存在当前事务则加入当前事务，否则开启新事务，spring 默认行为。
PROPAGATION_SUPPORTS：当前方法不需要事务上下文，如果存在当前事务则加入当前事务，否则裸奔。
PROPAGATION_MANDATORY：方法必须在事务中运行，当前事务不存在则报错。
PROPAGATION_REQUIRES_NEW：方法运行在自己的新事务中。
PROPAGATION_NOT_SUPPORTED
PROPAGATION_NEVER
PROPAGATION_NESTED：已存在当前事务则运行在嵌套事务中，否则开启新事务，涉及savepoint。

### 2.2 事务状态描述

```java
public interface TransactionStatus extends SavepointManager {
    boolean isNewTransaction();  //是否新事务
    boolean hasSavepoint();  //是否有恢复点
    void setRollbackOnly();
    boolean isRollbackOnly();
    void flush(); //Flush the underlying session to the datastore, if applicable: for example, all affected Hibernate/JPA sessions.
    boolean isCompleted();
}
```

这个接口描述的是一些处理事务提供简单的控制事务执行和查询事务状态的方法，事务管理器在回滚或提交时，需要对应的TransactionStatus。

## 3、Spring事务的实现方式

### 编程式

编程式即为，在业务代码中，通过编程的方式，使用DataSourceTransactionManager的setDataSource（设置数据源，关系型数据库实例），doBegin（开始事务），doCommit（提交事务），doRollback（回滚事务），完成一次数据库操作。这种方式的缺点是明显的，就是对于通用的数据库操作（操作行为大致相同，只是数据不同，可抽象成为统一的行为），代码侵入太强，不够优雅。

数据库：

```sql
CREATE TABLE `jdbc_student` (
  `id` int(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=17 DEFAULT CHARSET=utf8
```

测试类：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
@WebAppConfiguration
@Slf4j
public class SpringTransactionTest {
    @Autowired
    @Qualifier(value = "shopJdbcTemplate")
    private JdbcTemplate jdbcTemplate;

    // 使用 TransactionTemplate实现编程式事务
    @Test
    public void testTransactionTemplate(){
        DataSource dataSource = jdbcTemplate.getDataSource();
        PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
        TransactionTemplate transactionTemplate = new TransactionTemplate(transactionManager);
        transactionTemplate.execute(new TransactionCallback() {
            @Override
            public Object doInTransaction(TransactionStatus status) {
                Map student = getStudent(1);
                student.forEach( (k,v) ->{
                    System.out.println(k+","+v);
                });
                int update = update(1, "user122-tx");
                //throwException();
                insert(4,20);
                return null;
            }
        });
    }
    // 使用 PlatformTransactionManager  transactionDefinition 实现编程式事务
    @Test
    public void testTransacetion1(){
        DataSource dataSource = jdbcTemplate.getDataSource();
        PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
        TransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
        //int insert = insert(1, 10);
        // 记录状态点 类似jdbc的savePoint
        TransactionStatus status = transactionManager.getTransaction(transactionDefinition);
        System.out.println("start transaction");
        // 先插入一条数据
        try {
            Map student = getStudent(1);
            student.forEach( (k,v) ->{
                System.out.println(k+","+v);
            });
            int update = update(1, "user14-tx");
            //throwException();
            this.insert(2,20);
            transactionManager.commit(status);
        } catch (Exception e) {
            e.printStackTrace();
            transactionManager.rollback(status);
        }

    }

    private void throwException(){
        throw new RuntimeException("执行失败");
    }

    @SuppressWarnings("rawtypes")
    public Map getStudent(int id) {
        String sql = "select * from jdbc_student where id = ?";
        Map map = jdbcTemplate.queryForMap(sql, id);
        System.out.println("student.id=" + id + ": " + map);
        return map;
    }

    public int update(int id, String name){
        String sql = "update jdbc_student set name = ? where id = ?";
        int count = jdbcTemplate.update(sql, name, id);
        System.out.println("sql: " + sql + ", para: " + name + ", " + id);
        System.out.println("Affected rows : " + count);
        return count;
    }

    public int insert(int id, int age){
        String name = "user" + id;
        String sql = "insert into jdbc_student(id, name, age) values (?, ?, ?)";
        int count = jdbcTemplate.update(sql, id, name, age);
        System.out.println("sql: " + sql);
        System.out.println(String.format("para: %d, %s, %d", id, name, age));
        System.out.println("Affected rows : " + count);
        return count;
    }
}
```

## 声明式

 声明式即为，在操作数据库的bean的函数上，使用@Transactional标注，声明该方法内部将进行数据库事务处理。借助于AOP，容器在加载该bean的时候，对其进行增强，在标注了@Transactional的方法执行前和执行后动态织入数据库事务处理的相关代码，也就是借助Spring容器将编程式事务处理的代码动态织入进去，无需在业务代码中体现，对于不同的配置项（比如回滚条件等）配置在@Transactional上。

声明式事务处理是基于动态proxy实现的所以，如果你标注了@Transactional的方法，无法体现在proxy中（无论是自调用还是非public方法），都不会体现在其proxy中，自然也不会被事务所支持。

### xml配置方式声明事务

xml配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.2.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd    
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">

   <context:annotation-config/>

   <!-- Initialization for data source -->
   <bean id="dataSource" 
      class="org.springframework.jdbc.datasource.DriverManagerDataSource">
      <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://ip:端口/数据库?useUnicode=true&amp;characterEncoding=utf-8"/>
      <property name="username" value="用户" />
      <property name="password" value="密码" />
   </bean>

   <bean id="transactionManager" 
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <property name="dataSource"  ref="dataSource" />    
   </bean>

   <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">  
      <property name="dataSource" ref="dataSource"></property> 
   </bean>

   <bean id="transactionTemplate"
        class="org.springframework.transaction.support.TransactionTemplate">
        <property name="transactionManager">
            <ref local="transactionManager" />
        </property>
   </bean>   

   <bean id="tmTxDao" class="com.marcus.spring.tx.TmTxDao" />

    <!--配置事务增强 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="save*" read-only="false"/>
            <tx:method name="get*" read-only="true"/>
        </tx:attributes>
    </tx:advice>

    <!--aop配置，拦截哪些方法 -->
    <aop:config>
        <aop:pointcut id="pt" expression="execution(* com.marcus.spring.tx.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt"/>
    </aop:config>
</beans>
```

### xml配置注解声明事务

xml配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.2.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">

   <context:annotation-config/>

   <!-- Initialization for data source -->
   <bean id="dataSource" 
      class="org.springframework.jdbc.datasource.DriverManagerDataSource">
      <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://ip:端口/数据库名?useUnicode=true&amp;characterEncoding=utf-8"/>
      <property name="username" value="用户" />
      <property name="password" value="密码" />
   </bean>

   <bean id="transactionManager" 
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <property name="dataSource"  ref="dataSource" />    
   </bean>

   <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">  
      <property name="dataSource" ref="dataSource"></property> 
   </bean>

   <bean id="transactionTemplate"
        class="org.springframework.transaction.support.TransactionTemplate">
        <property name="transactionManager">
            <ref local="transactionManager" />
        </property>
   </bean>   

   <bean id="tmTxDao" class="com.marcus.spring.tx.TmTxDao" />

    <!--开启注解扫描-->
    <context:component-scan base-package="com.marcus.spring.tx"/>

    <!--注解方式实现事务-->
    <tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

然后在service需要事务的方法上加@Transactional注解就行

@Transactional使用注意。下面是阿里巴巴代码规范插件的

![](..\img\transaction1.png)

![](..\img\transaction2.png)