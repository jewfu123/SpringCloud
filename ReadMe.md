// -------- 2024-6-4 ---------- //

# 微服务

## 发起远程调用
1. Application 注入Bean
2. in Service，insert bean from restTemplate
	```Java
	@Service
	public class OrderService {
		@Autowired
		private RestTemplate restTemplate;

		public Order queryOrderById(Long orderId) {
			Order order = orderMapper.findById(orderId);
			String url = "http://localhost:8081/user" + order.getUserId();
			User user = restTemplate.getForObject(url, User.class);
			order.setUser(user);
			return order;
		}
	}
	```

## 搭建EurekaServer
1. 引依赖 eureka-server
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
			<version>4.1.1</version>
		</dependency>
		*注意版本对好，例如：
		springboot 3.3.0 vs. eureka 4.1.1 vs. java 17

2. 加注解 @EnableEurekaServer注解

3. 配置信息 application.yml中配置eureka地址
	```java
	server:
	  port: 10086
	spring:
	  application:
	    name: eurekaserver
	eureka:
	  client:
	    service-url:
	      defaultZone:  http://127.0.0.1:10086/eureka/
	```

4. How to create modules in eclipse ide:
	```java
	cd parent-project
	mvn archetype:generate -DgroupId=com.websystique.multimodule  -DartifactId=model-lib
	mvn archetype:generate -DgroupId=com.websystique.multimodule  -DartifactId=webapp1
	mvn archetype:generate -DgroupId=com.websystique.multimodule  -DartifactId=webapp2
	Now if you open the parent-project pom.xml, you will find all three modules being added in there.

	<modules>
	  <module>model-lib</module>
	  <module>webapp1</module>
	  <module>webapp2</module>
	</modules>
	Also, in each sub-module’s pom.xml, a parent section being added.
	```

5. add Eureka client 3 steps:
	1) add dependency in client's pom.xml
		eureka-client依赖

	2) add settings in application.yml
		eureka's url:

	3) add client app name in above yml.
		application:
			name: abcClient

6. 服务的拉取：
	1) 修改OrderService的代码，修改访问的Url路径，用服务名代替ip,端口：
		String url = "http://userservice/user/" + order.getUserId();

	2) 在Order-service项目的启动类OrderApplication中的RestTemplate添加负载均衡注解:
		@Bean
		@LoadBalanced
		public RestTemplate restTemplate() {
			return new RestTemplate();
		}


7. Eureka注册中心
	1. 搭建EurekaServer
		1) 引入eureka-server依赖
		2) 添加@EnableEurekaServer注解
		3) 在application.yml中配置eureka地址

	2. 服务注册
		1) 引入eureka-client依赖
		2) 在application.yml中配置eureka地址

	3. 服务发现
		1) 引入eureka-client依赖
		2) 在application.yml中配置eureka地址
		3) 给RestTemplate添加@LoadBalanced注解
		4) 用服务提供者的服务名称远程调用


## Feign（替代 RestTemplate）
#### 编写Feign的客户端
	in interface file: UserClient.java

	@FeignClient("userservice")
	public interface UserClient {
		@GetMapping("/user/{id}")
		User findById(@PathVariable("id") Long id);
	}

	*使用时，需要加入自动装配：
	in OrderService.java

	@AutoWired
	private OrderMapper orderMapper;

#### Application中加入注解：
	@EnableFeignClients
	public class OrderApplication {

		@Bean
		@LoadBalanced
		public RestTemplate restTemplate() {
			return new RestTemplate();

		}
	}

#### 使用Feign的好处：
	1. 实现远程调用
	2. 自身集成有负载均衡


#### RabbitMQ的概念术语：
	1. channel: 操作MQ的工具
	2. exchange: 路由消息到队列中
	3. queue: 缓存消息
	4. virtual host: 虚拟主机，是对queue, exchange等资源的逻辑分组


#### SpringAMQP如何接收消息？
	1. 引入amqp的starter依赖
	2. 配置RabbitMQ地址
	3. 定义类，添加@Component注解
	4. 类中声明方法，添加@RabbitListener注解，方法参数就是消息
	注意：消息一旦消费就会从队列中删除，RabbitMQ没有消息回溯功能

#### prefetch 消费预取限制
	修改application.yml文件，设置preFetch这个值，可以控制预取消息的上限。

#### Wokr模型的使用
	1. 多个消费者绑定到一个队列，同一条消息只会被一个消费者处理
	2. 通过设置prefetch来控制消费者预取的消息数量

#### 将队列绑定交换机
	1. 在consumer服务声明Exchange, Queue, Binding
		1> 声明queue
		2> 声明exchange
		3> 设置key
	2. 在consumer服务声明两个消费者

	*总结：
		1. 交换机的作用：
			1> 接收publisher发送的消息
			2> 将消息按照规则路由到与之绑定的队列
			3> 不能缓存消息，路由失败，消息丢失
			4> FanoutExchange的会将消息路由到每个绑定的队列

		2. 声明队列，交换机，绑定关系的Bean是什么？

		*交换机只起到路由的作用，不储存消息，路由后的消息，有可能会丢失。

#### 发布订阅--DirectExchange
	Direct Exchange 会将接收到的消息根据规则路由到指定的Queue，因此成为路由模式( routes )
	1. 每一个Queue都与Exchange设置一个BindingKey
	2. 发布者发送消息时，指定消息的RoutingKey
	3. Exchange将消息路由到BindingKey与消息RoutingKey一致的队列











































































