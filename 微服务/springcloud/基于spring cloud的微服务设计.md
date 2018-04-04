Spring Cloud构建微服务
==============


# 一、单体架构与微服务架构
## 1、单体架构服务
### 简介：
* 一个项目包含了应用所有功能，称之为单体应用。架构单体应用的架构风格，我们称之为单体架构，这是目前很多公司也是比较传统的架构风格

### 弊端：
* 复杂性逐渐变高： 系统岁功能的增加，愈发的臃肿复杂，越发复杂。
* 部署速度逐渐变慢： 随着系统代码量的增加，部署时长逐渐增加。
* 阻碍技术的创新：单体架构的开发框架，语言已经定性。如果需要更换技术框架，开发语言。改造十分困哪
* 伸缩性差：服务器伸缩性差，如果因为服务器中某块功能的承载达不到要求，就需要对整套服务器进行扩容。但实际上系统某一些模块儿的性能要求是极低的


## 2、微服务架服务

### 简介：
* 微服务架构是以开发一组小型服务的方式来开发一个独立的应用系统的。其中每个小型服务都运行在自己的进程中，并采用轻量的机制来相互通信，如HTTP，API。这些服务围绕业务功能进行构建。这些微服务可以使用不同的语言来编写，并且可以使用不同的数据存储技术。对这些微服务我们仅做最低限度的集中管理。

### 优点：
* 单一模块复杂性较低： 每个服务实现自己微小的功能，复杂度低
* 可以单服务进行部署：每个服务单独部署
* 各个模块技术，框架实现可以具有独立性：每个服务实现技术，框架可以自行决定
* 按需伸缩服务器：服务器可以根据单个服务进行拓展，如某个自服务存不足，可以只增加该服务器的内存，无需对整个大系统架构进行拓展

### 挑战：
* 运维要求较高: 微服务部署成功后，需要维护管理多个服务，对运维是一种挑战
* 分布式的复杂性： 微服务的核心任然是基于分布式开发，具有一定的复杂性
* 分布式的复杂性： 接口调整成本太高，如一个用户微服务，向商品微服务提供接口，一但提供接口产生变化，商品微服务需要调整
* 重复劳动：部分entity，或者代码可能存在各个微服务均存在的情况，特别是当开发语言不同时，无法形成工具类，同样的工作可能会在各个微服务重复

### 微服务设计原则：
* 单一职责原则：每个微服务具有自己独立的职责，各个微服务之间职责不应具有共性，不应职责混淆
* 服务自治原则：每个微服务具有自己独立治理，单独存在部署的能力
* 轻量级通讯原则：微服务间通讯应以轻量级通讯原则，如http,资源api等。采用scop等协议是不可取的
* 接口明确原则：微服务器的接口智能明确，参数明确。避免，减少接口改动。

### 微服务开发框架：
* Spring Cloud： http://projects.spring.io/spring-cloud
* Dubbo：http://dubbo.io
* Dropwizard：http://www.dropwizard.io

# 三、服务发现的可行方案以及实践案例
* 服务发现可行性方案参考博客网址：http://blog.daocloud.io/microservices-4/
## 1、为什么使用服务发现：
* 假设我们写的代码会调用 REST API 或者 Thrift API 的服务。为了完成一次请求，代码需要知道服务实例的网络位置（IP 地址和端口）。运行在物理硬件上的传统应用中，服务实例的网络位置是相对固定的；代码能从一个偶尔更新的配置文件中读取网络位置，如图：

![](./images/2017/12/1.png)

如果服务采用多服务部署，云化部署，下图，我们客户端如何判定访问具体哪台服务器呢？
![](./images/2017/12/2.png)


* 当服务实例的网络位置都是动态分配的。由于扩展、失败和升级，服务实例会经常动态改变，因此，客户端代码需要使用更加复杂的服务发现机制。
* 服务发现有两大模式：客户端发现模式和服务端发现模式。

## 2、客户端发现模式
* 使用客户端发现模式时，客户端决定相应服务实例的网络位置，并且对请求实现负载均衡。客户端查询服务注册表，后者是一个可用服务实例的数据库；然后使用负载均衡算法从中选择一个实例，并发出请求。
![](./images/2017/12/3.png)

