---
title: sentinel持久化改造记录
typora-root-url: ../
date: 2022-11-15 15:24:50
tags: sentinel
categories: sentinel
---



## 

## 前言

Sentinel Dashboard管理界面配置的规则默认存储在内存，Sentinel Dashboard一旦重启，配置规则就会消失，不适用于生成环境。

以下对`Sentinel Dashboard`的源码进行进行修改，实现规则持久化至`nacos`中，基于`sentinel 1.8`版本的master分支修改

github仓库地址:`https://github.com/alibaba/Sentinel.git`



## 改造Sentinel-Dashboard



### 修改依赖

修改sentinel-dashboard模块的pom.xml，将依赖`sentinel-datasource-nacos`的test注释掉

```xml
<!-- for Nacos rule publisher sample -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
    <!--<scope>test</scope>-->
</dependency>
```



### 修改nacos相关java代码

找到如下目录

```apl
sentinel-dashboard/src/test/java/com/alibaba/csp/sentinel/dashboard/rule/nacos
```

将整个目录拷贝到

```apl
sentinel-dashboard/src/main/java/com/alibaba/csp/sentinel/dashboard/rule/nacos
```



改造NacosConfig类

1. 添加各规则转换器
2. ConfigService bean修改添加nacos相关连接配置,以便启动时可以通过JVM参数指定对应的配置项

```java
@Configuration
public class NacosConfig {
    // 指定bean name
    @Bean("flowRuleEntityEncoder")
    public Converter<List<FlowRuleEntity>, String> flowRuleEntityEncoder() {
        return JSON::toJSONString;
    }
    // 指定bean name
    @Bean("flowRuleEntityDecoder")
    public Converter<String, List<FlowRuleEntity>> flowRuleEntityDecoder() {
        return s -> JSON.parseArray(s, FlowRuleEntity.class);
    }
    // 
    // 其它的EntityEncoder，EntityDecoder....省略
    // 如：degrade,param-flow,system,authority,gw-flow,gw-api-group
    // 

    // 添加nacos连接配置
    @Bean
    @ConfigurationProperties("spring.cloud.nacos.config")
    public NacosConfigProperties nacosConfigProperties() {
        return new NacosConfigProperties();
    }

    @Bean
    public ConfigService nacosConfigService(NacosConfigProperties configProperties) throws Exception {
        Properties properties = new Properties();
        properties.put(PropertyKeyConst.SERVER_ADDR, configProperties.getServerAddr());
        properties.put(PropertyKeyConst.NAMESPACE, configProperties.getNamespace());
        return ConfigFactory.createConfigService(properties);
    }

    public static class NacosConfigProperties{
        private String serverAddr;
        private String namespace;

        public String getServerAddr() {
            return serverAddr;
        }

        public void setServerAddr(String serverAddr) {
            this.serverAddr = serverAddr;
        }

        public String getNamespace() {
            return namespace;
        }

        public void setNamespace(String namespace) {
            this.namespace = namespace;
        }
    }
}
```



改造`NacosConfigUtil`,添加各种规则在Nacos存储的Data Id后缀

```java
 public final class NacosConfigUtil {

    public static final String GROUP_ID = "SENTINEL_GROUP";
    
    public static final String AUTHORITY_DATA_ID_POSTFIX = "-authority-rules";
    public static final String DEGRADE_DATA_ID_POSTFIX = "-degrade-rules";
    public static final String FLOW_DATA_ID_POSTFIX = "-flow-rules";
    public static final String GW_API_DATA_ID_POSTFIX = "-gw-api-rules";
    public static final String GW_FLOW_DATA_ID_POSTFIX = "-gw-flow-rules";
    public static final String PARAM_FLOW_DATA_ID_POSTFIX = "-param-flow-rules";
    public static final String SYSTEM_DATA_ID_POSTFIX = "-system-rules";
}
```



### 增加NacosProvider和NacosPublisher

