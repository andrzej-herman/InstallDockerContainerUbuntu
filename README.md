# Tutorial
Tutorial how to install docker container with Let's Encrypt SSL Certificate on Ubuntu Server 18.04

At first set A record of api domain in domain provider to IP address of linux server.
Login to Ubuntu Server using Putty (SSH on port: 22) 
Then follow the steps:

# Steps

## 1. Install docker

### Update apt repository
```
sudo apt-get update
```
### Remove current version if exists
```
sudo apt-get remove docker docker-engine docker.io
```
### Start docker installation proccess
```
sudo apt install docker.io
```
### Start docker
```
sudo systemctl start docker
```
### Enable docker
```
sudo systemctl enable docker
```
### Check docker version
```
sudo docker --version
```
### Display running containers
```
sudo docker ps
```
### Display all containers
```
sudo docker ps -a
```


## 2. Install API container

### Login to docker hub
```
sudo docker login
```
### Run API container on random port
```
sudo docker run --name <local_container_name> -P -d <docker_hub_login>/<docker_hub_container_name>
```
### Check number of port of running api container (remember it)
```
sudo docker ps 
```


## 3. Install nginx container to only to get certificate

### Run nginx container on random port
```
sudo docker run --name nginx_cert -P -d nginx
```
### Check number of port of running nginx container
```
sudo docker ps 
```


## 4. Install Let's Encrypt Certbot

### Install certbot repository
```
sudo add-apt-repository ppa:certbot/certbot
```
### Install certbot for nginx
```
sudo apt install python-certbot-nginx 
```


## 5. Get SSL Certificate

### Run command for domain certificate
```
sudo certbot --nginx -d <api_domain>
```
Then give email address
Agree to terms (A)
No to get emails (N)
Important !!! Choose option nr 1 (No redirect)

### Get local path of certificate files (remember them)
```
sudo certbot certificates 
```


## 6. Stop nginx_cert container

### Check containerID of nginx_cert
```
sudo docker ps
```
### Run stop command
```
sudo docker stop <containerID>
```


## 7. Prepare additional nginx config file

### Create a directory /var/nginx/conf.d
```
cd /var
sudo mkdir nginx
cd nginx
sudo mkdir conf.d
cd conf.d
```
### Create config file (<app_name>.conf) and get write privillages
```
sudo touch <app_name>.conf
ls -l
sudo chmod 771 slimapi.conf
```
### Open file using nano
```
sudo nano <app_name>.conf
```
### Put this content
```
server  {
  listen  80;
  server_name  <domain_name>;
  return 301 https://$server_name/$1;
}
server  {
  listen 443;
  server_name  <domain_name>;
  ssl  on;
  ssl_certificate  /etc/letsencrypt/live/<domain_name>/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/<domain_name>/privkey.pem;
  ssl_session_timeout  5m;
  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
  server_tokens off;
  access_log /dev/null;
  error_log /dev/stderr;
  location  / {
        proxy_pass http://<domain_name>:<port_api_container_is_running>;
        proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_hide_header X-Powered-By;
  }
}
```


## 8. Run nginx_<app_name> container on random port

### Run command (modify the container config)
```
sudo docker run --name nginx_slimapi -v /var/nginx/conf.d:/etc/nginx/conf.d -P -d nginx
```



