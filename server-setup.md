# 设置NPS   
按照官网文档安装并配置[nps](https://ehang-io.github.io/nps/#/?id=nps)。首先配置好SSH转发（TCP隧道到本地22端口），然后设置域名解析到本地http服务端口:   
![](https://assets.freddieonfire.tk/nps_url_parse.png)   
设置好nps的http端口（nps通过这个端口提供本地http服务）。在服务器nginx配置中添加端口转发：   
```
    server {
        listen 80;
        server_name #URL;
        location / {
            proxy_set_header Host  $http_host;
            proxy_pass http://127.0.0.1:#PORT;
        }
    }
```   
其中#URL为nps中设置的域名解析域名，#PORT为本地http服务的端口。现在访问下设置好的域名，出现错误 502 bad gateway。首先检查nginx日志：   
```
2021/06/15 19:36:49 [crit] 3638#0: *25 connect() to 127.0.0.1:#PORT failed (13: Permission denied) 
while connecting to upstream, client: ******, server: #URL, request: "GET / HTTP/1.1", 
upstream: "http://127.0.0.1:#PORT/", host: "#URL"
```   
再检查nps日志，发现没有来自#URL的访问记录和http请求，说明问题出在nginx和nps的连接上。参考[这个问题](https://stackoverflow.com/questions/23948527/13-permission-denied-while-connecting-to-upstreamnginx)，
可以知道是因为SELinux阻止了连接。sudo运行以下命令即可：   
```
sudo setsebool -P httpd_can_network_connect 1
```  
至此设置完成，可以正常访问。