参考从test目录拷贝过来`FlowRuleNacosProvider`添加其它类型的`NacosProvider`,如下的`DegradeRuleNacosProvider`，其它`param-flow`,`system`,`authority`,`gw-flow`,`gw-api-group`此处省略

```java
@Component("degradeRuleNacosProvider")
public class DegradeRuleNacosProvider implements DynamicRuleProvider<List<DegradeRuleEntity>> {

    @Autowired
    private ConfigService configService;
    @Autowired
    private Converter<String, List<DegradeRuleEntity>> converter;

    @Override
    public List<DegradeRuleEntity> getRules(String appName) throws Exception {
        String rules = configService.getConfig(appName + NacosConfigUtil.DEGRADE_DATA_ID_POSTFIX, NacosConfigUtil.GROUP_ID, 3000);
        if (StringUtil.isEmpty(rules)) {
            return new ArrayList<>();
        }
        return converter.convert(rules);
    }
}
```

参考从test目录拷贝过来`FlowRuleNacosPublisher`添加其它类型的`NacosPublisher`,如下的`DegradeRuleNacosPublisher`，其它`param-flow`,`system`,`authority`,`gw-flow`,`gw-api-group`此处省略

```java
@Component("degradeRuleNacosPublisher")
public class DegradeRuleNacosPublisher implements DynamicRulePublisher<List<DegradeRuleEntity>> {

    @Autowired
    private ConfigService configService;
    @Autowired
    private Converter<List<DegradeRuleEntity>, String> converter;

    @Override
    public void publish(String app, List<DegradeRuleEntity> rules) throws Exception {
        AssertUtil.notEmpty(app, "app name cannot be empty");
        if (rules == null) {
            return;
        }
        configService.publishConfig(app + NacosConfigUtil.DEGRADE_DATA_ID_POSTFIX, NacosConfigUtil.GROUP_ID, converter.convert(rules));
    }
}
```



### 改造Controller类

修改流控规则`FlowControllerV1`,添加`flow`的`NacosPublisher`和`NacosProvider`，处理增删查改4个接口

```java
@RestController
@RequestMapping(value = "/v1/flow")
public class FlowControllerV1 {
    
    // 这里只给出变动或新增的代码，其它原本的代码省略...
    
    // 注入FlowRuleNacosProvider
    @Autowired
    private FlowRuleNacosProvider flowRuleNacosProvider;
    // 注入FlowRuleNacosPublisher
    @Autowired
    private FlowRuleNacosPublisher flowRuleNacosPublisher;

    // 推送规则到nacos
    private void publishRules(String app) throws Exception {
        List<FlowRuleEntity> rules = repository.findAllByApp(app);
        this.flowRuleNacosPublisher.publish(app, rules);
    }

    
    @GetMapping("/rules")
    @AuthAction(PrivilegeType.READ_RULE)
    public Result<List<FlowRuleEntity>> apiQueryMachineRules(@RequestParam String app,
                                                             @RequestParam String ip,
                                                             @RequestParam Integer port) {
        // 省略....
        try {
            // 注解掉这行
            //List<FlowRuleEntity> rules = sentinelApiClient.fetchFlowRuleOfMachine(app, ip, port);
            // 通过NacosProvider从nacos获取规则
            List<FlowRuleEntity> rules = flowRuleNacosProvider.getRules(app);
            rules = repository.saveAll(rules);
            return Result.ofSuccess(rules);
        } catch (Throwable throwable) {
            // 省略....
        }
    }
    
    @PostMapping("/rule")
    @AuthAction(PrivilegeType.WRITE_RULE)
    public Result<FlowRuleEntity> apiAddFlowRule(@RequestBody FlowRuleEntity entity) {
        // 省略....
        try {
            // 注解掉这行
            //publishRules(entity.getApp(), entity.getIp(), entity.getPort()).get(5000, TimeUnit.MILLISECONDS);
            // 新增时重新同步至nacos
            publishRules(entity.getApp());
            return Result.ofSuccess(entity);
        } catch (Throwable t) {
            // 省略....
        }
    }
    
    @PutMapping("/save.json")
    @AuthAction(PrivilegeType.WRITE_RULE)
    public Result<FlowRuleEntity> apiUpdateFlowRule(Long id, String app,
                                                  String limitApp, String resource, Integer grade,
                                                  Double count, Integer strategy, String refResource,
                                                  Integer controlBehavior, Integer warmUpPeriodSec,
                                                  Integer maxQueueingTimeMs) {
        // 省略....
        try {
            // 注解这行
            //publishRules(entity.getApp(), entity.getIp(), entity.getPort()).get(5000, TimeUnit.MILLISECONDS);
            // 更新时重新同步至nacos
            publishRules(entity.getApp());
            return Result.ofSuccess(entity);
        } catch (Throwable t) {
            // 省略....
        }
    }
    
    @DeleteMapping("/delete.json")
    @AuthAction(PrivilegeType.WRITE_RULE)
    public Result<Long> apiDeleteFlowRule(Long id) {
        // 省略....
        try {
            // 注解这行
            //publishRules(oldEntity.getApp(), oldEntity.getIp(), oldEntity.getPort()).get(5000, TimeUnit.MILLISECONDS);
            // 删除时重新同步至nacos
            publishRules(oldEntity.getApp());
            return Result.ofSuccess(id);
        } catch (Throwable t) {
            // 省略....
        }
    }
}

```

