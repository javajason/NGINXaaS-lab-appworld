# Lab 2: Advanced Traffic Management with NGINX for Azure

## Overview
Welcome to **Lab 2**. In this session of **"Mastering cloud-native app delivery: Unlocking advanced capabilities and use cases with F5’s ADCaaS,"** we move beyond basic connectivity to functional load balancing. 

In this module, you will configure your NGINX for Azure instance to route traffic to a backend application running on a pre-provisioned Linux workload.

---

## 🏗️ Pre-provisioned Infrastructure
To focus specifically on NGINX configuration, the following backend resources have been pre-deployed for you:

* **Ubuntu VM:** A Linux virtual machine running a Dockerized "Hello World" application.
* **Internal IP:** Your instructor will provide the internal IP of your Ubuntu VM (e.g., `10.1.2.4`).

---

## 🚀 Lab Exercises

### Configure NGINX for Azure to Load Balance Docker Containers

In this section, we will configure NGINX for Azure to load balance traffic to the Docker container running on your Ubuntu VM.



1.  Browse to your **NGINX for Azure** deployment in the Azure Portal.
2.  Under **Settings**, select **NGINX configuration**.
3.  Select **Edit** to modify the configuration.
4.  **Update the main config:** Replace the existing configuration in `/etc/nginx/nginx.conf` with the following block to enable modular configuration:

```nginx
user nginx;
worker_processes auto;
worker_rlimit_nofile 8192;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    keepalive_timeout 65;

    # Task: Include modular configuration files
    include /etc/nginx/conf.d/*.conf;
}