* 客户端发现模式优缺点兼有。这一模式相对直接，除了服务注册外，其它部分无需变动。此外，由于客户端知晓可用的服务实例，能针对特定应用实现智能负载均衡，比如使用哈希一致性。这种模式的一大缺点就是客户端与服务注册绑定，要针对服务端用到的每个编程语言和框架，实现客户端的服务发现逻辑。常用的有Eureka，zookeeper

## 3、服务端发现模式

* 客户端通过负载均衡器向某个服务提出请求，负载均衡器查询服务注册表，并将请求转发到可用的服务实例。如同客户端发现，服务实例在服务注册表中注册或注销。
![](./images/2017/12/4.png)

* 服务端发现模式兼具优缺点。它最大的优点是客户端无需关注发现的细节，只需要简单地向负载均衡器发送请求，这减少了编程语言框架需要完成的发现逻辑。并且如上文所述，某些部署环境免费提供这一功能。这种模式也有缺点。除非负载均衡器由部署环境提供，否则会成为一个需要配置和管理的高可用系统组件,常用的有Consul + nginx

## 4、服务注册表

* 服务注册表，相当于一个记录当前所有服务实例的一个容器，是服务发现机制的核心。提供管理API用于服务的注册与注销，同时提供查询API用于获得可用的服务实例

## 5、服务注册
* 服务注册很好理解，就是服务启动时，调用服务注册的管理API将服务的网络地址注册到服务注册表中，提供给查询API获取

## 6、健康检查
* 服务发现组件会通过一些机制（如心跳检测）定时检测已注册的服务，如果发现当一些服务无法访问时候，就将该服务从服务注册表中移除。

# 四、Spring Cloud
## 1、简介
* Spring Cloud是一个微服务框架，相比Dubbo等RPC框架, Spring Cloud提供的全套的分布式系统解决方案。
* Spring Cloud为开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智能路由，微代理，控制总线）。分布式系统的协调导致了样板模式, 使用Spring Cloud开发人员可以快速地支持实现这些模式的服务和应用程序。他们将在任何分布式环境中运行良好，包括开发人员自己的笔记本电脑，裸机数据中心，以及Cloud Foundry等托管平台。

## 2、为什么Spring Cloud
* 正如简介所说，spring Cloud 提供了快速构建分布式系统的常见模式工具，使用Spring Cloud开发人员可以快速地支持实现这些模式的服务和应用程序。他们将在任何分布式环境中运行良好。

## 3、特性
* 分布式/版本化配置
* 服务注册和发现
* 路由
* service - to - service调用
* 负载均衡
* 断路器
* 分布式消息传递

# 五、Spring cloud 搭建微服务

## 1、环境
* IDE：intellij idea
* spring boot： 1.5.9.RELEASE (未了解过spring boot的朋友请自行了解)
* spring cloud： Camden.SR5
* database：h2 （一个内存数据库）

## 2、说明:
* 构建事例：商城系统中商品微服务与用户微服务，模拟查询商品和购买商品
* 实体说明：示例中所有实体对象(entity) 均省去getter，setter方法

## 3、构建根项目
* 用maven 创建一个根项目 microservice-shop
  ![](./images/2017/12/5.png)

* 删除src目录

* 在pom.xml中添加Spring Cloud的组件： dependencyManagement
```xml
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>1.5.9.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
  </parent>
```
```xml
   <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Camden.SR5</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
  ```
* 并添加Spring boot的web组件：
```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
```

## 4、服务提供者
* 在microservice-shop 下构建一个maven的module作为服务提供者： microservice-shop-user
![](./images/2017/12/5.png)
* 构建为quickstart
![](./images/2017/12/6.png)


* 因跟项目中已经引用了，spring boot和springboot的test组件，故，model的pom.xml中不再添加spring-boot-starter-web和spring-boot-starter-test组件

