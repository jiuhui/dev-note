# Spring Cloud Eureka

Eureka Server端采用的是P2P的复制模式，但是它不保证复制操作一定能成功，因此它提供的是一个最终一致性的服务实例视图；Client端在Server端的注册信息有一个带期限的租约，一旦Server端在制定期间没有收到Client端发送的心跳，则Server端会认Client端注册的服务器是不健康的，定时任务会将其从注册表中剔除。
基于什么考虑因素选择Eureka呢，主要有如下几点：
1、选择AP而不是CP
2、如果团队是Java语言体系，则偏好Java语言开发的，技术体系上比较同意，出问题也好排查修复，对组件的掌控力较强，方便扩展维护
3、是Netflix开源套件的一部分，跟其他组件整合比较好 ribbon 、zuul。

## Eureka核心类

### InstanceInfo

Eureka使用InstanceInfo来代表注册的服务实例

```java
@ProvidedBy(EurekaConfigBasedInstanceInfoProvider.class)
@Serializer("com.netflix.discovery.converters.EntityBodyConverter")
@XStreamAlias("instance")
@JsonRootName("instance")
public class InstanceInfo {
    private static final String VERSION_UNKNOWN = "unknown";
    private static final Logger logger = LoggerFactory.getLogger(InstanceInfo.class);
    public static final int DEFAULT_PORT = 7001;
    public static final int DEFAULT_SECURE_PORT = 7002;
    public static final int DEFAULT_COUNTRY_ID = 1;
    // 实例id
    private volatile String instanceId;
    //
    private volatile String appName;
    @Auto
    private volatile String appGroupName;
    private volatile String ipAddr;
    private static final String SID_DEFAULT = "na";
    /** @deprecated */
    @Deprecated
    private volatile String sid;
    private volatile int port;
    private volatile int securePort;
    @Auto
    private volatile String homePageUrl;
    @Auto
    private volatile String statusPageUrl;
    @Auto
    private volatile String healthCheckUrl;
    @Auto
    private volatile String secureHealthCheckUrl;
    @Auto
    private volatile String vipAddress;
    @Auto
    private volatile String secureVipAddress;
    @XStreamOmitField
    private String statusPageRelativeUrl;
    @XStreamOmitField
    private String statusPageExplicitUrl;
    @XStreamOmitField
    private String healthCheckRelativeUrl;
    @XStreamOmitField
    private String healthCheckSecureExplicitUrl;
    @XStreamOmitField
    private String vipAddressUnresolved;
    @XStreamOmitField
    private String secureVipAddressUnresolved;
    @XStreamOmitField
    private String healthCheckExplicitUrl;
    /** @deprecated */
    @Deprecated
    private volatile int countryId;
    private volatile boolean isSecurePortEnabled;
    private volatile boolean isUnsecurePortEnabled;
    private volatile DataCenterInfo dataCenterInfo;
    private volatile String hostName;
    private volatile InstanceInfo.InstanceStatus status;
    private volatile InstanceInfo.InstanceStatus overriddenStatus;
    @XStreamOmitField
    private volatile boolean isInstanceInfoDirty;
    private volatile LeaseInfo leaseInfo;
    @Auto
    private volatile Boolean isCoordinatingDiscoveryServer;
    @XStreamAlias("metadata")
    private volatile Map<String, String> metadata;
    @Auto
    private volatile Long lastUpdatedTimestamp;
    @Auto
    private volatile Long lastDirtyTimestamp;
    @Auto
    private volatile InstanceInfo.ActionType actionType;
    @Auto
    private volatile String asgName;
    private String version;
}
```

