## 兼容IE的返回信息

将返回信息直接输出到页面中

```java
public Response returnMsg(HttpServletResponse response, String msg) {
    response.setContentType("text/html;charset=utf-8");
    try(PrintWriter writer = response.getWriter();){
        writer.write(msg);
        writer.flush();  
        return Response.ok().build();
    }catch(Exception e){
        throw new RuntimeException(e.getMessage(),e);
    }
}
```