* pom.xml中添加JPA 和H2 依赖:
```xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
    </dependency>
```
* 添加spring boot的yml文件，application.yml
```yml
    server:
      port: 9100 #服务器启动端口
    spring:
      jpa:
        generate-ddl: false
        show-sql: true #是否显示jpa的sql语句
        hibernate:
          ddl-auto: none
          naming:
            physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl #策略
      datasource:
        platform: h2 #定义数据库名称
        schema: classpath:schema.sql #定义数据库初始化建表语句
        data: classpath:data.sql #定义数据库初始化数据
      application:
        name: microservice-shop-user #服务名称
    logging:
      level:
        root: info
        org.hibernate: info
        com.ycj: info
```
* 添加初始化数据库数据
schema.sql：
```java
    DROP  TABLE  user if EXISTS ;
      CREATE TABLE  user(
      id bigint generated by default as IDENTITY ,
      longinName VARCHAR (40),
      realName VARCHAR (20),
      age int(3),
      balance DECIMAL (10,2),
      address VARCHAR (255),
      PRIMARY KEY (id)
    );
```
data.sql：
```java
    INSERT  INTO user(id,longinName,realName,age,balance,address)VALUES (1,'zhangsan','张三',20,1000.00,'北京');
    INSERT  INTO user(id,longinName,realName,age,balance,address)VALUES (2,'lisi','李四',20,2000.00,'上海');
    INSERT  INTO user(id,longinName,realName,age,balance,address)VALUES (3,'wangwu','王五',20,3000.00,'广州');
    INSERT  INTO user(id,longinName,realName,age,balance,address)VALUES (4,'zhaoliu','赵六',20,4000.00,'杭州');
    INSERT  INTO user(id,longinName,realName,age,balance,address)VALUES (5,'zhuqi','朱七',20,5000.00,'深圳');
    INSERT  INTO user(id,longinName,realName,age,balance,address)VALUES (6,'zhuqi','朱七',20,6000.00,'成都');
```
* 创建用户对象（User）：
```java
    @Entity
    public class User implements Serializable{
      @Id
      @GeneratedValue(strategy = GenerationType.AUTO)
      private Long id;
      /**
       * 用户登录名
       */
      @Column
      private String longinName;
      /**
       * 用户真实姓名
       */
      @Column
      private String realName;
      /**
       * 用户年龄
       */
      @Column
      private String age;
      /**
       * 用户余额
       */
      @Column
      private String balance;
      /**
       * 用户地址
       */
      @Column
      private String address;
    }
```
* 创建用户数据交互层Repsotry, (UserRepsotry)：
```java
    @Repository
      public interface UserRepsotry extends JpaRepository<User, Long> {
    }
```
* 创建用户数据业务逻辑层Service，因只作为服务提供者，故暂时只提供一个主键查询方法：
接口：
```java
    public interface UserService {

      User getByPrimaryKey(Long id) throws Exception;
    }
```
实现：
```java
    @Service
    public class UserServiceImpl implements UserService {

      @Autowired
      private UserRepsotry userRepsotry;

      @Override
      public User getByPrimaryKey(Long id) throws Exception {
          return userRepsotry.findOne(id);
      }
    }
```
* 创建controller层，作为接口提供方：
```java
    @RestController
    public class UserController {
      @Autowired
      private UserService userService;

      @GetMapping("/user/{id}")
      public User getByPrimaryKey(@PathVariable("id") Long id) throws Exception {
          return userService.getByPrimaryKey(id);
      }
    }
```
* 创建Spring boot的启动类：
```java
    @SpringBootApplication
    public class MicroserviceShopUserApplication {

    	public static void main(String[] args) {
    		SpringApplication.run(MicroserviceShopUserApplication.class, args);
    	}
    }
```
  注意：SpringBootApplication：注解会从当前类所在包进行扫描，扫描当前包，及其子包

* 服务提供者搭建完成，启动服务器访问接口：http://localhost:9100/user/2 ,响应：
  ```json
    {"id":1,"longinName":"zhangsan","realName":"张三","age":"20","balance":"1000.00","address":"北京"}
  ```
* 构建完毕后的项目结构：

  ![](./images/2017/12/7.png)
