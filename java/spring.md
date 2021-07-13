## @Autowired给静态变量赋值

> 将yml中的字段赋值到静态变量，使用@Value，思路相同

```java
package com.baidu.boy.config;

import com.baidu.boy.testBO.HibernateDemo;
import com.baidu.boy.testBO.PlatformTransactionManagerDemo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class PoinGlobalUtil {

    private static HibernateDemo hibernate;  // A 是hibernate
    private static PlatformTransactionManagerDemo transaction;  // B 是TransactionPlat

    @Autowired
    private void setA(HibernateDemo hibernate) {
        PoinGlobalUtil.hibernate = hibernate; 
    }
    
    @Autowired
    private void setB(PlatformTransactionManagerDemo b) {
        PoinGlobalUtil.transaction = b;
    }

    public static void init () {
        System.out.println(hibernate.name + " " + transaction.name);
    }

    public static void saveOrUpdate(String msg) {
        hibernate.saveA(msg);
    }
}

```

## SpringUtil -> SpringBean

> 通过springBean获取指定类的SpringBean

```java
@Component
public class SpringBean {
    private static ApplicationContext springContext;

    public static Object getBean(String name) {
        return springContext.getBean(name);
    }

    public static <T> T getBean(Class<T> clazz) {
        return springContext.getBean(clazz);
    }

    @Override
    public void setApplicationContext(ApplicationContext ac) throws Exception {
        SpringBean.springContext = applicationContext;
    }

    public static ApplicationContext getApplicationContext() {
        return springContext;
    }
}
```

## redis实现session共享

https://www.cnblogs.com/david1216/p/11468772.html

https://blog.csdn.net/liuxiao723846/article/details/80733565

### SpringBoot 使用SpringSession

#### 1 引入maven

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
</properties>

<dependencies>
    <!--spring boot-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--spring session-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
</dependencies>   
```

#### 2 配置redis

```properties
server.port=8080
spring.redis.host=localhost
spring.redis.port=6379

#spring session使用存储类型，默认就是redis所以可以省略
spring.session.store-type=redis
```

#### 3 启动类加入注解

```java
// @EnableCaching
@EnableRedisHttpSession
@SpringBootApplication
public class SpringsessionApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringsessionApplication.class, args);
    }
}
```

#### 4 编写控制器

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.servlet.http.HttpServletRequest;

@RestController
public class IndexController {

    @RequestMapping("/show")
    public String show(HttpServletRequest request){
        return "I'm " + request.getLocalPort();
    }


    @RequestMapping(value = "/session")
    public String getSession(HttpServletRequest request) {
        request.getSession().setAttribute("userName", "admin");
        return "I'm " + request.getLocalPort() + " save session " + request.getSession().getId();
    }

    @RequestMapping(value = "/get")
    public String get(HttpServletRequest request) {
        String userName = (String) request.getSession().getAttribute("userName");
        return "I'm " + request.getLocalPort() + " userName :" + userName;
    }
    
}
```

#### 5 配置nginx

```conf
upstream  ngixServers{
        server localhost:8081;
        server localhost:8082;
    }
    server {
        listen       8888;
        server_name  localhost;
        location / {
        proxy_pass http://ngixServers;
        }
    }
```

测试session_id是否相同即可