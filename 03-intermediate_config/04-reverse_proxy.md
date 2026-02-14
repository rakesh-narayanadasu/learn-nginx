# Reverse Proxy

Let's explore reverse proxies, their functionality, benefits, and differences from load balancers in web architecture.

## What Is a Reverse Proxy?

A **reverse proxy** sits between clients and one or more backend servers. It receives incoming requests, routes them to the appropriate server pool, and returns the server’s response to the client. Common use cases include:

* Hiding backend server identities
* SSL/TLS offloading
* Caching static assets
* Distributing traffic across multiple application servers

>  A reverse proxy can improve security, performance, and scalability by centralizing request handling, encryption, and caching.

## Reverse Proxy vs. Load Balancer

While both components sit in front of your servers, their primary responsibilities differ:

| Feature              | Reverse Proxy                            | Load Balancer                            |
| -------------------- | ---------------------------------------- | ---------------------------------------- |
| Main Role            | Hide backend details and forward traffic | Distribute traffic evenly across servers |
| SSL/TLS Offloading   | Yes                                      | Sometimes (depends on implementation)    |
| Caching              | Yes                                      | Rarely                                   |
| Application Firewall | Often integrated                         | Rarely                                   |

## Placing Application Frameworks Behind a Reverse Proxy

Modern web apps often use frameworks like React (Node.js), Flask (Python), Rails (Ruby), or Laravel (PHP). By default, these bind to local ports (e.g., React on 3000, Flask on 5000). In production:

* The reverse proxy exposes only itself to the Internet
* Backend servers remain isolated on private networks

>  Never expose application-framework ports directly to the public Internet. Always route through a hardened reverse proxy.

## SSL/TLS Termination (Offloading)

Offloading SSL/TLS decryption to the reverse proxy reduces CPU load on your application servers. Clients connect over HTTPS to the proxy, which decrypts the traffic, forwards plain HTTP to backends, then re-encrypts responses.

### 1. Basic HTTP Reverse Proxy

```nginx  theme={null}
http {
    upstream backend {
        server 10.10.0.101:80;
        server 10.10.0.102:80;
        server 10.10.0.103:80;
    }
}

server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://backend/;
    }
}
```

### 2. HTTPS Termination at the Proxy

```nginx  theme={null}
server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate     /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    ssl_protocols       TLSv1.2 TLSv1.3;

    location / {
        proxy_pass http://backend/;
    }
}
```

### 3. End-to-End TLS Encryption

When compliance mandates encrypted links all the way to your app servers, enable HTTPS in `proxy_pass`:

```nginx  theme={null}
http {
    upstream backend {
        server 10.10.0.101:443;
        server 10.10.0.102:443;
        server 10.10.0.103:443;
    }
}

server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate     /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    ssl_protocols       TLSv1.2 TLSv1.3;

    location / {
        proxy_pass https://backend/;
    }
}
```

## Caching to Reduce Backend Load

Caching static files and repeatable responses (images, CSS, JSON) at the proxy layer decreases latency and backend CPU usage. NGINX can act as a cache server to serve frequent requests directly from local storage.

### Sample Cache Configuration

```nginx  theme={null}
http {
    proxy_cache_path /var/lib/nginx/cache levels=1:2 zone=app_cache:8m;
    proxy_cache_key "$scheme$request_method$host$request_uri$is_args$args";
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404 1m;
}

server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_cache        app_cache;
        proxy_cache_bypass $http_cache_control;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_pass         http://backend/;
    }
}
```

## Further Reading

