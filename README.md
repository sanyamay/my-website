# my-website

# 1. Server Provisioning:

I created a Virtual Private Cloud (VPC). Following that, I created subnets and route tables, and then attached them to the VPC. Next, I configured firewall rules, specifically allowing inbound traffic for SSH, HTTP, and HTTPS services in the security group.

Upon completion of the VPC setup, I created an instance using an Ubuntu machine and named it "myhos." I then launched the instance and set the hostname using the following command, considering my domain name is "lkshmisakha.site"
hostnamectl set-hostname lkshmisakha.site

2. Website Configuration :

# First installed the nginx server by using the follwing commands
apt install nginx -y

# Install MySQL
sudo apt install -y mysql-server

# Secure MySQL installation
sudo mysql_secure_installation

# Wordpress installation
wget https://wordpress.org/latest.zip

# Unzip tool 
apt install unzip

# Create the directory
mkdir -p /usr/share/nginx

# Unziping file
unzip latest.zip -d /usr/share/nginx/

# For setting up WordPress, it is necessary to install some PHP extensions. I have installed the following PHP extensions:
apt install php-imagick php8.1-fpm php8.1-mbstring php8.1-bcmath php8.1-xml php8.1-mysql php8.1-common php8.1-gd php8.1-cli php8.1-curl php8.1-zip

# Moving files in the created directory 
mv /usr/share/nginx/wordpress /usr/share/nginx/lkshmisakha.site

# After that, navigate to your WordPress directory.
cd /usr/share/nginx/lkshmisakha.site/

# Copy the sample configuration file and rename it to wp-config.php.
cp wp-config-sample.php wp-config.php

# make sure only the file owner can read this file.
chmod 640 wp-config.php

# We also need to set the Nginx user (www-data) as the owner of the WordPress site
chown www-data:www-data /usr/share/nginx/lkshmisakha.site/ -R

# We will create the server block file in /etc/nginx/conf.d/ directory.
bash -c 'cat <<EOF > /etc/nginx/conf.d/lkshmisakha.site.conf
server {
  listen 80;
  listen [::]:80;
  server_name www.lkshmisakha.site lkshmisakha.site;
  root /usr/share/nginx/example.com/;
  index index.php index.html index.htm index.nginx-debian.html;

  error_log /var/log/nginx/wordpress.error;
  access_log /var/log/nginx/wordpress.access;

  location / {
    try_files $uri $uri/ /index.php;
  }

   location ~ ^/wp-json/ {
     rewrite ^/wp-json/(.*?)$ /?rest_route=/$1 last;
   }

  location ~* /wp-sitemap.*\.xml {
    try_files $uri $uri/ /index.php$is_args$args;
  }

  error_page 404 /404.html;
  error_page 500 502 503 504 /50x.html;

  client_max_body_size 20M;

  location = /50x.html {
    root /usr/share/nginx/html;
  }

  location ~ \.php$ {
    fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
    fastcgi_buffers 1024 4k;
    fastcgi_buffer_size 128k;
  }

  #enable gzip compression
  gzip on;
  gzip_vary on;
  gzip_min_length 1000;
  gzip_comp_level 5;
  gzip_types application/json text/css application/x-javascript application/javascript image/svg+xml;
  gzip_proxied any;

  # A long browser cache lifetime can speed up repeat visits to your page
  location ~* \.(jpg|jpeg|gif|png|webp|svg|woff|woff2|ttf|css|js|ico|xml)$ {
       access_log        off;
       log_not_found     off;
       expires           360d;
  }

  # disable access to hidden files 
  location ~ /\.ht {
      access_log off;
      log_not_found off;
      deny all;
  }
}
EOF'

# Test Nginx configurations.
nginx -t


# Start and enable Nginx
 systemctl start nginx
 systemctl enable nginx

# Grant executable permissions to admin.php
chmod 755 /usr/share/nginx/lkshmisakha.site/wp-admmin/admin.php

# Display Nginx status
systemctl status nginx

# For securing the website, I used the following command:
apt install certbot python3-certbot-nginx

# After installing Certbot, use the following command to secure my site:
certbot --nginx -d  lkshmisakha.site

Please find the below URL 
URL :- https://lkshmisakha.site
