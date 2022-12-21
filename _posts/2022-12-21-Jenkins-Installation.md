---
title: "[Devops] Jenkins Installation"
date: 2022-12-21
categories: [ngoprek, server, cloud, kubernetes, jenkins, devops, ]
tags:
  - Jekyll
  - update
---

## Guide ## Prerequisites
## Update Packages

```
$ sudo apt-get update
$ sudo apt-get install -y \
ca-certificates \
curl \
gnupg \
lsb-release
```

## Install JavaDK11

```
sudo apt search openjdk
sudo apt install openjdk-11-jdk
java -version
```

## Install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins -y
```

## Generate Initial Password Jenkins

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

## Init Credential

```
Username : admin
Password : initialAdminPassowrd
```

## Access

`http://your_ip:8080`
![Dashboard1](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/1-uijenkins.jpg)

## Reverse Proxy to 443
- Generate SSL

```
apt install -y certbot
certbot certonly
```

- Install Nginx

```
apt update -y
apt install -y nginx
nano /etc/nginx/conf.d/jenkins.conf
```

```
upstream jenkins {
  keepalive 32; # keepalive connections
  server jenkins_vm_ip:8080; # jenkins ip and port
}

# Required for Jenkins websocket agents
map $http_upgrade $connection_upgrade {
  default upgrade;
  '' close;
}

server {
    listen 443;
    server_name jenkins.lab.id;
    ssl_certificate /etc/letsencrypt/live/jenkins.lab.id/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/jenkins.lab.id/privkey.pem;

  # pass through headers from Jenkins that Nginx considers invalid
  ignore_invalid_headers off;

  location / {
      sendfile off;
      proxy_pass         http://jenkins;
      proxy_redirect     default;
      proxy_http_version 1.1;

      # Required for Jenkins websocket agents
      proxy_set_header   Connection        $connection_upgrade;
      proxy_set_header   Upgrade           $http_upgrade;

      proxy_set_header   Host              $host;
      proxy_set_header   X-Real-IP         $remote_addr;
      proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
      proxy_set_header   X-Forwarded-Proto $scheme;
      proxy_max_temp_file_size 0;

      #this is the maximum upload size
      client_max_body_size       10m;
      client_body_buffer_size    128k;

      proxy_connect_timeout      90;
      proxy_send_timeout         90;
      proxy_read_timeout         90;
      proxy_buffering            off;
      proxy_request_buffering    off; # Required for HTTP CLI commands
      proxy_set_header Connection ""; # Clear for keepalive
  }

}
```

```
service nginx reload
```

# Flow
![Dashboard3](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/2-flow-jenkins.png)

## Scenario
- Creata sample app web. 
- Github Repo integrate with Jenkins.

### Create Web Apps on Repository
Repository : [jenkins-argo](https://github.com/ammarun11/jenkins-argo)

- Create Credentials
![Dashboard4](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/3-credentials-jenkins.png)

- Create Job Pipelines
   - Pipeline Name : pipeline-demo
   - Pipeline Script from SCM : Git
   - Git Repo : https://github.com/rjhaikal/webpage 
   - Credentials : rjhaikal/****(dockerhub)
   - Branch : /master
   - Script Path : Jenkinsfile

![Dashboard5](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/4-pipeline1.png)
![Dashboard6](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/4-pipeline2.png)

## Build the Pipeline
Run Build now. Pipeline has build and successfully run, on the docker repository now has tags 1 and tags latest.
![Dashboard7](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/5-pipeline3.png)
![Dashboard8](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/5-pipeline4.png)