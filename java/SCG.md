

## 接出

> 总前	<-	gateway	<-	web  

```java
import org.reactivestreams.Publisher;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.cloud.gateway.filter.rewrite.CachedBodyOutputMessage;
import org.springframework.cloud.gateway.support.BodyInserterContext;
import org.springframework.cloud.gateway.support.DefaultServerRequest;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.core.io.buffer.DataBufferFactory;
import org.springframework.core.io.buffer.DataBufferUtils;
import org.springframework.core.io.buffer.DefaultDataBufferFactory;
import org.springframework.http.HttpHeaders;
import org.springframework.http.ReactiveHttpOutputMessage;
import org.springframework.http.server.reactive.ServerHttpRequestDecorator;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.http.server.reactive.ServerHttpResponseDecorator;
import org.springframework.web.reactive.function.BodyInserter;
import org.springframework.web.reactive.function.BodyInserters;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.server.ServerWebExchange;

import com.cebbank.poin.pte.annotation.Constant;
import com.cebbank.poin.pte.annotation.core.param.PTESystemParameters;
import com.cebbank.poin.p...
import reactor.core.Publisher.Flux;
import reactor.core.publisher.Nono;

public class PoinAddReqMacGatewayFilterFactory extends AbstractGatewayFilterFactory<Object> {
    public static final Logger logger = LoggerFactory.getLogger(PoinAddReqMacGatewayFilterFactory.class);
    private static final String SIGNATURE = "signature";
    String macValue = "";
    
    @Override
    public GatewayFilter apply(Object config) {
        return new CheckMacFilter();
    }
    
    class CheckMacFilter implements GatewayFilter, Ordered{
        @Override
        public int getOrder() {
            return -10;
        }
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            ServerRequest serverRequest = new DefaultServerRequest(exchange);
            Mono<DataBuffer> modifiedBody = serverRequest.bodyToMono(DataBuffer.class).flatMap(o->addMAC(exchange,o));
            BodyInserter<Mono<DataBuffer>,ReactiveHttpOutputMessage> bodyInserter = BodyInserters.fromPublisher(modifiedBody,DataBuffer.class);
            HttpHeaders headers = new HttpHeaders();
            CachedBodyOutputMessage outputMessage = new CachedBodyOutputMessage(exchange,headers);
            return bodyInserter.insert(outputMessage, new BodyInserterContext()).then(Mono.defer()->{
            	ServerHttpRequestDecorator decorator = new ServerHttpRequestDecorator(exchange.getRequest()) {
                    @Override
                    public HttpHeaders getHeaders() {
                        headers.putAll(super.getHeaders());
                        
                        // 检查是否加密
                        String macFlag = exchange.getRequest().getHeaders().getFirst(PkiForCebASYM.EDSP_MAC_FLAG);
                        if(!Boolean.valueOf(macFlag)){
                            return headers;
                        }
                        // 获取非对称加密的flag
                        String edspFlag = PkiForCebASYM.getInstance().getValueFromPte(PkiForCebASYM.ESSCAPI_EDSP);
                        // 若EsscAPI.edsp=ASYM,则使用非对称加密
                        if (PkiForCebASYM.ESSCAPI_EDSP_VALUE.equals(edspFlag)){
                            String signture = ...
                            String pkName = PkiFroCebASYM.getInstance().getValueFromPte(PkiForCebASYM.ESSCAPI_PKNAME);
                            headers.set(PkiForCebASYM.EDSP_MAC_KEY, pkName);
                            headers.set(PkiForCebASYM.EDSP_MAC_VALUE,signature);
                        }else {
                            // 使用国密
                            ...
                        }
                        return headers;
                    }
                    @Override
                    public Flux<DataBuffer> getBody() {
                        return outputMessage.getBody();
                    }
                };
                // ==
               ServerHttpResponseDecorator responseDecorator = new ServerHttpResponseDecorator(exchange.getResponse()){
                   DataBufferFactory bufFactory = exchange.getResponse().bufferFactory();
                   ServerHttpResponse response = exchange.getResponse();
                   
                   @Override
                   public Mono<Void> writeWith(Publisher<? extends DataBuffer>) body;
                   return super.writeWith(fluxBody.buffer().map(buf -> {
                       DataFufferFactory dataBufferFactory = new DataultDataBufferFactory();
                       DataBuffer join = dataBufferFactory.join(buf);
                       byte[] content = new byte[join.readableByteCount()];
                       join.read(content);
                       DataBufferUtils.release(join);
                       String responseMacVal = response.getHeaders().getFirst("EDSP_MAC_Value");
                       String reqkeynameStr = response.getHeaders().getFirst("EDSP_MAC_Key");
                       
                       // 若签名为空，则不验
                       if (null==responseMacVla || "".equals(responseMacVla)){
                           return bufFactory.wrap(content);
                       }
                       
                       // 获取非对称加密的flag
                       String edspFlag = PkiForCebASYM.getInstance.get...
                       // 若。。则使用非对称验签
                       if (...){
                           // 非对称验签
                       }else {
                           // 对称验签
                       }
                       logger.info("验mac成功")
                       return bufFactory.wrap(content);
                   }));
               }
                return super.writeWith(body);
            }
             @Override
             public Mono<Void> writeAndFlushWith(Publisher<? extends Publisher<? extends Publisher>> body){
                 return writeWith(Flux.from(body).flatMapSequential(p -> p));
             } 
                                                                                      };  
      return chain.filter(exchange.mutate().request(decoreator).response(responseDecorator).build());
        }));
    }
        
        private Mono<DataBuffer> addMac(ServerWebExchange exchange, DataBuffer buf){
            DataBufferFactory bufFactory = exchange.getResponse().bufferFactory();
            byte[] content = new byte[buf.readableByteCount()];
            buf.read(content);
            DataBufferUtils.release(buf);
            
            // 获取是否加密flag
            String flag = exchange.getRequest().getHeaders().getFirst("EDSP-MAC-Flag");
            if (!Boolean.valueOf(flag)){
                return Mono.just(bufFactory.wrap(content));
            }
            // 判断是非对称，则非对称生成签名
            
            return Mono.just(bufFactory.wrap(content));
        }
}
```

