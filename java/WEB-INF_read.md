## 1.web项目读取WEB-INF目录下的文件

### ServletContext

```java
@Autowired
private ServletContext servletContext;

@GetMapping("/readLine")
public void readLine(){
    try (InputStream in = servletContext.getResourceAsStream("/WEB-INF/ec.properties")) {
        Properties p = new Properties();
        p.load(in);

        p.entrySet().forEach(System.out::println);
    } catch (Exception ex) {
        ex.printStackTrace();
    }
}

// ec.properties 配置文件必须是no bom格式下创建的，否则需要显示转编码格式为UTF-8
InputStreamReader reader = new InputStreamReader(in, "UTF-8");
```



### request

```java
@GetMapping("/test")
public void test(HttpServletRequest request) {
    ServletContext servletContext = request.getServletContext();
    String realPath = servletContext.getRealPath("/");
    //System.out.println(realPath);

    realPath += "/WEB-INF/ec.properties";
    //System.out.println(realPath);

    File file = new File(realPath);
    try (InputStream in = new FileInputStream(file)) {
        Properties prop = new Properties();
        prop.load(in);

        prop.entrySet().forEach(e ->{
            String key = (String)e.getKey();
            String value = (String)e.getValue();
 			System.out.println(key +" " + value);
        });
    } catch (Exception ex) {
        ex.printStackTrace();
    }

}
```

