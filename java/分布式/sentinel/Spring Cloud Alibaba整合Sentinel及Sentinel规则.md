# Spring Cloud Alibaba整合Sentinel及Sentinel规则

## 1.部署Sentinel服务（docker版本）

可以看之前的文章

http://localhost:8080进入控制台页面（账号密码默认sentinel）

![login](Spring%20Cloud%20Alibaba%E6%95%B4%E5%90%88Sentinel%E5%8F%8ASentinel%E8%A7%84%E5%88%99.assets/1070077-20191022155749479-1684457228.png)

## 2.整合Sentinel

在dependencies中添加依赖，即可整合Sentinel

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

添加配置文件：

```
spring:
  cloud:
    sentinel:
      transport:
        # 指定sentinel控制台地址
        dashboard: localhost:8080
```

**PS：**其他的配置项

```
spring:
  cloud:
    sentinel:
      transport:
        # 指定sentinel控制台地址
        dashboard: localhost:8080
        # 指定和控制台通信的IP，若不配置，会自动选择一个IP注册
        client-ip: 127.0.0.1
        # 指定和控制台通信的端口哦，默认值8719
        # 若不配置，会自动扫猫从8719开始扫猫，依次+1，知道值找到未被占用的端口
        port: 8719
        # 心跳发送周期，默认值null
        # 但在SimpleHttpHeartbeatSender会用默认值10秒
        heartbeat-interval-ms: 10000
```

## 3.sentinel控制台配置规则

### 3.1 流控规则

打开Sentinel控制台，点击**簇点链路**，可以看见微服务曾经被访问过的路径

![](Spring%20Cloud%20Alibaba%E6%95%B4%E5%90%88Sentinel%E5%8F%8ASentinel%E8%A7%84%E5%88%99.assets/1070077-20191022161008278-1677485641.png)

点击**流控**按钮，便可以为应用设置流控规则

![](Spring%20Cloud%20Alibaba%E6%95%B4%E5%90%88Sentinel%E5%8F%8ASentinel%E8%A7%84%E5%88%99.assets/1070077-20191022161112145-876235606.png)

- 资源名：唯一名称，默认请求路径
- 针对来源：Sentinel可以针对调用者进行限流，填写微服务名，默认default（不区分来源）
- 阈值类型/单机阈值：
  - QPS（每秒钟的请求数量）：当调用该api的QPS达到阈值的时候，进行限流
  - 线程数：当调用该api的线程数达到阈值的时候，进行限流
- 是否集群：不需要集群，暂不研究
- 流控模式：
  - 直接：api达到限流条件时，直接限流
  - 关联：当关联的资源达到阈值时，就限流自己
  - 链路：只记录指定链路上的流量（指定资源从入口资源进来的流量，如果达到阈值，就进行限流）【api级别的针对来源】
- 流控效果：
  - 快速失败：直接失败，抛异常
  - Warm Up：根据codeFactor（冷加载因子，默认3）的值，从阈值/codeFactor，经过预热时长，才达到设置的QPS阈值
  - 排队等待：匀速排队，让请求以匀速的速度通过，阈值类型必须设置为QPS，否则无效

### 3.2 降级规则（断路器模式)

点击**降级**按钮，便可以为应用设置降级规则


**![](Spring%20Cloud%20Alibaba%E6%95%B4%E5%90%88Sentinel%E5%8F%8ASentinel%E8%A7%84%E5%88%99.assets/1070077-20191022164414130-978608979.png)

降级策略:

- RT：平均响应时间（秒级统计）超出阈值且在时间窗口内的请求 >= 5时，触发降级；时间窗口结束后，关闭降级【Sentinel默认最大的RT为4900ms，可以通过-Dcsp.sentinel.statistic.max.rt=xxx修改】
- 异常比例：QPS >= 5 且异常比例（秒级统计）超过阈值时，触发降级；时间窗口结束后，关闭降级
- 异常数：异常数（分钟统计）超过阈值时，触发降级；时间窗口结束后，关闭降级【时间窗口 < 60秒可能会出现问题】

### 3.3 热点规则（热点参数限流规则）

Sentinel默认显示的端点并不支持热点规则，要显示热点规则，需要自己添加代码：

```
@GetMapping("test")
@SentinelResource("test")
public String testHot(@RequestParam(required = false) String a,
                      @RequestParam(required = false) String b) {
    return a + "-" + b;
}
```

点击**热点**按钮，便可以为test设置热点规则
[![img](Spring%20Cloud%20Alibaba%E6%95%B4%E5%90%88Sentinel%E5%8F%8ASentinel%E8%A7%84%E5%88%99.assets/1070077-20191022170532556-1152698646.png)]

**在时间窗口以内，一旦该api指定索引的参数QPS达到了域名，就会触发限流**

- 参数索引：从0开始，上面的代码中：a的参数索引为0；b的参数索引为1【参数索引对应的参数必须时基本类型或者String】

### 3.4 系统规则

**阈值类型**

- LOAD（仅对 Linux/Unix-like 机器生效）：当系统 load1 超过阈值，且系统当前的并发线程数超过系统容量时才会触发系统保护。系统容量由系统的 maxQps * minRt 计算得出。设定参考值一般是 CPU cores * 2.5
- RT：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒
- 线程数：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护
- 入口 QPS：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护
- CPU 使用率：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0）

### 3.5 授权规则

点击**授权**按钮，便可以为应用设置授权规则
![](Spring%20Cloud%20Alibaba%E6%95%B4%E5%90%88Sentinel%E5%8F%8ASentinel%E8%A7%84%E5%88%99.assets/1070077-20191022170532556-1152698646.png)

