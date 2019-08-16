# Spring boot项目引入mongo多数据源配置

<!--不仅仅是一个教程，我觉得是一种思维的方式-->

首先Spring boot引入mongo单数据源很好做，引入 spring-boot-starter-data-mongodb然后配置文件增加spring.data.mongodb.uri=xxx配置就行了。在调用的时候注入mongoTemplate这类就可以调用了。那么我们思考可以得出多数据源就是生成多个mongoTemple类，然后我们在需要哪一个的时候就注入哪一个。一般都能分析到这步，但是接下来怎么做就无处下手了，然后就网上找各种教程了。下面我以我实际配置的例子和我一步一步的思考，首先找到mongoTemple的源码，看他是怎么实现的，我们跟着源码实现自己的。



MongoTemplate源码

```java
public MongoTemplate(MongoClient mongoClient, String databaseName) {
        this((MongoDbFactory)(new SimpleMongoDbFactory(mongoClient, databaseName)), (MongoConverter)null);
    }

    public MongoTemplate(MongoDbFactory mongoDbFactory) {
        this((MongoDbFactory)mongoDbFactory, (MongoConverter)null);
    }

    public MongoTemplate(MongoDbFactory mongoDbFactory, @Nullable MongoConverter mongoConverter) {
        this.writeConcernResolver = DefaultWriteConcernResolver.INSTANCE;
        this.writeResultChecking = WriteResultChecking.NONE;
        Assert.notNull(mongoDbFactory, "MongoDbFactory must not be null!");
        this.mongoDbFactory = mongoDbFactory;
        this.exceptionTranslator = mongoDbFactory.getExceptionTranslator();
        this.mongoConverter = mongoConverter == null ? getDefaultMongoConverter(mongoDbFactory) : mongoConverter;
        this.queryMapper = new QueryMapper(this.mongoConverter);
        this.updateMapper = new UpdateMapper(this.mongoConverter);
        this.projectionFactory = new SpelAwareProxyProjectionFactory();
        this.mappingContext = this.mongoConverter.getMappingContext();
        if (this.mappingContext instanceof MongoMappingContext) {
            this.indexCreator = new MongoPersistentEntityIndexCreator((MongoMappingContext)this.mappingContext, this);
            this.eventPublisher = new MongoMappingEventPublisher(this.indexCreator);
            if (this.mappingContext instanceof ApplicationEventPublisherAware) {
                ((ApplicationEventPublisherAware)this.mappingContext).setApplicationEventPublisher(this.eventPublisher);
            }
        }

    }
```



SimpleMongoDbFactory源码

```java
public class SimpleMongoDbFactory implements DisposableBean, MongoDbFactory {
    private final MongoClient mongoClient;
    private final String databaseName;
    private final boolean mongoInstanceCreated;
    private final PersistenceExceptionTranslator exceptionTranslator;
    @Nullable
    private WriteConcern writeConcern;

    public SimpleMongoDbFactory(MongoClientURI uri) {
        this(new MongoClient(uri), uri.getDatabase(), true);
    }

    public SimpleMongoDbFactory(MongoClient mongoClient, String databaseName) {
        this(mongoClient, databaseName, false);
    }

    private SimpleMongoDbFactory(MongoClient mongoClient, String databaseName, boolean mongoInstanceCreated) {
        Assert.notNull(mongoClient, "MongoClient must not be null!");
        Assert.hasText(databaseName, "Database name must not be empty!");
        Assert.isTrue(databaseName.matches("[^/\\\\.$\"\\s]+"), "Database name must not contain slashes, dots, spaces, quotes, or dollar signs!");
        this.mongoClient = mongoClient;
        this.databaseName = databaseName;
        this.mongoInstanceCreated = mongoInstanceCreated;
        this.exceptionTranslator = new MongoExceptionTranslator();
    }

// 进入到MongoClient(uri)这个方法里面
    public MongoClient(MongoClientURI uri) {
        super(uri);
    }
// 再看一下mongoClientURI的构造方法
    
    public MongoClientURI(String uri) {
        this(uri, new Builder());
    }
```



```java
/**
 * @author: fanjiuhui
 * @Date: 2019/8/13
 * @Description:
 */
@Configuration
@ConfigurationProperties(prefix = "spring.data.mongodb.shop")
public class ShopMongoConfig {
    private String uri;

    @Bean(name = "shopMongoTemplate")
    public MongoTemplate shopMongoTemplate(){
        return new MongoTemplate(new SimpleMongoDbFactory( new MongoClientURI(uri)));

    }
}

```

```java
/**
 * @author: fanjiuhui
 * @Date: 2019/8/13
 * @Description:
 */
@Configuration
@ConfigurationProperties(prefix = "spring.data.mongodb.center")
public class UskStatisticCenterMongoConfig {
    private String uri;

    @Bean(name = "centerMongoTemplate")
    public MongoTemplate shopMongoTemplate(){
        return new MongoTemplate(new SimpleMongoDbFactory( new MongoClientURI(uri)));
    }
}

```

把这两个方法公共的部分抽象一下

```java
/**
 * @author: fanjiuhui
 * @Date: 2019/8/13
 * @Description:
 */
public abstract class AbstractMongoConfig {
    protected String uri;

    protected MongoDbFactory mongoDbFactory(){
        return  new SimpleMongoDbFactory( new MongoClientURI(uri));
    }

    abstract public MongoTemplate getMongoTemplate();
}

```

```java
/**
 * @author: fanjiuhui
 * @Date: 2019/8/13
 * @Description:
 */
@Configuration
@ConfigurationProperties(prefix = "spring.data.mongodb.shop")
public class ShopMongoConfig extends AbstractMongoConfig{

    @Override
    @Bean(name = "shopMongoTemplate")
    public MongoTemplate getMongoTemplate() {
        return new MongoTemplate(mongoDbFactory());
    }
}
```

```java
/**
 * @author: fanjiuhui
 * @Date: 2019/8/13
 * @Description:
 */
@Configuration
@ConfigurationProperties(prefix = "spring.data.mongodb.center")
public class UskStatisticCenterMongoConfig extends AbstractMongoConfig{


    public MongoTemplate shopMongoTemplate(){
        return new MongoTemplate(new SimpleMongoDbFactory( new MongoClientURI(uri)));
    }

    @Override
    @Bean(name = "centerMongoTemplate")
    public MongoTemplate getMongoTemplate() {
        return new MongoTemplate(mongoDbFactory());
    }
}
```

