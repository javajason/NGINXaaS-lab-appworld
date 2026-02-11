# Lab 4: Protecting Applications with Rate Limiting

## Overview
Welcome to **Lab 4**. In this module of **"Mastering cloud-native app delivery,"** we focus on application resilience. You will protect the **OWASP Juice Shop** application from brute-force attempts and DoS-like traffic patterns by implementing **NGINX Rate Limiting**.

---

## 🏗️ Pre-provisioned Infrastructure
The following resources are prepared for this lab:
* **Ubuntu VM:** Running OWASP Juice Shop in a Docker container on **Port 3000**.
* **Internal IP:** Your instructor will provide the private IP of the Juice Shop VM.

---

## 🚀 Lab Exercises

### Task 1: Configure NGINX for Juice Shop
First, we must point NGINX to the new Juice Shop backend.

1. Browse to your **NGINX for Azure** resource in the Portal.
2. Under **Settings**, select **NGINX configuration**.
3. Create a new file: `/etc/nginx/conf.d/juiceshop.conf`.
4. Paste the following configuration, replacing `[VM_INTERNAL_IP]` with your Ubuntu VM's private IP:

```nginx
upstream juiceshop_backend {
    server [VM_INTERNAL_IP]:3000;
}

server {
    listen 80;
    server_name juiceshop.example.com;

    location / {
        proxy_pass http://juiceshop_backend;
        proxy_set_header Host $host;
    }
}