> 总前 -> 网关 -> web  接入

```java
public class PoinCheckMac2GatewayFilterFactory extends AbstractGatewayFilterFactory<Object>{
    @Override
    public GatewayFilter apply(Object config){
        return new CheckMacFilter();
    }
    
    class CheckMacFilter implements GatewayFilter, Ordered{
        @Override
        public int getOrder() {
            return -10;
        }
        
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain){
            DataBufferFactory bufFactory = exchange.getResponse().bufferFactory();
            
            ServerHttpResponseDecorator responseDecorator = new ServerHttpResponseDecorator(exchange.getResponse()){
                @Override
                public Mono<Void> writeWith(Publisher<? extends DataBuffer> body){
                    if (body instanceof Flux){
                        Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>)body;
                        return super.writeWith(fluxBody.map(buf ->{
                            byte[] content = new byte[buf.readableByteCount()];
                            buf.read(content);
                            DataBufferUtils.release(buf);
                            String macFlag = (String)exchange.getAttributes().get("macFlag");
                            // 若接入的请求中有签名则响应时也生成签名，否则不生成
                            if (null == macFlag || "".equals()){
                                return bufFactory.wrap(content);
                            }
                            
                            // 获取非常加密的flag，
                            // 非对称生成签名
                            // 对称生成签名
                            return bufFactory.wrap(content);
                        }));
                    }
                    return super.writeWith(body);
                }
                @Override
                public Mono<Void> writeAndFlushWith(Publisher<? extends Publisher<? extends DataBuffer>> body){
                    return writeWith(Flux.from(body).flatMapSequential(p->p));
                }
            };
            
            Flux<DataBuffer> body = exchange.getRequest().getBody();
            Flux<DataBuffer> map = body.map(buffer -> {
                DataBuffer buf = buffer.slice(0,buffer.readableByteCount());
                byte[] b = new byte[buf.readableByteCount()];
                buf.read(b);
                
                HttpHeaders headers = new HttpHeaders();
                headers.putAll(exchange.getRequest().getHeaders());
                
                List<String> list = headers.get("EDSP-MAC_Value");
                List<String> reqkeyname = headers.get("EDSP-MAC-Key");
                
                .. 验签
                return buffer;
            });
            
            // 重建request，response
            ServerHttpRequest mutateRequest = new ServerHttpRequestDecorator(exchange.getRequest()){
              @Override
              public Flux<DataBuffer> getBody() {
                  return map;
              }
            };
            return chain.filter(exchange.mutate().request(mutatedRequest).response(responseDecorator).build());
        }
    }
}
```



