# Spring boot项目接入apollo

## 1、apollo客户端

开发环境地址：http://10.30.2.214:8070/

生产环境、UAT环境地址：http://config-center.hivoice.cn/

##### （1）创建项目

![](..\img\apollo01.png)

创建项目要求应用ID的唯一性，应用Id就是后面在项目中配置的app.id,是用来让配置和项目关联起来的。

##### （2）集群

![](..\img\apollo02.png)

新创建的项目默认有一个default集群，不同集群的配置是不一样的，如果项目只在一个集群部署（北京阿里云）那么就不需要再创建集群了。直接在这个默认的集群上配置。

如果项目部署在多个集群上，比如我们有的项目是部署北京阿里云和上海阿里云两个集群上的，两个集群上的配置是不一样的，就需要再添加一个集群，集群的名字定义为bj或者sh，配置文件中通过配置

```
apollo.cluster=bj指定对应的集群。
```

##### （3）发布

新增或者修改的配置文件通过发布才能生效。

## 2、spring boot 项目接入（参考data-location项目）

##### 1、pom文件增加 Apollo的maven依赖 

```java
<dependency>
   <groupId>com.ctrip.framework.apollo</groupId>
   <artifactId>apollo-client</artifactId>
   <version>1.4.0</version>
</dependency>
```

##### 2、需要在spring boot项目配置apollo服务地址。创建一个`apollo-env.properties`，放在程序的classpath下。配置apollo各个环境地址。

```less
dev.meta=http://10.30.2.214:8080
uat.meta=http://cc-conf-uat.hivoice.cn:8080
pro.meta=http://cc-conf-pro.hivoice.cn:8080
```

##### 3、apollo配置项。创建一个application-apollo.properties配置文件，放在程序的classpath下。用来专门配置apollo的其他配置项。

```less
# 设置在应用启动阶段就加载 Apollo 配置
apollo.bootstrap.enabled=true
# 将 Apollo 配置加载提到初始化日志系统之前，需要托管日志配置时开启
apollo.bootstrap.eagerLoad.enabled =true
# 在apollo配置上配置的 应用id
app.id=biz-cont-data-location
# 指定集群  这个配置对应客户端配置的集群
apollo.cluster=bj
```

##### 4、通过注解开启apollo服务，在spring boot服务的启动类上加  @EnableApolloConfig 注解

##### 5、本地开发

如果一旦连接apollo配置服务就会在本地缓存配置的，这样在无法连接apollo配置服务的时候也能做开发，apollo文档是这么说的

 ![](..\img\apollo03.png)

但是这种方法是不太好的。因为这样如果项目首次从git上拉取下来如果不接入apollo配置中心不能运行。程序的开发最好是不要依赖环境变化。所以我们把本地开发的配置文件创建一个

application-local.properties配置文件，这个文件和application-apollo.properties配置文件是对应的。通过在 application.properties 配置文件中的

```
spring.profiles.active=local
或者
spring.profiles.active=apollo
 
```

这个配置项来指定使用apollo配置中心还是本地配置文件。

配置文件大概有这些

![](..\img\apollo04.png)

##### 6、启动的时候通过环境变量设置启动参数  -Denv=dev -Denv=uat-Denv=pro 分别在开发、uat和生成环境配置。对应的就是指定apollo-env.properties 中配置的三个环境的apollo服务地址。