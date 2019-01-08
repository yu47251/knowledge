# Eureka
- 启动类中的启动方式：
- 注意：大多数网上的介绍，web方法中传入的参数为true，新版本提供多种容器方式，SERVLET,REACIVE,NONE。
- SERVLET对应的是tomcat，REACTIVE对应netty

```java
package com.example.eurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.WebApplicationType;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
       new SpringApplicationBuilder(EurekaServerApplication.class)
			.web(WebApplicationType.SERVLET).run(args);
    }

}

```