## 5、服务消费者
* 在microservice-shop 下构建一个maven的module作为服务消费者： microservice-shop-product，搭建方式参考服务提供者的搭建
* 构建消费端的 application.yml：
```yml
    server:
      port: 8100
    spring:
      application:
        name: microservice-shop-product
```
* 创建接收消费者中的User对象的是Entity：
```java
    public class User implements Serializable{
        private Long id;
        private String longinName;
        private String realName;
        private String age;
        private String balance;
        private String address;
    }
```
* 创建Spring boot启动类 MicroserviceShopProductApplication ：
```java
    @SpringBootApplication
    public class MicroserviceShopProductApplication {

      public static void main(String[] args) {
          SpringApplication.run(MicroserviceShopProductApplication.class, args);
      }
    }
```
* 在启动类中构建RestTemplate 实例，用于在其他地方注入
```java
    @Bean
    public RestTemplate  restTemplate() {
       return new RestTemplate();
    }
```
注：RestTemplate 是spring提供的用于调用远程服务器的一个类，有点类似于一个WebService客户端请求的模版，可以调用http请求的WebService，并将结果转换成相应的对象类型；

* 构建Controller，并在中调用服务提供者microservice-shop-user的用户查询接口
```java
@RestController
public class ProductController {

    @Autowired
    RestTemplate restTemplate;

    @GetMapping("user/{id}")
    public User checkUser(@PathVariable("id") Long id) {
        return restTemplate.getForObject("http://localhost:9100/user/" + id, User.class);
    }
}
```
* 启动microservice-shop-user，microservice-shop-product两个服务
  调用http://localhost:9100/user/{id} 和调用http://localhost:8100/user/{id} 取得同样的结果，则服务提供者与消费者均搭建成功，如id为1时，取值都为：
```json
    {"id":1,"longinName":"zhangsan","realName":"张三","age":"20","balance":"1000.00","address":"北京"}
```
* 服务器提供者，消费者均搭建完毕，如何解决如下问题？

  → 服务并未统筹管理，且并未涉及服务注册表等，看起来仅仅是项目之间的接口调用。
  → restTemplate.getForObject("http://localhost:9100/user/" + id, User.class); 地址为固定地址，如果涉及服务器变动，则需要手动修改代码中的接口地址，虽说可以提供配置文件，一旦服务大规模集群后，接口地址配置繁多。

## 6、服务注册与发现工具：Eureka
### a、简介
* Eureka 是Netflix开源的一个RESTful服务，用来定位运行在AWS域（Region）中的中间层服务。
* Eureka由两个组件组成：Eureka服务器和Eureka客户端。
* Eureka服务器用作服务注册服务器
* Eureka客户端是一个java客户端，用来简化与服务器的交互、作为轮询负载均衡器，并提供服务的故障切换支持。Netflix在其生产环境中使用的是另外的客户端，它提供基于流量、资源利用率以及出错状态的加权负载均衡。
### b、构建Eureka 服务器
* 在microservice-shop 下构建一个maven的module作为Eureka服务 microservice-shop-eureka
* 在pom.xml中添加eureka组件：
```xml
		<dependency>
			 <groupId>org.springframework.cloud</groupId>
			 <artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>
```
* 配置application.yml：
```yml
    server:
      port: 8761
    eureka:
      client:
        register-with-eureka: false #本机是否启用eureka客户端，eureka集群时候开启
        fetch-registry: false
        service-url:
          defaultZone: http://127.0.0.1:8761/eureka #服务注册地址
```
* 编写eurka服务器的启动类：
```java
		@SpringBootApplication
		@EnableEurekaServer
		public class MicroserviceShopEurekaApplication {
			public static void main(String[] args) {
				SpringApplication.run(MicroserviceShopEurekaApplication.class, args);
			}
		}
```
* 启动eurka服务，并访问 http://127.0.0.1:8761/eureka

 ![](./images/2017/12/8.png)
