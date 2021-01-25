## httpClientUtil

> ```xml
> <dependency>    
> 	<groupId>org.apache.httpcomponents</groupId>    
>     <artifactId>httpclient</artifactId>    
>     <version>4.5.13</version>
> </dependency>
> ```

```java
package com.baidu.httpdemo.util;

import org.apache.http.HttpResponse;
import org.apache.http.NameValuePair;
import org.apache.http.client.HttpClient;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.util.EntityUtils;

import javax.annotation.PostConstruct;
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;

public class HttpClientDemo {
    private static final HttpClientDemo CLIENT_DEMO = new HttpClientDemo();
    private static final String GET ="GET";
    private static final String POST ="POST";

    private static final RequestConfig CONFIG =
            RequestConfig.custom()
                    .setSocketTimeout(60000) // 响应
                    .setConnectTimeout(60000)// 从池中连接
                    .setConnectionRequestTimeout(60000)// 连接远端
                    .build();
    private static PoolingHttpClientConnectionManager connectPool;

    private HttpClientDemo() {
    }

    public static HttpClientDemo getInstance() {
        return CLIENT_DEMO;
    }

    @PostConstruct
    private void postConstruct() {
        connectPool = new PoolingHttpClientConnectionManager();
        connectPool.setMaxTotal(10); // 最大连接数
        connectPool.setDefaultMaxPerRoute(10); // 最大路由数
    }

    private CloseableHttpClient getClient() {
        return HttpClients.custom().setConnectionManager(connectPool).setDefaultRequestConfig(CONFIG).build();
    }

    public String postRequest(String url, String json) {
        if (null == json || "".equals(json)) {
            throw new RuntimeException(url + "-json数据不能为空");
        }

        log.debug("url: {}", url);
        HttpPost post = new HttpPost(url);
        post.addHeader("Content-Type", "application/json");
        post.addHeader("Accept", "application/json");
        post.setEntity(new StringEntity(json, StandardCharsets.UTF_8));

        try {
            HttpClient client = getClient();
            HttpResponse response = client.execute(post);
            int code = response.getStatusLine().getStatusCode();
            log.info("code -> {}", code);

            if (code >= 400) {
                throw new RuntimeException(EntityUtils.toString(response.getEntity()));
            }
            return EntityUtils.toString(response.getEntity());
        } catch (Exception e) {
            throw new RuntimeException("path:" + url + " post请求异常:" + e.getMessage(), e);
        }
    }


    public String getRequest(String path) {
        HttpGet get = new HttpGet(path);
        HttpClient client = getClient();

        try {
            HttpResponse response = client.execute(get);
            int code = response.getStatusLine().getStatusCode();
            if (code >= 400)
                throw new RuntimeException("errCode:" + code);
            return EntityUtils.toString(response.getEntity());
        } catch (Exception e) {
            throw new RuntimeException("path:" + path + " get请求异常:" + e.getMessage(), e);
        }
    }


    /**
     * 带参get
     *
     * @param path
     * @param parametersBody
     * @return
     * @throws Exception
     */
    public String getRequest(String path, List<NameValuePair> parametersBody) throws Exception {
//        List<NameValuePair> parametersBody = new ArrayList();
//        parametersBody.add(new BasicNameValuePair("userId", "admin"));
        URIBuilder uriBuilder = new URIBuilder(path);
        uriBuilder.setParameters(parametersBody);
        HttpGet get = new HttpGet(uriBuilder.build());
        HttpClient client = getClient();

        try {
            HttpResponse response = client.execute(get);
            int code = response.getStatusLine().getStatusCode();
            if (code >= 400)
                throw new RuntimeException((new StringBuilder()).append("Could not access protected resource. Server returned http code: ").append(code).toString());
            return EntityUtils.toString(response.getEntity());
        } catch (Exception e) {
            throw new RuntimeException("postRequest -- IO error!", e);
        }
    }

    public List<String[]> listUrls(String path) {
        path = null == path || "".equals(path) ? "/rest.txt" : path;
        String filePath = HttpClientDemo.class.getResource(path).getPath();
        List<String[]> urls = new ArrayList<>();

        try (InputStream in = new FileInputStream(new File(filePath));
             InputStreamReader inputStreamReader = new InputStreamReader(in);
             BufferedReader reader = new BufferedReader(inputStreamReader)) {

            String readLine = null;
            while ((readLine = reader.readLine()) != null && !"".equals(readLine)) {
                String[] split = readLine.split(",");
                urls.add(split);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        if (null == urls || urls.size() == 0) {
            throw new RuntimeException("服务组合配置文件无数据");
        }
        return urls;
    }

    public static void main(String[] args) {
        List<String[]> urls = HttpClientDemo.getInstance().listUrls("");

        String json = "";
        for (String[] url : urls) {
            String path = url[0];
            String method = url[1];

            if (GET.equals(method)) {
                json = HttpClientDemo.getInstance().getRequest(path);
            } else if (POST.equals(method)) {
                json = HttpClientDemo.getInstance().postRequest(path, json);
            }
        }

        System.out.println(json);
    }
}
```