##  精简版

> SpringBoot 2.3.7
>
> gateway Hoxton.SR9

```java
package com.baidu.gateway.filter;

import org.reactivestreams.Publisher;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.core.io.buffer.DataBufferUtils;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpRequestDecorator;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.http.server.reactive.ServerHttpResponseDecorator;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

/**
 * @author gengwei  接出,验签，获取body，前提是数据为json
 * 使用request和response中body进行加签名和验签，并对body进行重写
 * 接入也写一个filter，加签和验签与接出的位置相反
 */
@Component
public class AddSignatureGatewayFilterFactory extends AbstractGatewayFilterFactory<Object> {

    @Override
    public GatewayFilter apply(Object config) {
        return new FromMeGatewayFilter();
    }

    class FromMeGatewayFilter implements GatewayFilter, Ordered {
        @Override
        public int getOrder() {
            return -2;
        }

        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            // 重建response
            ServerHttpResponse responseDecorator = new ServerHttpResponseDecorator(exchange.getResponse()) {
                @Override
                public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
                    if (body instanceof Flux) {
                        Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>) body;
                        return super.writeWith(fluxBody.map(dataBuffer -> {
                            byte[] content = new byte[dataBuffer.readableByteCount()];
                            dataBuffer.read(content);
                            DataBufferUtils.release(dataBuffer);

                            // TODO 设置签名
                            System.out.println("response body:"+new String(content));
                            //this.getHeaders().set("headName","headValue");
                            //String headerName = exchange.getResponse().getHeaders().getFirst("headerName");
                            return exchange.getResponse().bufferFactory().wrap(content);
                        }));
                    }
                    return super.writeWith(body);
                }
            };


            // 重建request
            ServerHttpRequest requestDecorator = new ServerHttpRequestDecorator(exchange.getRequest()){
                @Override
                public Flux<DataBuffer> getBody() {
                    Flux<DataBuffer> map = exchange.getRequest().getBody().map(dataBuffer -> {
                        DataBuffer buf = dataBuffer.slice(0, dataBuffer.readableByteCount());
                        byte[] b = new byte[buf.readableByteCount()];
                        buf.read(b);
                        // TODO 验签...
                        System.out.println("request body:"+new String(b));
                        //this.getHeaders().set("headName","headValue");
                        //String headerName = exchange.getResponse().getHeaders().getFirst("headerName");
                        return dataBuffer;
                    });
                    return map;
                }
            };
            return chain.filter(exchange.mutate().request(requestDecorator).response(responseDecorator).build());
        }
    }
}
```

```yml
server:
  port: 9000
spring:
  cloud:
    gateway:
      routes:
        - id: test_route
          uri: http://127.0.0.1:9999
          predicates:
            - Path=/clientTest/**
          filters:
            # 若是AbstractGatewayFilterFactory的子类，则只写前缀即可
            - AddSignature
```

```java
@RestController
public class ClientController {

    @GetMapping("/clientTest")
    public Map<String, Object> clientTest(@RequestBody Map<String, Object> map) {
		//System.out.println("client端获取到的参数为："+map);
        map.put("result", "我收到了");
        return map;
    }
}
```





