其它`Controller`，如：`degrade`,`param-flow`,`system`,`authority`,`gw-flow`,`gw-api-group`也是一样的改造逻辑，此处省略...



## 业务微服务项目

添加POM依赖

```xml
<dependency>
  <groupId>com.alibaba.csp</groupId>
  <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

添加项目配置

```yaml
spring:
  cloud:
    sentinel:
      transport:
        port: "20051"
        dashboard: 192.168.2.x:8080   # 配置Sentinel dashborad 控制台地址
      datasource:                   #添加Nacos数据源配置
        flow:
          nacos:                          #数据源为NACOS中
            server-addr: ${spring.cloud.nacos.config.server-addr}   #NACOS SERVER URL
            dataId: ${spring.application.name}-flow-rules           
            groupId: SENTINEL_GROUP
            namespace: c8247bcf-16a1-42da-bc98-0f6f4c7b5ecc          
            data-type: json
            rule-type: flow
        authority:
          nacos:                          #数据源为NACOS中
            server-addr: ${spring.cloud.nacos.config.server-addr}   #NACOS SERVER URL
            dataId: ${spring.application.name}-authority-rules
            groupId: DEFAULT_GROUP
            namespace: c8247bcf-16a1-42da-bc98-0f6f4c7b5ecc
            data-type: json
            rule-type: authority
        degrade:
          nacos:                          #数据源为NACOS中
            server-addr: ${spring.cloud.nacos.config.server-addr}   #NACOS SERVER URL
            dataId: ${spring.application.name}-degrade-rules
            groupId: SENTINEL_GROUP
            namespace: 83fe3f08-a12f-4887-9cc9-6ac73320d658
            data-type: json
            rule-type: degrade
        param-flow:
          nacos:                          #数据源为NACOS中
            server-addr: ${spring.cloud.nacos.config.server-addr}   #NACOS SERVER URL
            dataId: ${spring.application.name}-param-flow-rules
            groupId: SENTINEL_GROUP
            namespace: c8247bcf-16a1-42da-bc98-0f6f4c7b5ecc
            data-type: json
            rule-type: param-flow
        system:
          nacos:                          #数据源为NACOS中
            server-addr: ${spring.cloud.nacos.config.server-addr}   #NACOS SERVER URL
            dataId: ${spring.application.name}-system-rules
            groupId: SENTINEL_GROUP
            namespace: c8247bcf-16a1-42da-bc98-0f6f4c7b5ecc
            data-type: json
            rule-type: system
```





