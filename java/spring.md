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