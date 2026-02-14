# Load Balancer

This explains load balancing with Nginx, how to configure upstream pools, choose the right algorithm, and ensure high availability for your web applications.

## What Is a Load Balancer?

A load balancer is a network device software or hardware that distributes incoming traffic across multiple backend servers. It prevents any single server from becoming a performance bottleneck or single point of failure. Without a load balancer, every request hits one server. If that lone server crashes, your entire site goes down—and you lose traffic, revenue, and user trust.

## Why Use Nginx as a Load Balancer?

Nginx not only distributes traffic but also performs health checks on backend servers. When a node fails, Nginx automatically marks it unhealthy and stops sending traffic its way, keeping your site available on remaining nodes.

> By default, Nginx excludes failed servers from the pool. You can tune probe intervals, timeouts, and retry counts for advanced health checks.

## Configuring Upstream Pools

Upstream blocks group your backend servers into a single logical name. Later, you reference that name with `proxy_pass` in a server block.

```nginx  theme={null}
upstream backend {
    server 10.10.0.101:80;
    server 10.10.0.102:80;
    server 10.10.0.103:80;
}

server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://backend;
    }
}
```

## Load Balancing Methods

Nginx supports multiple algorithms to suit different workloads. Here’s a quick summary:

| Algorithm            | Use Case                                | Directive                     |
| -------------------- | --------------------------------------- | ----------------------------- |
| Round Robin          | Even distribution (default)             | —                             |
| Weighted Round Robin | Prioritize higher-capacity servers      | `weight=`                     |
| IP Hash              | Sticky sessions based on client IP      | `ip_hash`                     |
| Least Connections    | Send to server with fewest active conns | `least_conn`                  |
| Least Time\*         | Fastest response (NGINX Plus only)      | `least_time last_byte/header` |

\* Requires NGINX Plus subscription

### 1. Round-Robin (Default)

Distributes requests evenly in a circular order.

```nginx  theme={null}
upstream backend {
    server 10.10.0.101:80;
    server 10.10.0.102:80;
    server 10.10.0.103:80;
}

server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://backend;
    }
}
```

### 2. Weighted Round-Robin

Assign heavier weights to more powerful servers so they receive a larger share of traffic.

```nginx  theme={null}
upstream backend {
    server 10.10.0.101:80 weight=4;
    server 10.10.0.102:80 weight=2;
    server 10.10.0.103:80 weight=1;
}

server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://backend;
    }
}
```

### 3. IP Hash (Sticky Sessions)

Ensures the same client IP always hits the same server ideal for session persistence when data is stored in memory on each backend.

```nginx  theme={null}
upstream backend {
    ip_hash;
    server 10.10.0.101:80;
    server 10.10.0.102:80;
    server 10.10.0.103:80;
}

server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://backend;
    }
}
```

>  Sticky sessions can lead to uneven load if some clients generate more traffic. Use only when backend-level session sharing isn’t an option.

### 4. Least Connections

Routes each new request to the server with the fewest active connections—ideal for dynamic workloads.

```nginx  theme={null}
upstream backend {
    least_conn;
    server 10.10.0.101:80;
    server 10.10.0.102:80;
    server 10.10.0.103:80;
}

server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://backend;
    }
}
```

### 5. Least Time (NGINX Plus)

Selects the backend with the fastest response time either time to first byte or last byte. This requires NGINX Plus.

```nginx  theme={null}
upstream backend {
    least_time last_byte/header;
    server 10.10.0.101:80;
    server 10.10.0.102:80;
    server 10.10.0.103:80;
}

server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://backend;
    }
}
```

## References

