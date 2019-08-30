一.tomcat修改https访问

1>可以到阿里云购买免费版SLL证书并下载(有效期一年)

https://common-buy.aliyun.com/?spm=5176.2020520163.cas.4.1fc9Az7GAz7GIJ&commodityCode=cas#/buy

2>修改tomcat配置文件,server.xml,

 (1)增加如下配置

```
 <Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol" SSLEnabled="true"
              maxThreads="150" scheme="https" secure="true"
              clientAuth="false" sslProtocol="TLS"
       keystoreFile="conf/cert/2640918_www.ztuo.cn.pfx"
       keystoreType="PKCS12"
       keystorePass="证书密码" />
 ```
 
 注意:keystoreFile证书路径,keystorePass证书密码

 (2)找到80端口,或者其他访问的端口配置

 修改其中配置项 redirectPort为443

3>修改tomcat配置文件,web.xml
在<web-app>标签最后增加如下配置

```
 <security-constraint>
    <web-resource-collection>
        <web-resource-name>SSL</web-resource-name>
        <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <user-data-constraint>
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>
  </security-constraint>
```

4>重启tomcat

二.nginx修改https访问

修改配置文件 nginx.conf
```
server {
    listen 443;
    server_name localhost;
    ssl on;
    root html;
    index index.html index.htm;
    #证书路径
    ssl_certificate   cert/2133242_aidelove.com.pem;
    #证书路径
    ssl_certificate_key  cert/2133242_aidelove.com.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
        root html;
        index index.html index.htm;
    }
    
            #静态资源
    location ^~ /h5/ {
            alias /app/ztuo/h5/;
    }
    location /uc {
            client_max_body_size    5m;
            proxy_pass http://127.0.0.1:6002;
            proxy_set_header Host $host;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header X-Real-IP $remote_addr;
        }
}
```


重启ng
