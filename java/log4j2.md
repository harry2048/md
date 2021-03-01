## 1. 配置文件

### log4j2  要使用高于2.6版本

高于2.6的版本提高了效率，减少了临时对象的创建

### log4j2.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
    status="warn" 日志框架本身的输出日志级别
    monitorInterval="5" 自动加载配置文件的间隔时间，不低于 5 秒
-->
<Configuration status="debug" monitorInterval="5">

    <!--
        集中配置属性进行管理
        使用时通过:${name}
    -->
    <properties>
        <property name="LOG_HOME">/logs</property>
    </properties>

    <!--日志处理-->
    <Appenders>
        <!--控制台输出 appender-->
        <Console name="Console" target="SYSTEM_ERR">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] [%-5level] %c{36}:%L %m%n"/>
        </Console>

        <!--日志文件输出 appender-->
        <File name="file" fileName="${LOG_HOME}/myfile.log">
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] %l %c{36} %m%n"/>
        </File>
        
        <!--使用随机读写刘的日志文件输出 appender，性能提高-->
        <RandomAccessFile name="accessFile" fileName="${LOG_HOME}/myAcclog.log">
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] %l %c{36} - %m%n" />
        </RandomAccessFile>

        <!--按照一定规则拆分的日志文件的 appender-->
        <RollingFile name="rollingFile" fileName="${LOG_HOME}/myrollog.log"
                     filePattern="/logs/$${date:yyyy-MM-dd}/myrollog-%d{yyyy-MM-dd-HH-mm}-%i.log">
            <!--日志级别过滤器-->
            <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY" />
            <!--日志消息格式-->
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] %l %c{36} - %msg%n" />
            <Policies>
                <!--在系统启动时，出发拆分规则，生产一个新的日志文件-->
                <OnStartupTriggeringPolicy />
                <!--按照文件大小拆分，10MB -->
                <SizeBasedTriggeringPolicy size="10 MB" />
                <!--按照时间节点拆分，规则根据filePattern定义的-->
                <TimeBasedTriggeringPolicy />
            </Policies>
            <!--在同一个目录下，文件的个数限定为 30 个，超过进行覆盖-->
            <DefaultRolloverStrategy max="30" />
        </RollingFile>

    </Appenders>

    <!--logger 定义-->
    <Loggers>

        <!--自定义异步 logger 对象
            includeLocation="false" 关闭日志记录的行号信息
            additivity="false" 不在继承 rootlogger 对象
        -->
        <AsyncLogger name="com.itheima" level="trace" includeLocation="false" additivity="false">
            <AppenderRef ref="Console"/>
        </AsyncLogger>

        <!--使用 rootLogger 配置 日志级别 level="trace"-->
        <Root level="trace">
            <!--指定日志使用的处理器-->
            <AppenderRef ref="Console" />
        </Root>
    </Loggers>
</Configuration>
```

## 2. SpringBoot使用log4j2

### 引入maven dependent

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
        	<groupId>org.springframework.boot</groupId>
        	<artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<!--异步日志依赖-->
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.3.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
<!--剔除logback依赖，当不使用web时，使用下列方式剔除-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### application.yml

```yml
# 指定log4j2配置文件路径
logging:
  config: classpath:log4j2.xml
```

### log4j2.xml

生产环境中要去掉console的输出

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration monitorInterval="5" status="info">
    <Properties>
        <property name="LOG_PATTERN" value="%date{yyyy-MM-dd HH:mm:ss.SSS} %t %level %c %m%n" />
        <property name="FILE_PATH" value="logs/" />
        <property name="FILE_NAME" value="boy.log"/>
    </Properties>

    <appenders>
        <console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="${LOG_PATTERN}"/>
            <ThresholdFilter level="info"/>
        </console>
        <RollingFile name="rollingFileInfo" fileName="${FILE_PATH}/info.log" filePattern="${FILE_PATH}/${FILE_NAME}-INFO-%d{yyyy-MM-dd}_%i.log">
            <PatternLayout pattern="${LOG_PATTERN}"/>
            <Policies>
                <!--interval属性用来指定多久滚动一次，默认是1 hour-->
                <TimeBasedTriggeringPolicy interval="24"/>
                <SizeBasedTriggeringPolicy size="200MB"/>
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
            <DefaultRolloverStrategy max="100"/>
        </RollingFile>
    </appenders>

    <loggers>
        <logger name="org.mybatis" level="info" additivity="false">
            <AppenderRef ref="Console"/>
        </logger>
        <Logger name="org.springframework" level="WARN" additivity="false">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="rollingFileInfo"/>
        </Logger>
        <!-- 局部异步 includeLocation 是否打印行号 -->
        <AsyncLogger  name="com.baidu.boy" level="info" additivity="false" includeLocation="false">
            <AppenderRef ref="rollingFileInfo"/>
            <AppenderRef ref="Console"/>
        </AsyncLogger >
        <!-- 缺省输出,当没有配置时,走root配置 -->
        <root level="INFO">
            <appender-ref ref="Console"/>
            <appender-ref ref="rollingFileInfo"/>
        </root>
    </loggers>
</configuration>
```