### c、将用户微服务注册到Eureka
* 打开microservice-shop-user 在其pom.xml文件中添加Eureka的组件
```xml
	 <dependency>
			 <groupId>org.springframework.cloud</groupId>
			 <artifactId>spring-cloud-starter-eureka</artifactId>
	 </dependency>
```
* 在microservice-shop-user 的application.yml文件中添加Eureka的配置
```yml
  eureka:
    client:
      healthcheck:
        enabled: true #开启健康检查,心跳等定期检查服务器情况 ,需要添加spring-boot-starter-actuator 组件
      serviceUrl:
        defaultZone: http://127.0.0.1:8761/eureka
    instance:
      prefer-ip-address: true
      instance-id: ${spring.application.name}:${spring.cloud.client.ipAddress}:${spring.application.instance_id:${server.port}} # 修改注册服务名
      metadata-map:
        zone: ABC      # eureka可以理解的元数据
        lilizhou: BBC  # 不会影响客户端行为
      lease-renewal-interval-in-seconds: 5 #续约更新时间间隔（默认30秒）
      lease-expiration-duration-in-seconds: 90 	# 续约到期时间（默认90秒）
```
* 在microservice-shop-user的启动类上添加想eureka注册的注解 @EnableEurekaClient 或者 @EnableDiscoveryClient
```java
		@SpringBootApplication
		@EnableEurekaClient //Eureka专用，只能Eureka进行服务注册
		//@EnableDiscoveryClient //可向其他服务进行注册
		public class MicroserviceProviderUserApplication {
		    public static void main(String[] args) {
		        SpringApplication.run(MicroserviceProviderUserApplication.class, args);
		    }
		}
```
 注意: @EnableEurekaClient 只可以向Eureka进行服务注册, @EnableDiscoveryClient 可以像其他服务注册，如果开启了健康检查，必须要添加spring-boot-starter-actuator组件，否则无法执行健康检查，服务无法启动

* 启动microservice-shop-eureka 和 microservice-shop-user，打开eureka的管理页面,可以看到服务的注册信息，如下：
 ![](./images/2018/01/9.png)
	如上图，则已经成功将用户微服务注册到了eureka
### c、将商品服务注册到eureka，并通过服务方式进行调用

* 参照用户微服务，将商品微服务注册到eureka
 ![](./images/2018/01/10.png)
### d、eureka的权限认证

* eureka权限认证需要添加spring-boot-starter-security组件,在microservice-shop-eureka的pom.xml中添加
```xml
	<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
	</dependency>
```
* 修改microservice-shop-eureka配置文件，在application.yml中添加权限配置
```yml
  security:
    basic:
      enabled: true
    user:
      name: user
      password: password123
```
* 修改microservice-shop-eureka配置文件，将eureka.client.service-url.defaultZone修改为 http://user:password123@localhost:8761/eureka ,同时将microservice-shop-user，microservice-shop-product中也该为相同配置
```yml
  eureka:
    client:
      service-url:
        defaultZone: http://user:password123@localhost:8761/eureka
```
* 启动microservice-shop-eureka 服务，访问登录地址，检查是否提示需要登录。
* 启动microservice-shop-user，microservice-shop-product，是否能够正常注册，修改掉注册账号密码，检查是否正常注册。


## 7、客户端软件负载均衡算法者：Ribbon
### a、简介
* Ribbon 是 Netflix 发布的云中间层服务开源项目，其主要功能是提供客户侧软件负载均衡算法，将 Netflix 的中间层服务连接在一起。Eureka 是一个 RESTful 服务，用来定位运行在 AWS 域（Region）中的中间层服务。与Eureka天然集成，且Eureka中自带Ribbon

### b、Ribbon的意义
* Ribbon 的负载均衡算法、区域感知负载均衡器久经考验，可以直接拿来使用。，Ribbon 维护了一个服务器列表，如果服务器有宕机现象，Ribbon 能够自行将其剔除；但如果该服务器故障排除，重新启动，或者增加新的负载节点，我们需要手工调用 Ribbon 的接口将其动态添加进 Ribbon 的服务器列表。这样明显不够尽如人意。如何能够在服务节点启动时，自行添加服务列表？—— Eureka。Eureka 提供了 Application Service 客户端的自行注册的功能。此外，Eureka 的缓存机制能够防止大规模宕机带来的灾难性后果
* 在microservice-shop 下构建一个maven的module作为Eureka服务 microservice-shop-ribbon

### c、Ribbon的使用
* Eureka带有Ribbon组件，故不用再添加Ribbon的组件
* 在构建RestTemplate 的bean时候，添加 @LoadBalanced 注解，这个注解会自动构造LoadBalancerClient接口的实现类并注册到Spring容器中。接下来使用RestTemplate进行rest操作的时候，会自动使用负载均衡策略，它内部会在RestTemplate中加入LoadBalancerInterceptor这个拦截器，这个拦截器的作用就是使用负载均衡。代码如下：
```java
	@LoadBalanced
	 @Bean
	 public RestTemplate restTemplate() {
			 return new RestTemplate();
	 }
```
### d、检查ribbon的服务识别

