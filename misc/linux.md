# debian
## 查看ip
`
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
`

# ngnix 参考文档：https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-debian-11
## You can check with the systemd init system to make sure the service is running
sudo apt update && sudo apt install nginx
## Adjusting the Firewall
sudo ufw app list
## Nginx Full: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
## Nginx HTTP: This profile opens only port 80 (normal, unencrypted web traffic)
## Nginx HTTPS: This profile opens only port 443 (TLS/SSL encrypted traffic)
sudo ufw allow 'Nginx HTTP'
sudo ufw status

systemctl status nginx
sudo systemctl stop nginx
sudo systemctl start nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl disable nginx
# To re-enable the service to start up at boot
sudo systemctl enable nginx


#Setting Up Server Blocks (Optional)
sudo mkdir -p /var/www/<your_domain>/html
