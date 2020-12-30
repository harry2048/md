## jwt权限校验

> 非严格的权限校验，无状态
>
> 先获取jwt，访问时header中带着jwt，验证jwt通过后继续执行。可以设置失效时间
>
> 被截取jwt后，无法停止黑客。将jwt存入redis，可以做强制下线

* jwt和token的区别，jwt是无状态的，不需要访问数据库。token每次都要访问数据库来验证

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.7.0</version>
</dependency>
```

```java
package com.online.taxi.common.util;

import java.util.Date;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.Jwts;

/**
 * subject：可以使用用户的唯一标识创建jwt，将jwt返回给客户端，客户端其他请求带有jwt
 */
public class JwtUtil {
    /**
     * 密钥，仅服务端存储
     */
    private static String secret = "ko346134h_we]rg3in_yip1!";

    /**
     *
     * @param subject   可以是用户的唯一标识
     * @param issueDate 签发时间
     * @return
     */
    public static String createToken(String subject, Date issueDate) {
        String compactJws = Jwts.builder()
                .setSubject(subject)
                .setIssuedAt(issueDate)
                .setExpiration(new Date(System.currentTimeMillis()+10000))
                .signWith(io.jsonwebtoken.SignatureAlgorithm.HS512, secret)
                .compact();
        return compactJws;

    }

    /**
     * 解密 jwt
     * @param token
     * @return
     * @throws Exception
     */
    public static String parseToken(String token) {
        try {
            Claims claims = Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
            if (claims != null){
                return claims.getSubject();
            }
        }catch (ExpiredJwtException e){
            e.printStackTrace();
            System.out.println("jwt过期了");
        }

        return "";
    }

    public static void main(String[] args) {
        String subject = "gw";
        String token = createToken(subject,new Date());
        System.out.println(token);
        try {
//			Thread.sleep(10010);
        	Thread.sleep(100);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
        System.out.println("原始值："+parseToken(token));

    }

}
```