### demo

```java
@SpringBootTest
@Slf4j
class BoyApplicationTests {
    @Test
    void contextLoads() {
        for (int i = 0; i < 10; i++) {
            log.info("this is info" + i);
            log.warn("what's it ? -> {}", "is warn");
            log.error("this is error" + i);
        }
    }
}
```

## 3. maven项目使用log4j2

### 引入maven

```xml
<dependencies>
        <!--使用slf4j 作为日志门面-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.26</version>
        </dependency>

        <!--使用 log4j2 的适配器进行绑定-->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>2.9.1</version>
        </dependency>

        <!--log4j2日志门面-->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.11.1</version>
        </dependency>
        <!--log4j2 日志实现-->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.11.1</version>
        </dependency>

        <!--异步日志依赖-->
        <dependency>
            <groupId>com.lmax</groupId>
            <artifactId>disruptor</artifactId>
            <version>3.3.4</version>
        </dependency>

        <!--junit 单元测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <!-- 设置编译版本为1.8 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

### log4j2.xml

```xml
	<?xml version="1.0" encoding="UTF-8"?>
<!--
    status="warn" 日志框架本身的输出日志级别
    monitorInterval="5" 自动加载配置文件的间隔时间，不低于 5 秒
-->
<Configuration status="debug" monitorInterval="5">

    <!--
        集中配置属性进行管理
        使用时通过:${name}
    -->
    <properties>
        <property name="LOG_HOME">/logs</property>
    </properties>

    <!--日志处理-->
    <Appenders>
        <!--控制台输出 appender-->
        <Console name="Console" target="SYSTEM_ERR">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] [%-5level] %c{36}:%L --- %m%n" />
        </Console>

        <!--日志文件输出 appender-->
        <File name="file" fileName="${LOG_HOME}/myfile.log">
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] %l %c{36} - %m%n" />
        </File>

        <!--<Async name="Async">-->
            <!--<AppenderRef ref="file"/>-->
        <!--</Async>-->

        <!--使用随机读写刘的日志文件输出 appender，性能提高-->
        <RandomAccessFile name="accessFile" fileName="${LOG_HOME}/myAcclog.log">
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] %l %c{36} - %m%n" />
        </RandomAccessFile>

        <!--按照一定规则拆分的日志文件的 appender-->
        <RollingFile name="rollingFile" fileName="${LOG_HOME}/myrollog.log"
                     filePattern="/logs/$${date:yyyy-MM-dd}/myrollog-%d{yyyy-MM-dd-HH-mm}-%i.log">
            <!--日志级别过滤器-->
            <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY" />
            <!--日志消息格式-->
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] %l %c{36} - %msg%n" />
            <Policies>
                <!--在系统启动时，出发拆分规则，生产一个新的日志文件-->
                <OnStartupTriggeringPolicy />
                <!--按照文件大小拆分，10MB -->
                <SizeBasedTriggeringPolicy size="10 MB" />
                <!--按照时间节点拆分，规则根据filePattern定义的-->
                <TimeBasedTriggeringPolicy />
            </Policies>
            <!--在同一个目录下，文件的个数限定为 30 个，超过进行覆盖-->
            <DefaultRolloverStrategy max="30" />
        </RollingFile>

    </Appenders>

    <!--logger 定义-->
    <Loggers>


        <!--自定义异步 logger 对象
            includeLocation="false" 关闭日志记录的行号信息
            additivity="false" 不在继承 rootlogger 对象
        -->
        <AsyncLogger name="com.itheima" level="trace" includeLocation="false" additivity="false">
            <AppenderRef ref="Console"/>
        </AsyncLogger>

        <!--使用 rootLogger 配置 日志级别 level="trace"-->
        <Root level="trace">
            <!--指定日志使用的处理器-->
            <AppenderRef ref="Console" />

            <!--使用异步 appender  效率较低-->
            <!--<AppenderRef ref="Async" />-->
        </Root>
    </Loggers>
</Configuration>
```

### demo

```java
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Slf4jTest {

    public static final Logger LOGGER = LoggerFactory.getLogger(Slf4jTest.class);

    // 快速入门
    @Test
    public void test01()throws Exception{
        // 日志输出
        LOGGER.error("error");
        LOGGER.warn("wring");
        LOGGER.info("info");
        LOGGER.debug("debug");
        LOGGER.trace("trace");
    }
}
```

