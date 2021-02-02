## 1 创建maven项目，指定jdk1.8

```xml
<!-- 在maven的settings.xml的<profiles>中添加 -->
<profile>
	<id>jdk-1.8</id>
    <activation>
    	<activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
    	<maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```

## 2 指定阿里云仓库

```xml
<mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
```

## 3 maven test 中文乱码

```xml
<!--若有乱码，则在pom文件中添加-->
<build>
	<plugins>
    	<groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.7.2</version>
        <configuration>
        	<forkMode>once</forkMode>
            <argLine>-Dfile.encoding=UTF-8</argLine>
        </configuration>
    </plugins>
</build>
```

## 4 maven 域

### 依赖传递

<scope>为test时不传递，为compile时传递依赖

传递时，就近原则。使用最先依赖jar包的版本号

### 域

* compile  **默认**

  main目录下的java代码<font color=#FF0000 >可以</font>访问这个范围下的依赖

  test目录下的java代码<font color=#FF0000 >可以</font>访问这个范文下的依赖

  打包时，会打到WEB-INF下的lib文件夹中

* test

  main目录下的java代码<font color=#FF0000 >不可以</font>访问

  test目录下的java代码<font color=#FF0000 >可以</font>访问

  打包时，不会打进lib

* provided     tomcat jar包

  main目录下的java代码<font color=#FF0000 >可以</font>访问

  test目录下的java代码<font color=#FF0000 >可以</font>访问

  打包时，不会打进lib

* runntime  数据库驱动

  main目录下的java代码<font color=#FF0000 >可以</font>访问

  test目录下的java代码<font color=#FF0000 >可以</font>访问

  打包时，会打进lib

  跳过编译

* system  

  配合systemPath标签使用，读取本地jar包

## 5 排除依赖

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
```

## 6 统一maven版本号

```xml
<properties>
    <java.version>1.8</java.version>
    <spring.version>5.0.0.RELEASE</spring.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
 		<version>${spring.version}</version>
    </dependency>
</dependencies>
```

## 7 继承父项目

### 继承

* 父项目<packaging>pom</packaging>

* 在子项目中pom.xml中加入

```xml
<parent>
	<groupId>com.baidu</groupId>
    <artifactId>parent</artifactId>
    <version>4.0.0.RELEASE</version>
    <!--指定父pom的路径-->
    <relativePath>../parent/parent.xml</relativePath>
</parent>
```

* 父项目中管理依赖

```xml
<dependencyManagement>
	<dependencies>
    	<dependency>
        	<groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

因为父项目管理了依赖，子项目不需要版本号和域
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
    </dependency>
</dependencies>
```

### 聚合 module

父项目引入子项目

```xml
<!--在父项目的pom文件中-->
<groupId>com.baidu</groupId>
<artifactId>harry</artifactId>
<version>4.0.0.RELEASE</version>
<packaging>pom</packaging>

<properties>
    <java.version>1.8</java.version>
    <spring.version>5.0.0.RELEASE</spring.version>
</properties>
<modules>
    <!--相对路径-->
	<module>harry-core</module>
</modules>
<dependencyManagement>
	<dependencies>
    	<dependency>
        	<groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>
        <dependencies>
    	<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
<build>
	<pluginManagement>
    	<plugins>
        	<plugin>
            	<groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                	<source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
    <plugins>
    	<plugin>
        	<groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>2.6</version>
            <configuration>
            	<failOnMissingWebXml>false</failOnMissingWebXml>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### demo

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.baidu</groupId>
        <artifactId>parent</artifactId>
        <version>4.0.0.RELEASE</version>
        <!--指定父pom的路径-->
        <relativePath>../parent/parent.xml</relativePath>
    </parent>
    
    <groupId>org.tinygame</groupId>
    <artifactId>herostory</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
    </dependencies>

    <build>
        <!--编译main目录下的xml-->
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
        </resources>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## 8 maven使用tomcat插件启动web项目

```xml
<!-- 在pom文件中添加 -->
<project>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <path>/</path>
                    <port>8080</port>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

https://blog.csdn.net/ron03129596/article/details/79161139

