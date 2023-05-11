# nginx-r-proxy

Reverse proxy:

Step 1 — Installing Nginx
sudo apt update
sudo apt install nginx
If not installing then add following:
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm


sudo ufw allow 'Nginx HTTP'

We might need the below cmds but not sure:
# systemctl enable nginx
# systemctl start nginx
# firewall-cmd --permanent --zone=public --add-service=http
# firewall-cmd --permanent  --zone=public --add-service=https
# firewall-cmd --reload

If not starting then run
sudo fuser -k 80/tcp
sudo fuser -k 443/tcp


Now you can verify that Nginx is running:
systemctl status nginx

Step 2 — Configuring your Server Block
This is the base config file:   /etc/nginx/nginx.conf
Create the following file and include in above file.

sudo nano /etc/nginx/sites-available/{your_domain}

/etc/nginx/sites-available/your_domain

server {
    listen 80;
    listen [::]:80;

    server_name your_domain www.your_domain;
        
    location / {
        proxy_pass app_server_address;
        include proxy_params;
    }
}

My last example:
server {
    listen 80;
    listen [::]:80;

    server_name tijaricloud.com www.tijaricloud.com;

    location / {
        proxy_pass https://0.0.0.0:3006/;
        include proxy_params;
    }
    location /admin/ {
        proxy_pass https://0.0.0.0:4300/;
        include proxy_params;
    }
    location /merchant/ {
        proxy_pass https://0.0.0.0:4200/;
        include proxy_params;
    }
}
Then include this in main file



Save and exit, with nano you can do this by hitting CTRL+O then CTRL+X.

In this file add following:     /etc/nginx/proxy_params

proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
Next, enable this configuration file by creating a link from it to the sites-enabled directory that Nginx reads at startup:

sudo ln -s /etc/nginx/sites-available/{your_domain}/etc/nginx/sites-enabled/
You can now test your configuration file for syntax errors:
sudo nginx -t

With no problems reported, restart Nginx to apply your changes:
sudo systemctl restart nginx

To view errors go to /var/log/nginx

If 
(13: Permission denied) while connecting to upstream:[nginx]
Disclaimer
Make sure there are no security implications for your use-case before running this.
Answer
I had a similar issue getting Fedora 20, Nginx, Node.js, and Ghost (blog) to work. It turns out my issue was due to SELinux.
This should solve the problem:
setsebool -P httpd_can_network_connect 1

In frontend project we need to give base url if it is not on root route

Like in angular we give in build command:

ng build --prod --base-href yoursubfolder


Or set it in the index.html of your build like:

<head> <base href="/yoursubfolder/"> </head>