* [Nginx Load Balancing Documentation](https://nginx.org/en/docs/http/load_balancing.html)
* [Health Checks in NGINX Plus](https://docs.nginx.com/nginx/admin-guide/load-balancer/monitoring-health/)
* [Nginx Upstream Module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html)
* [NGINX Plus Features](https://www.nginx.com/products/nginx/)


# Demo Load Balancer


Learn how to configure Nginx as a load balancer using three common algorithms:

1. **Round Robin**: Distributes requests evenly across all backend servers.
2. **Weighted Round Robin**: Assigns traffic proportionally based on server weights.
3. **IP Hash**: Routes each client IP to the same backend to maintain session persistence.

## Load Balancing Algorithm Comparison

| Algorithm            | Behavior                                           | Use Case                                          |
| -------------------- | -------------------------------------------------- | ------------------------------------------------- |
| Round Robin          | Cycles through all servers equally                 | Simple distribution without session affinity      |
| Weighted Round Robin | Traffic proportionate to configured server weights | Differing server capacities or performance levels |
| IP Hash              | Routes same client IP to the same backend          | Session persistence without cookies or headers    |

## Environment Setup

We have three Ubuntu nodes:

* **nginx**: Nginx load balancer (192.230.202.10)
* **node01** & **node02**: Apache backend servers

On each backend, Apache is running and UFW is active but only port 22 is open by default:

```bash  theme={null}
# On node01 (repeat on node02)
root@node01:~# systemctl status apache2
● apache2.service - The Apache HTTP Server
   Active: active (running) ...

root@node01:~# ufw status
Status: active

To                         Action    From
--                         ------    ----
22/tcp                     ALLOW     Anywhere
22/tcp (v6)                ALLOW     Anywhere (v6)
```

> For security, restrict HTTP access so only the load balancer (192.230.202.10) can reach port 80 on your backends.


```bash  theme={null}
# On node01
root@node01:~# ufw allow from 192.230.202.10 proto tcp to any port 80

# Verify the rule
root@node01:~# ufw status
Status: active

To                         Action    From
--                         ------    ----
22/tcp                     ALLOW     Anywhere
192.230.202.10 80/tcp      ALLOW     Anywhere
22/tcp (v6)                ALLOW     Anywhere (v6)

# Repeat on node02
root@node02:~# ufw allow from 192.230.202.10 proto tcp to any port 80
```

## Configuring the Load Balancer

On the **nginx** node, create or edit your site configuration:

```bash  theme={null}
root@nginx:~# cd /etc/nginx/sites-available/
root@nginx:/etc/nginx/sites-available# vim apache-app
```

Start with a basic server block:

```nginx  theme={null}
server {
    listen 80;
    server_name apache.example.com;
    root /var/www/html;

    # Default index files
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 1. Round Robin

Add an `upstream` block for your backend pool:

```nginx  theme={null}
# Upstream (Round Robin)
upstream apache_example {
    server 192.230.202.12:80;
    server 192.230.202.3:80;
}

server {
    listen 80;
    server_name apache.example.com;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        proxy_pass http://apache_example;
    }
}
```

Enable the site and reload Nginx:

```bash  theme={null}
root@nginx:/etc/nginx/sites-available# nginx -t
root@nginx:/etc/nginx/sites-available# ln -s apache-app /etc/nginx/sites-enabled/
root@nginx:/etc/nginx/sites-available# nginx -s reload
```

Test with `curl` to see alternating responses:

```bash  theme={null}
root@nginx:~# curl localhost
<p>Served by node02</p>
root@nginx:~# curl localhost
<p>Served by node01</p>
```

### 2. Weighted Round Robin

Adjust the `upstream` block to assign server weights:

```nginx  theme={null}
# Upstream (Weighted Round Robin)
upstream apache_example {
    server 192.230.202.12:80 weight=10;
    server 192.230.202.3:80  weight=1;
}
```

Reload Nginx to apply:

```bash  theme={null}
root@nginx:~# nginx -s reload
```

Now about 10× more requests go to the higher-weighted server.

### 3. IP Hash

Enable `ip_hash` for session stickiness by client IP:

```nginx  theme={null}
# Upstream (IP Hash)
upstream apache_example {
    ip_hash;
    server 192.230.202.12:80;
    server 192.230.202.3:80;
}
```

Reload Nginx:

```bash  theme={null}
root@nginx:~# nginx -s reload
```

>  With `ip_hash`, each client IP always hits the same backend server. This is ideal for stateful applications.

You can extend this setup with health checks, SSL termination, or alternative Nginx modules.
