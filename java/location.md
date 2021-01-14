## 获取classes路径

```java
URL resource = HibernateDemo.class.getClassLoader().getResource("");
String path = resource.getPath();
```

> 打印如下
>
> /D:/workspace/github/boy/target/classes/

## 各个路径

```java
String path1 = HibernateDemo.class.getResource("/").getPath();
System.out.println("path1:" +path1);

String path2 = HibernateDemo.class.getResource("").getPath();
System.out.println("path2:" +path2);

String path3 = new File("").getAbsolutePath();
System.out.println("path3:"+path3);

String path4 = new File("").getCanonicalPath();
System.out.println("path4:" +path4);
// 打印如下
//path1:/D:/workspace/github/boy/target/classes/
//path2:/D:/workspace/github/boy/target/classes/com/baidu/boy/testBO/
//path3:D:\workspace\github\boy
//path4:D:\workspace\github\boy
```

```
// 项目中使用方式
URL url = HibernateDemo.class.getResource("/");
if (null == url){
	url = HibernateDemo.class.getResource("");
}
String path = url.getPath();
```



## 获取maven项目resource路径

```java
String path = "application.properties";// 非/ 开头，不需要classpath
InputStream in = HibernateDemo.class.getClassLoader().getResourceAsStream(path);
```