* [NGINX Reverse Proxy Official Docs](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
* [TLS/SSL Termination Strategies](https://security.example.com/tls-termination)
* [Best Practices for Web Caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)


# Demo Reverse Proxy

In this we will set up NGINX as a reverse proxy and simple load balancer for two Flask apps running on separate hosts (`node01` and `node02`) at port 5000. Incoming HTTP requests hit NGINX, which forwards them to healthy backends—improving scalability and fault tolerance.

Below is the topology for this demo:

| Host     | Role                          | IP Address     |
| -------- | ----------------------------- | -------------- |
| `nginx`  | Reverse proxy server (NGINX)  | 192.230.206.12 |
| `node01` | Flask application (port 5000) | 192.230.206.3  |
| `node02` | Flask application (port 5000) | 192.230.206.6  |

### SSH into the Backends

From the `nginx` host, connect to each Flask server:

```bash  theme={null}
root@nginx ~ ➜ ssh node01
# or
root@nginx ~ ➜ ssh node02
```

## Step 1: Verify Backend Servers

On **node01**:

```bash  theme={null}
root@node01 ~ ➜ curl localhost:5000
# <h1>Hello, Human!</h1>[Not Authenticated]
```

On **node02**:

```bash  theme={null}
root@node02 ~ ➜ curl localhost:5000
# <h1>Hello, Human!</h1>[Not Authenticated]
```

## Step 2: Configure Firewall Rules

Ensure only the NGINX proxy can reach port 5000 on your Flask servers, while keeping SSH open for management.

>  If UFW is already enabled, skip `sudo ufw enable` and proceed with rule creation.

```bash  theme={null}
# 1. Enable UFW (if not already running)
sudo ufw enable

# 2. Allow SSH management
sudo ufw allow 22/tcp

# 3. Permit Flask traffic from the proxy only
sudo ufw allow from 192.230.206.12 proto tcp to any port 5000

# 4. Verify the active rules
sudo ufw status
# Status: active
# To                         Action      From
# --                         ------      ----
# 22/tcp                     ALLOW       Anywhere
# 5000/tcp                   ALLOW       192.230.206.12
# 22/tcp (v6)                ALLOW       Anywhere (v6)
```

## Step 3: Configure NGINX as a Reverse Proxy

1. Remove the default site and switch to the config directory:

   ```bash  theme={null}
   cd /etc/nginx/sites-enabled
   sudo rm default
   cd /etc/nginx/sites-available
   ```

2. Create `/etc/nginx/sites-available/helloworld` with upstream and server blocks:

   ```nginx  theme={null}
   # Upstream definition for Flask backends
   upstream hello_world {
       server 192.230.206.3:5000;
       server 192.230.206.6:5000;
   }

   server {
       listen 80;
       server_name helloworld.com;

       root /var/www/html;
       index index.html index.htm;

       location / {
           proxy_pass http://hello_world;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
   ```

3. Enable the site and reload NGINX:

   ```bash  theme={null}
   sudo ln -s /etc/nginx/sites-available/helloworld /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo nginx -s reload
   ```

## Step 4: Test the Reverse Proxy

Use `curl` with a custom Host header to emulate requests to `helloworld.com`:

```bash  theme={null}
# Root path
curl -H "Host: helloworld.com" http://localhost
# /foo endpoint
curl -H "Host: helloworld.com" http://localhost/foo
# /bar endpoint
curl -H "Host: helloworld.com" http://localhost/bar
# <h1>Bar page</h1>...
```

## Step 5: Simulate a Backend Failure

Comment out one server in the `upstream` block to test NGINX’s resilience:

```nginx  theme={null}
# /etc/nginx/sites-available/helloworld
upstream hello_world {
    server 192.230.206.3:5000;
    # server 192.230.206.6:5000;  # node02 offline
}
```

Reload NGINX and re-run the curl tests; traffic should automatically route to the healthy backend.

```bash  theme={null}
sudo nginx -s reload
curl -H "Host: helloworld.com" http://localhost/foo
curl -H "Host: helloworld.com" http://localhost/bar
```

>  Ensure that `helloworld.com` resolves to your NGINX server IP (192.230.206.12) in DNS or `/etc/hosts` before testing.

You’ve now configured NGINX as a robust reverse proxy and simple load balancer for your Flask applications. Next, explore advanced proxy settings, SSL termination, and health checks in the [NGINX documentation](https://nginx.org/en/docs/).
