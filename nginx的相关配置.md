1.编译参数说明
```
编译参数说明：
--prefix=/usr/local/nginx                   #指定安装路径
--with-http_stub_status_module      #获取nginx自上次启动以来的工作状态
--with-http_ssl_module                    #支持https请求，需已安装openssl
--with-http_gzip_static_module        #静态预压缩文件
--with-http_concat_module              #用于合并多个文件在一个响应报文中
--with-jemalloc                                 #Tengine链接jemalloc库，使用jemalloc来分配和释放内存
--with-http_v2_module                    #提供对HTTP/2的支持
--with-http_secure_link_module      #保护服务器文件不被任意下载盗用
```
2.安全防护配置文件
```
#禁止IP直接访问
server {
listen 80 default;
server_name _;
return 403;
}
server {
listen 443 default;
server_name _;
ssl_certificate /usr/local/nginx/certs/cardniu.crt;
ssl_certificate_key /usr/local/nginx/certs/cardniu.key;
return 403;
}

#防止DDOS、CC攻击
map $http_x_forwarded_for $clientRealIp {
"" $remote_addr;
~^(?P<firstAddr>[0-9\.]+),?.*$ $firstAddr;
}

geo $clientRealIp $whiteiplist {
default 1;
172.22.23.241 1;
172.22.23.251 0;
192.168.31.236 0;
172.22.25.0/24 0;
}

map $whiteiplist $limit {
1 $clientRealIp;
0 "";
}

limit_conn_zone $limit zone=TotalConnLimitZone:20m ;
limit_conn TotalConnLimitZone 50;
limit_conn_log_level notice;

limit_req_zone $limit zone=ConnLimitZone:20m rate=20r/s;
limit_req zone=ConnLimitZone burst=10 nodelay;
limit_req_log_level notice;
```

3.[文档参考链接](http://blog.51cto.com/hzcsky/1625354)