* 将microservice-shop-product 中调用microservice-shop-user服务的代码作出调整 通过以服务名称方式进行访问，将restTemplate.getForObject("http://localhost:9100/user/" + id, User.class);  改为  restTemplate.getForObject("http://microservice-shop-user/user/" + id, User.class);
* 访问microservice-shop-product， 调用用户服务的接口：http://localhost:8101/user/1
```json
	{"id":1,"longinName":"zhangsan","realName":"张三","age":"20","balance":"1000.00","address":"北京"}
```

### d、检查ribbon的负载
* 启动一个 microservice-shop-user服务，完成之后修改 application.yml文件的端口，再次启动，从Eureka客户端可以看到，用户服务被注册了2个，端口分别为9100,9101；
	![](./images/2018/01/11.png)
* 继续通过microservice-shop-product ，服务调用用户服务的接口，并观察两台用户服务是否 调用日志产生。
### e、调整ribbon的负载策略
* ribbon的默认负载策略为轮询策略
* ribbon支持的负载策略有：
	- BestAvailableRule：选择一个最小的并发请求的server
	- AvailabilityFilteringRule：过滤掉那些因为一直连接失败的被标记为circuit tripped的后端server，并过滤掉那些高并发的的后端server（active connections 超过配置的阈值）
	- WeightedResponseTimeRule：根据响应时间分配一个weight，响应时间越长，weight越小，被选中的可能性越低。
	- RetryRule： 对选定的负载均衡策略机上重试机制。
	- RoundRobinRule： roundRobin方式轮询选择server。
	- RandomRule： 随机选择一个server。
	- ZoneAvoidanceRule： 复合判断server所在区域的性能和server的可用性选择server。
* 配置负载策略，在product微服务的 application.yml中添加配置策略为随机策略，配置如下：
```yml
  microservice-provider-user:
    ribbon:
      NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```
## 8、Java HTTP客户端库:Feign
### a、简介
* Feign使得 Java HTTP 客户端编写更方便。Feign 灵感来源于Retrofit、JAXRS-2.0和WebSocket。Feign最初是为了降低统一绑定Denominator到HTTP API的复杂度，不区分是否支持Restful。Feign旨在通过最少的资源和代码来实现和HTTP API的连接。通过可定制的解码器和错误处理，可以编写任意的HTTP API
### b、用Feign改造 microservice-shop-product 微服务
* 在microservice-shop-product的pom.xml中添加 Feign的组件
```xml
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-feign</artifactId>
	</dependency>
```
* microservice-shop-product 微服务启动器中移除RestTemplate 的构建
```Java
		@SpringBootApplication
		@EnableFeignClients
		@EnableEurekaClient
		public class MicroserviceShopProductApplication {
			//  @Bean
			//    @LoadBalanced
			//    public RestTemplate restTemplate() {
			//        return new RestTemplate();
			//    }
			public static void main(String[] args) {
					SpringApplication.run(MicroserviceShopProductApplication.class, args);
			}
		}
```
	- @EnableFeignClients 声明需要使用Feign客户端能力。

* 编写UserFeignUtis 用于访问调用microservice-shop-user用户维护服务接口
```java
		@FeignClient("microservice-shop-user")
		public interface UserFeignClient {
			@RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
			User getByPrimaryKey(@PathVariable("id") Long id) throws Exception;
		}
```
 - @FeignClient() 注解指定FeignClient访问的微服务名称。
 - public User findUser(@PathVariable("id") Long id) ，通过定义访问用户微服务的接口

* 修改microservice-shop-product 微服务中ProductController调用 microservice-shop-user的方式，如下：
```Java
		@RestController
		public class ProductController {

		    @Autowired
		    UserFeignClient userFeignClient;

		    @GetMapping("user/{id}")
		    public User checkUser(@PathVariable("id") Long id) throws Exception {
		        return userFeignClient.getByPrimaryKey(id);
		    }
		}
```
* 启动eureka，用户微服务，商品微服务，检查接口是否正常
