post请求 发json

```java
@Bean
public RestTemplate restTemplate(){
    SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
    factory.setReadTimeout(10000);//单位为ms
    factory.setConnectTimeout(10000);//单位为ms
    return new RestTemplate(factory);
}
```

```java
String url = "http://localhost:8080/ms/webapi/..";
HttpHeaders headers = new HttpHeaders();
httpHeaders.setContentType(MediaType.APPLICATION_JSON);
Map<String, Object> map = new HashMap<>();
map.put("name","zhangsan");
HttpEntity<Object> entity = new HttpEntity<Object>(map, headers);

RestTemplate restTemplate = new RestTemplate();
ResponseEntity<String> exchange = restTemplate.exchange(path, HttpMethod.POST, entity, String.class);
```

