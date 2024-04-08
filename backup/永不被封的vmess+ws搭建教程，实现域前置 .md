1、在云服务器上安装 V2Ray ：
sudo apt update
sudo apt install curl unzip
curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh
chmod +x install-release.sh
sudo bash install-release.sh
sudo nano /usr/local/etc/v2ray/config.json

2、配置V2ray config.json代码
{
"log": {
"loglevel": "warning",
"access": "/var/log/v2ray/access.log",
"error": "/var/log/v2ray/error.log"
},
"inbounds": [
{
"port": 10000,
"listen": "127.0.0.1",
"protocol": "vmess",
"settings": {
"clients": [
{
"id": "改成自己的uuid",
"alterId": 64
}
]
},
"streamSettings": {
"network": "ws",
"wsSettings": {
"path": "/ray"
}
}
}
],
"outbounds": [
{
"protocol": "freedom",
"settings": {}
}
]
}
3、关闭防火墙
ufw disable
4、启动 V2Ray 服务
sudo systemctl start v2ray
检查 V2Ray 服务状态
systemctl status v2ray
设置 V2Ray 自动启动
sudo systemctl enable v2ray
再次检查 V2Ray 服务状态
systemctl status v2ray

5、安装Nginx作为反向代理：

首先云将服务器ip地址指向域名，给域名添加记录，使用cloudclfare解析域名

安装 Nginx
sudo apt install nginx
创建并配置 Nginx 虚拟主机文件
sudo nano /etc/nginx/conf.d/v2ray.conf
配置代码
server {
listen 80;
server_name 自己的域名;

index index.html;
root /usr/share/nginx/html/;

access_log /var/log/nginx/v2ray.access;
error_log /var/log/nginx/v2ray.error;

location /ray {
if ($http_upgrade != "websocket") {
return 404;
}
proxy_redirect off;
proxy_pass http://127.0.0.1:10000/;
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
}

重启 Nginx 服务
sudo systemctl restart nginx
检查nginx是否已经成功启动
sudo systemctl status nginx
设置Nginx 自动启动
sudo systemctl enable nginx
再次检查Nginx状态
systemctl status nginx