## 1 代理访问

> es开启xpack后，kibana会弹出登录页面，web项目嵌套时，使用nginx代理kibana地址，实现自动登录

```conf
server {
        listen       80;
        server_name  localhost;
        location / {
            # kibana  ip:port
			proxy_pass http://127.0.0.1:5601;
			# username:password
			proxy_set_header  Authorization "Basic ZWxhc3RpYzoxMjM0NTY=";
			# 带参访问kibana的ip
			proxy_set_header  Host $host;
            proxy_set_header  X-Real-IP $remote_addr;
            proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
        }
# .....
}
```

> Authorization的值
>
> linux中:  echo elastic:PleaseChangeMe | base64       ZWxhc3RpYzoxMjM0NTY=
>
> 或者cmd使用  curl --user elastic:123456 -v "http://127.0.0.1:11111"   es的ip:port查看