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









































































































