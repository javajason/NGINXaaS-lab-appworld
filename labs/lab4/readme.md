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
```
5. Submit and verify you can access the Juice Shop via your browser (ensure your hosts file is updated for juiceshop.example.com).

### Task 2: Implement Rate Limiting

Now, we will restrict the number of requests a single client can make to prevent abuse.

1. Go back to the NGINX Configuration editor in the Azure Portal.

2. Open your main /etc/nginx/nginx.conf file.

3. In the http block (above the include line), define the rate limit zone:

```nginx
http {
    # Define a shared memory zone 'mylimit' to track IP addresses 
    # and allow 1 request per second (1r/s)
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;

    include /etc/nginx/conf.d/*.conf;
}
```
3. Click Submit.

5. Now, open /etc/nginx/conf.d/juiceshop.conf and apply the limit to the location block:
   
```nginx
  server {
    listen 80;
    server_name juiceshop.example.com;

    location / {
        # Apply the rate limit
        # 'burst=5' allows a small buffer, 'nodelay' ensures immediate response
        limit_req zone=mylimit burst=5 nodelay;

        proxy_pass http://juiceshop_backend;
        proxy_set_header Host $host;
    }
}
```
6. Click Submit.

### Task 3: Test the Rate Limit

 1. To see the rate limiting in action, we need to send requests faster than 1 per second.

 2. Open a terminal on your local machine.
 
 3. Like before, update your local system's DNS `/etc/hosts` file. This time, you will add the hostname `juiceshop.example.com` after your previous update (`cafe.example.com`), as shown below:

    ```bash
    cat /etc/hosts

    127.0.0.1 localhost
    ...

    # Nginx for Azure testing
    11.22.33.44 cafe.example.com
    11.22.33.44 juiceshop.example.com

    ...
    ```

    where
   - `11.22.33.44` replace with your `n4a-publicIP` resource IP address.

 4. Once you have updated the host your /etc/hosts file, save it and quit vi tool.
 
 5. Run a loop to hit the site rapidly:

```bash
  for i in {1..10}; do curl -I http://juiceshop.example.com; done
```

6. Observe the results: You should see several 200 OK responses followed by 503 Service Temporarily Unavailable (or 429 Too Many Requests depending on NGINX version/config) as the rate limit kicks in.

### Task 4: Verify via Log Analytics

 1. Navigate to your Log Analytics Workspace.

 2. Run the following query to see the rejected requests:

```code snippet
NGINXAccessLogs
| where HttpStatus >= 400
| project TimeGenerated, ClientIp, RequestUri, HttpStatus
| order by TimeGenerated desc
```


Congratulations on completing Lab 4!