资源名所代表的资源只允许*流控应用*中添加的微服务使用（白名单）、不允许使用（黑名单）

### 3.6 代码配置规则

#### 流控规则

**参数**

|      Field      |                     说明                     |           默认值            |
| :-------------: | :------------------------------------------: | :-------------------------: |
|    resource     |      资源名，资源名是限流规则的作用对象      |                             |
|      count      |                   限流阈值                   |                             |
|      grade      |        限流阈值类型，QPS 或线程数模式        |           QPS模式           |
|    limitApp     |              流控针对的调用来源              | default，代表不区分调用来源 |
|    strategy     |         default，代表不区分调用来源          |        根据资源本身         |
| controlBehavior | 流控效果（直接拒绝 / 排队等待 / 慢启动模式） |          直接拒绝           |

**代码**

```
private void initFlowQpsRule() {
    List<FlowRule> rules = new ArrayList<>();
    FlowRule rule = new FlowRule(resourceName);
    // 设置QPS阈值为20
    rule.setCount(20);
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    rule.setLimitApp("default");
    rules.add(rule);
    FlowRuleManager.loadRules(rules);
}
```

#### 降级规则

**参数**

|   Field    |                    说明                    | 默认值 |
| :--------: | :----------------------------------------: | :----: |
|  resource  |        资源名，即限流规则的作用对象        |        |
|   count    |                    阈值                    |        |
|   grade    | 降级模式，根据 RT 降级还是根据异常比例降级 |   RT   |
| timeWindow |            降级的时间，单位为 s            |        |

**代码**

```
private void initDegradeRule() {
    List<DegradeRule> rules = new ArrayList<>();
    DegradeRule rule = new DegradeRule();
    rule.setResource(KEY);
    // set threshold RT, 10 ms
    rule.setCount(10);
    rule.setGrade(RuleConstant.DEGRADE_GRADE_RT);
    rule.setTimeWindow(10);
    rules.add(rule);
    DegradeRuleManager.loadRules(rules);
}
```

#### 热点规则

**参数**

|       Field       |                             说明                             |  默认值  |
| :---------------: | :----------------------------------------------------------: | :------: |
|     resource      |                         资源名，必填                         |          |
|       count       |                        限流阈值，必填                        |          |
|       grade       |                           限流模式                           | QPS 模式 |
|   durationInSec   |                 统计窗口时间长度（单位为秒）                 |    1s    |
|  controlBehavior  |            流控效果（支持快速失败和匀速排队模式）            | 快速失败 |
| maxQueueingTimeMs |           最大排队等待时长（仅在匀速排队模式生效）           |   0ms    |
|     paramIdx      | 热点参数的索引，必填，对应 SphU.entry(xxx, args) 中的参数索引位置 |          |
| paramFlowItemList | 参数例外项，可以针对指定的参数值单独设置限流阈值，不受前面 count 阈值的限制。仅支持基本类型 |          |
|    clusterMode    |                    是否是集群参数流控规则                    |  false   |
|   clusterConfig   |                       集群流控相关配置                       |          |

**代码**

```
ParamFlowRule rule = new ParamFlowRule(resourceName)
    .setParamIdx(0)
    .setCount(5);
// 针对 int 类型的参数 PARAM_B，单独设置限流 QPS 阈值为 10，而不是全局的阈值 5.
ParamFlowItem item = new ParamFlowItem().setObject(String.valueOf(PARAM_B))
    .setClassType(int.class.getName())
    .setCount(10);
rule.setParamFlowItemList(Collections.singletonList(item));

ParamFlowRuleManager.loadRules(Collections.singletonList(rule));
```

#### 系统规则

**参数**

|       Field       |            说明            |   默认值   |
| :---------------: | :------------------------: | :--------: |
| highestSystemLoad |        最大的 load1        | -1(不生效) |
|       avgRt       | 所有入口流量的平均响应时间 | -1(不生效) |
|     maxThread     |    入口流量的最大并发数    | -1(不生效) |
|        qpa        |     所有入口资源的 QPS     | -1(不生效) |

**代码**

```
private void initSystemRule() {
    List<SystemRule> rules = new ArrayList<>();
    SystemRule rule = new SystemRule();
    rule.setHighestSystemLoad(10);
    rules.add(rule);
    SystemRuleManager.loadRules(rules);
}
```

#### 授权规则

**参数**

|  Field   |                             说明                             |           默认值            |
| :------: | :----------------------------------------------------------: | :-------------------------: |
| resource |                 资源名，即限流规则的作用对象                 |                             |
| limitApp |   对应的黑名单/白名单，不同 origin 用 , 分隔，如 appA,appB   | default，代表不区分调用来源 |
| strategy | 限制模式，AUTHORITY_WHITE 为白名单模式，AUTHORITY_BLACK 为黑名单模式，默认为白名单模式 |       AUTHORITY_WHITE       |

**代码**

```
AuthorityRule rule = new AuthorityRule();
rule.setResource("test");
rule.setStrategy(RuleConstant.AUTHORITY_WHITE);
rule.setLimitApp("appA,appB");
AuthorityRuleManager.loadRules(Collections.singletonList(rule));
```

```
docker run -d --restart always  --name  seata1.3.0 -p 8091:8091  -v /seata/docker/seata-server:/seata-server -e SEATA_IP=192.168.66.5 -e SEATA_PORT=8091 --privileged=true seataio/seata-server:1.3.0 
```

```
docker run -d --restart always  --name  seata1.3.0 -p 8091:8091  -v /seata/docker/seata-server:/seata-server -e SEATA_IP=172.168.1.35 -e SEATA_PORT=8091 seataio/seata-server:1.3.0 
```