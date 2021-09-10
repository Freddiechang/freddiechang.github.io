# 设置NPS   
按照官网文档安装并配置[nps](https://ehang-io.github.io/nps/#/?id=nps)。首先配置好SSH转发（TCP隧道到本地22端口），然后设置域名解析到本地http服务端口:   
![](https://assets.freddie.party/nps_url_parse.png)   
设置好nps的http端口（nps通过这个端口提供本地http服务）。用到的端口及说明：   
```
24615: web管理界面
24616: 客户端与nps连接端口
24617: 转发到客户端SSH的端口
24618: nps的http服务端口
24619: VNC服务转发端口
```
在服务器nginx配置中添加端口转发：   
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
为了方便直接通过网站访问下载的内容，可以在Nginx默认目录html下设置软链接到下载文件夹。此时因为SELinux默认阻止Nginx访问默认文件夹以外的文件，直接通过网站访问会出现403错误，同时Nginx日志
会出现   
```
13: Permission denied.
```   
错误。参考[这个问题](https://stackoverflow.com/questions/22586166/why-does-nginx-return-a-403-even-though-all-permissions-are-set-properly)，先暂时关闭SELinux以确定问题：   
```
setenforce Permissive
```   
然后重启Nginx，如果问题解决，重新开启SELinux：   
```
setenforce 1
```   
然后给目标目录加上一个label：   
```
chcon -Rt httpd_sys_content_t /path/to/www
```   
问题解决。
