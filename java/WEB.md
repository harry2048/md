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

## 2 获取kibana的cookie访问主页

```html
<!DOCTYPE html>
<html lang="en"> <!-- zh-CN -->
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>后端获取cookie访问kibana</title>
    <script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js"></script>
</head>
<body>
写代码是一件非常快乐的事 happy
<p>wo love you </p>

<h1>这是标题1</h1>
<h2>这是标题2</h2>
<input type="button" value="login" onclick="login()"/>
<script type="text/javascript">
    function login() {
        var cooki = getKibanaCookie();
        var exdate  = new Date();
        exdate.setTime(exdate.getTime() + 10*60*1000);      //保存10分钟
        // web项目中启动，document.cookie才会生效
        document.cookie =cooki+';expires='+exdate.toGMTString()+';';
        window.open("http://localhost:5601");
    }

    function getKibanaCookie(){
        var url = "http://localhost:8080/testString";
        var coo;
        $.ajax({
            url: url,
            dataType: 'json',
            type: 'GET',
            async:false,
            success: function(result){
                coo= result.kbn_cookie;
            }
        });
        return coo;
    }
</script>
</body>
</html>
```

```java
// SpringBoot
@CrossOrigin
@GetMapping("/testString")
public Map<String, Object> testString() {
    String url = "http://localhost:5601/internal/security/login";
    HttpHeaders headers = new HttpHeaders();
    headers.add("kbn-xsrf","123");
    Map<String, Object> map = new HashMap<>(2);
    map.put("username", "elastic");
    map.put("password", "123456");
    HttpEntity<Object> entity = new HttpEntity<>(map, headers);


    RestTemplate restTemplate = new RestTemplate();
    ResponseEntity<Object> exchange = restTemplate.exchange(url, HttpMethod.POST, entity, Object.class);
    String cookie = exchange.getHeaders().get("set-cookie").get(0).split(";")[0];

    Map<String, Object> result = new HashMap<>(2);
    result.put("kbn_cookie", cookie);
    return result;
}
```

