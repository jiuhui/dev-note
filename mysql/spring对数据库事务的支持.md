# spring对数据库事务的支持

Spring中最为常用的事务管理器为org.springframework.jdbc.datasource.DataSourceTransactionManager。Spring基于DataSourceTransactionManager提供了两种方式来支持数据库事务处理，编程式和声明式。

## 编程式

编程式即为，在业务代码中，通过编程的方式，使用DataSourceTransactionManager的setDataSource（设置数据源，关系型数据库实例），doBegin（开始事务），doCommit（提交事务），doRollback（回滚事务），完成一次数据库操作。这种方式的缺点是明显的，就是对于通用的数据库操作（操作行为大致相同，只是数据不同，可抽象成为统一的行为），代码侵入太强，不够优雅。

## 声明式

声明式即为，在操作数据库的bean的函数上，使用@Transactional标注，声明该方法内部将进行数据库事务处理。借助于AOP，容器在加载该bean的时候，对其进行增强，在标注了@Transactional的方法执行前和执行后动态织入数据库事务处理的相关代码，也就是借助Spring容器将编程式事务处理的代码动态织入进去，无需在业务代码中体现，对于不同的配置项（比如回滚条件等）配置在@Transactional上。

声明式事务处理是基于动态proxy实现的所以，如果你标注了@Transactional的方法，无法体现在proxy中（无论是自调用还是非public方法），都不会体现在其proxy中，自然也不会被事务所支持。