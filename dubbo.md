1。什么是分布式架构
  分布式就是多个计算机的集合，这写计算机对于用户来说就想单个相关系统

  正如淘宝，对于用户来说就是一个服务，但是后条有上百个服务进行支持，在后台的这些服务互相支持调用
  随着互联网的发展，网站的应用的规模不断扩大，常规的垂直硬用架构已经无法应对，分布式服务架构以及流动计算架构势在必行
  需要一个智力系统确保架构有条不紊的演进
2.restful api 
  1.每一个url代表一种资源。或资源的资质
  2.客户端与服务端之间，传递这种资源的某种表现层
  3.客户端的的四个动词，对服务端的资源的操作
3.RPC
  远程调用，A服务和B服务之间不在同一台服务，A服务想调用B服务，A服务先到A服务的代理，A服务和B服务建立连接socket,A服务的参数传递诶B服务的socket,在传递到B服务的带来把参数以及地址传递给A服务，B服务响应返回给B代理，在传递给B的SOCKET,影响给B  
4.dubbo覆盖顺序
 VM启动参数>application.properties>dubbo.properties
5.配置覆盖关系
	以 timeout 为例，显示了配置的查找顺序，其它 retries, loadbalance, actives 等类似：
	方法级优先，接口级次之，全局配置再次之。
	如果级别一样，则消费方优先，提供方次之。 
	参数说明
	<!--timeout 默认1000-->
	<!--retries充实次数，不保扩第一次-->
	<!--幂等 不管执行多少次结果都一样 删除 查询 修改  非幂等 影响结果的操作 新增-->

6灰度发布多版本
  version
7.三种配置方式
 1.使用dubbo-start 暴露 service  引入 Reference
 2.使用provider.xml/@ImportResource(locations = "classpath:provider.xml")
 3.使用注解
   @Configuration
public class DubboConfiguration {

    @Bean
    public ApplicationConfig applicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("boot");
        return applicationConfig;
    }

    @Bean
    public RegistryConfig registryConfig() {
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setAddress("zookeeper://127.0.0.1:2181");
//        registryConfig.setClient("curator");
        return registryConfig;
    }

    @Bean
    public ProtocolConfig protocolConfig() {
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setName("dubbo");
        protocolConfig.setPort(20882);
        return protocolConfig;
    }

    @Bean
    public ServiceConfig<UserService> serviceServiceConfig(UserService userService) {
        ServiceConfig<UserService> serviceServiceConfig = new ServiceConfig();
        serviceServiceConfig.setInterface(UserService.class);
        serviceServiceConfig.setRef(userService);

        return serviceServiceConfig;
    }
}
8.dubbo高可用
   现象 zookeeper宕机 还是可以消费dubbo暴露服务
   原因
    监控中心当掉不影响使用，知识丢失部分采样数据
    数据库宕掉后，注册中心扔能通过缓存提供服务列表查询，但不能注册新的服务
    注册中心集群，一台宕机，自动切换另外一台
    注册中心全部宕机，服务提供者和消费者仍能通过本地缓存通讯
    服务提供者无状态，任意一台宕掉后，不用想使用
    服务提供者全部宕机，服务消费者不可用，并无限次重新等待服务提供者回复

    高可用，通过设计，较少系统不能提供服务时间

9.dubbo负载均衡
  Random loadBalance
  基于权重的随机负载均衡机制
  RoundRobin loadblance
  基于权重的伦旭堵在均衡
  leastACtive loadbalance
  最少活跃数负载均衡机制
  总是条响应快的模块进行调用
  consistenthash loadblance
  一致性hash，负载均衡
  方法名参数一直，都会去同一台服务
10.权重设置
  权重可以在管理平台直接修改，也可以直接那一台服务的权重编辑
11.服务降级
 什么是服务降级
  当服务压力剧增的的情况下，根据实际业务情况及流量，对一些服务和页面
  有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证和弦交易的正常运作或高效运作


  可以通过服务降级功能临时屏蔽某个出错的非关键服务，并定义降级后的返回策略  

  1.不发起远程调用返回为空
  2.当服务失败的时候返回为空

 12，dubbo结合Hystrix
   添加依赖
   <dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
		<version>2.0.1.RELEASE</version>
	</dependency>
    提供者方法注解
     1.HystrixCommand 方法配置
     2.EnableHystrix
     消费者
     1.HystrixCommand 方法配置，添加fallbackMethod = "hello"
     2.EnableHystrix
    大坑
     java.lang.NoSuchMethodError: org.springframework.boot.builder.SpringApplicationBuilder.<init>([Ljava/lang/Object;)V
     原因版本不兼容最好都是最新版本
 13集群容错
  Failover Cluster(默认)
  失败自动切换，当出现失败，重试其他服务器，通常用于读操作，但重试会带来更长延迟，统统过
  retries=2来设置重试次数，不包换第一次
  重试次数配置如下：
	<dubbo:service retries="2" />
	或
	<dubbo:reference retries="2" />
	或
	<dubbo:reference>
	<dubbo:method name="findFoo" retries="2" />
	</dubbo:reference>
  Faillfast Cluster
   快速失败，只发起一次调用，失败立即报错，通常使用非幂等的写操作。比如新增记录

  Failback Cluster
  失败自动恢复，后台记录失败请求，定时重发，通常用于消息通知操作

  Forking Cluster
  并行调用多个服务器，只要一个成功即返回，通常用于实施先较高的读操作
  但是需要浪费资源，通过forks=2设置最大并行数
  
  Broadcast Cluster
  广播调用所以提供者，逐个调用，任意一台失败，就失败通常使用与多有提供者更新
  缓存或日志等本地资源信息
  集群模式配置
	按照以下示例在服务提供方和消费方配置集群模式
	<dubbo:service cluster="failsafe" />
	或
	<dubbo:reference cluster="failsafe" />
 14.RPC原理
  1.服务消费者调用以本地调用方式调用服务
  2.client stud 接受到调用后负责方法，参数等组装成能够进行网络消息体
  3.client stud 找到服务地址，并将消息发送到服务端
  4.server stud 接受到消息后进行解码
  5.server stud 根据解码结果调用本地服务
  6.本地服务之星并将结果返回给server stud
  7.server stud 将返回结果打包成次奥西发送至消费方
  8.client stud 接受到消息，并进行解码
  9，消费方的到最终结果
 15.netty通讯原理
  是一个异步时间驱动的网络应用程序框架。用于快速可维护的高性能协议服务器和
  客户端，他极大简化了TCP和udp套件字服务网络编程
 16.dubbo原理
  config   



































































