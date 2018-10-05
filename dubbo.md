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




















































