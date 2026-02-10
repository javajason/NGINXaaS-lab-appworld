# Lab 2: Advanced Traffic Management with NGINX for Azure

## Overview
Welcome to **Lab 2**. In this session of **"Mastering cloud-native app delivery: Unlocking advanced capabilities and use cases with F5’s ADCaaS,"** we move beyond basic connectivity to functional load balancing. 

In this module, you will configure your NGINX for Azure instance to route traffic to a backend application running on a pre-provisioned Linux workload using the Azure Portal.

---

## 🏗️ Pre-provisioned Infrastructure
To focus specifically on NGINX configuration, the following backend resources have been pre-deployed for you:

* **Ubuntu VM:** A Linux virtual machine running a Dockerized "Cafe" application.
* **Internal IP:** Your instructor will provide the internal IP of your Ubuntu VM (e.g., `n4a-ubuntuvm`).

---

## 🚀 Lab Exercises

### Configure NGINX for Azure to Load Balance Docker Containers

In this section, we will configure NGINX for Azure to load balance traffic to the Docker containers running on your Ubuntu VM.



#### 1. Access the Configuration Editor
1. Open the **Azure Portal** and navigate to your **Resource Group**. 
2. Click on your NGINX for Azure resource (usually named **nginx4a**), which will open the Overview section. 
3. From the left pane, click on **NGINX Configuration** under the Settings section.

#### 2. Create the Upstream Configuration
1. Click on **+ New File** to create a new NGINX config file. 
2. Name the new file: `/etc/nginx/conf.d/cafe-docker-upstreams.conf`.
   > **Important:** You must use the full Linux `/directory/filename` path for every config file. The Azure Portal does not support drag-and-drop.
3. Copy and paste the following contents into the editor:

```nginx
# Nginx 4 Azure, Cafe Nginx Demo Upstreams
# cafe-nginx servers
#
upstream cafe_nginx {
    zone cafe_nginx 256k;
    
    # These correspond to the ports exposed by the Docker containers
    server n4a-ubuntuvm:81;
    server n4a-ubuntuvm:82;
    server n4a-ubuntuvm:83;

    keepalive 32;
}
```
 4. Click Submit to save this part of the configuration.

3. Create the Virtual Server Configuration

  1.  Click + New File again.

  2. Name the second file: /etc/nginx/conf.d/cafe.example.com.conf.

  3. Copy and paste the following contents into the editor:

   # Nginx 4 Azure - Cafe Nginx HTTP
```nginx
server {
    
    listen 80;      # Listening on port 80

    server_name cafe.example.com;   # Set hostname to match in request
    status_zone cafe.example.com;   # Metrics zone name

    access_log  /var/log/nginx/cafe.example.com.log main;
    error_log   /var/log/nginx/cafe.example.com_error.log info;

    location / {
        proxy_pass http://cafe_nginx;        # Proxy and load balance to the upstream group
        add_header X-Proxy-Pass cafe_nginx;  # Custom verification header
    }
}
```
