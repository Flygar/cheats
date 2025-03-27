### 参考文档
Nginx on Debian 11
```sh
## 文档链接
## How To Install Nginx on Debian 11
https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-debian-11

## Step 1 – Installing Nginx
sudo apt update && sudo apt install nginx

## Step 2 – Adjusting the Firewall
sudo ufw app list

## Nginx Full: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
## Nginx HTTP: This profile opens only port 80 (normal, unencrypted web traffic)
## Nginx HTTPS: This profile opens only port 443 (TLS/SSL encrypted traffic)
sudo ufw allow 'Nginx Full' # sudo ufw allow 'Nginx HTTP'
sudo ufw status

## Step 3 – Checking your Web Server
systemctl status nginx
## You can access the default Nginx landing page to confirm that the software is running properly by navigating to your server’s IP address.
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
## http://your_server_ip

## Step 4 – Managing the Nginx Process
sudo systemctl stop nginx
sudo systemctl start nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl disable nginx
## To re-enable the service to start up at boot
sudo systemctl enable nginx


## Step 5 – Setting Up Server Blocks (Optional)
## 默认目录: /var/www/html，最好不动，创建新的
## <your_domain>: api.flygar.org 
sudo mkdir -p /var/www/your_domain/html
echo $USER
sudo chown -R $USER:$USER /var/www/your_domain/html
## 保证权限正确
sudo chmod -R 755 /var/www/your_domain
## 编辑index写入下面的html内容
# sudo vim /var/www/your_domain/html/index.html
# <html>
#     <head>
#         <title>Welcome to your_domain</title>
#     </head>
#     <body>
#         <h1>Success! Your Nginx server is successfully configured for <em>your_domain</em>. </h1>
# <p>This is a sample page.</p>
#     </body>
# </html>
## serve this content, you must create a server block with the correct directives that point to your custom web root. Instead of modifying the default configuration file directly, make a new one
sudo vim /etc/nginx/sites-available/your_domain
## 下面这段server的意思：访问your_domain的80端口会代理到localhost的3000端口
# server {
#         listen 80;
#         listen [::]:80;

#         server_name your_domain;

#         location / {
#                  proxy_pass  http://localhost:3000;
#         }
# }
## Next, enable this server block by creating a symbolic link to your custom configuration file inside the sites-enabled directory
sudo ln -s /etc/nginx/sites-available/your_domain /etc/nginx/sites-enabled/

## To avoid a possible hash bucket memory problem
sudo vim /etc/nginx/nginx.conf
## Find the server_names_hash_bucket_size directive and remove the # symbol to uncomment the line
# ...
# http {
#     ...
#     server_names_hash_bucket_size 64;
#     ...
# }
# ...


## syntax errors check
sudo nginx -t

## restart Nginx to enable your changes
sudo systemctl restart nginx
```

### 配合 certbot 一起食用
```sh
# 链接
https://certbot.eff.org/instructions?ws=nginx&os=snap&tab=standard
```
