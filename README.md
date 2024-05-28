# Reverse Proxy with Threefold Dashboard

## Table of Contents  

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Deploy Linux Machine](#deploy-linux-machine)
- [Preparing a Simple Application Server on Port 8000](#preparing-a-simple-application-server-on-port-8000)
- [NGINX Reverse Proxy](#nginx-reverse-proxy)
- [CADDY Reverse Proxy](#caddy-reverse-proxy)

## Introduction

A reverse proxy is an essential tool for exposing an application server to the internet. Whether you're running a Node.js app or a Flask server, these applications usually bind to localhost, making them accessible only on the host machine. While binding to an external IP is an option, using a reverse proxy in production enhances security by isolating the server, centralizing firewall protection, and reducing the attack surface for threats like [denial-of-service (DoS) attacks](https://en.wikipedia.org/wiki/Denial-of-service_attack).

## Prerequisites

To complete this tutorial, you will need:

- An Ubuntu server with a public IP, set up according to our [Micro Virtual Machine](https://manual.grid.tf/documentation/dashboard/solutions/vm.html) setup guide.
- The location of the application server you want to proxy, referred to as `app_server_address` in this tutorial. This can be an IP address with a TCP port (e.g., the default Python server address <http://127.0.0.1:8000>). If you don’t have an application server ready for testing, you will be guided through setting up a Python `http.server` instance that binds to <http://127.0.0.1:8000>.
- A domain name pointed at your server’s public IP. This will be configured with [Nginx](https://nginx.org/en/) or [Caddy](https://caddyserver.com/) to proxy your application server.

## Deploy Linux Machine

To deploy your Ubuntu instance, follow our [How to deploy a micro virtual machine](https://manual.grid.tf/documentation/dashboard/solutions/vm.html) guide. Once deployed, you can attach a domain name to your instance by referring to our [How to add a domain name to my deployed micro virtual machine](https://manual.grid.tf/documentation/dashboard/solutions/add_domain.html) guide.

## Preparing a Simple Application Server on Port 8000

You can easily use the following command to prepare a default Python server that serves the current directory:

```sh
python3 -m http.server 8000 &
```

## NGINX Reverse Proxy

After accessing your machine via `ssh`, you can easily follow the next steps.

### Step 1: Install Nginx

First, install Nginx on your server:

```sh
apt update
apt install nginx nano
```

### Step 2: Configure Nginx

Create a new Nginx configuration file for your application. For example, create a file named `myapp.conf` in the `/etc/nginx/sites-available` directory:

```sh
nano /etc/nginx/sites-available/myapp.conf
```

Add the following configuration to the file, replacing `your_domain` with your domain name, and `localhost:8000` with the address and port your application server is listening on:

```nginx
server {
    listen 80;
    server_name your_domain;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Step 3: Enable the Configuration

Enable the new configuration by creating a symbolic link to it in the `sites-enabled` directory:

```sh
ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/
```

### Step 4: Test the Configuration

Test the Nginx configuration to ensure there are no syntax errors:

```sh
nginx -t
```

### Step 5: Restart Nginx

Restart Nginx to apply the new configuration:

```sh
service nginx restart
```

### Final Step(Nginx): Configure Your Application

Make sure your application server `(e.g., Node.js, Flask, or Django)` is configured to bind to localhost or `127.0.0.1` and the correct port `(e.g., 8000)`.

## CADDY Reverse Proxy

After accessing your machine via `ssh`, you can easily follow the next steps.

### Step 1: Install Caddy

First, install Caddy on your server:

```sh
apt update
apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list
apt update
apt install caddy nano
```

### Step 2: Configure Caddy

Create a Caddyfile to define your reverse proxy configuration. The default location for the Caddyfile is /etc/caddy/Caddyfile, but you can place it in another location and specify it with the `--config` flag when running Caddy.

```sh
nano /etc/caddy/Caddyfile
```

Add the following configuration to the file, replacing `your_domain` with your domain name, and `localhost:8000` with the address and port your application server is listening on:

```caddy
80: {
  reverse_proxy localhost:8000
}
```

- **Caddy automatically manages SSL certificates for you, so you don't need to manually configure HTTPS.**

### Step 3: Start and Enable Caddy

Start the Caddy service and enable it to start on boot:

```sh
caddy reload --config /etc/caddy/Caddyfile 
```

### Final Step(Caddy): Configure Your Application

Make sure your application server `(e.g., Node.js, Flask, or Django)` is configured to bind to localhost or `127.0.0.1` and the correct port `(e.g., 8000)`.
