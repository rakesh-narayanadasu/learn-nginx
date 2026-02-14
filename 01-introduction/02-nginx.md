# Introduction to Nginx

## What Is NGINX?

NGINX is a high-performance web server first released over 20 years ago. It’s available in two editions: the open source Community Edition and the commercial NGINX Plus. NGINX powers static content delivery, load balancing, reverse proxy, and more across Linux, macOS, and Windows.

## Cross-Platform Support

NGINX overcomes the scalability and performance bottlenecks of legacy web servers. It installs easily on all major operating systems and delivers consistent throughput under heavy load.

## Historical Context

Originally created to challenge Apache HTTP Server and Microsoft Internet Information Services (IIS), NGINX quickly gained traction thanks to its lightweight, asynchronous design.

## Asynchronous, Event-Driven Architecture

One of NGINX’s key innovations is handling 10,000+ concurrent connections with minimal overhead. This makes it ideal for serving static assets HTML, images, audio, and video—more efficiently than traditional, process-based servers.


>  NGINX processes multiple client requests within a single worker process using non-blocking I/O.


| Feature                     | NGINX                      | Apache                   |
| --------------------------- | -------------------------- | ------------------------ |
| Architecture                | Asynchronous, event-driven | Process/thread-based     |
| Max. concurrent connections | ≥10,000                    | Varies, lower throughput |
| CPU & memory usage          | Low                        | Higher                   |
| Static content performance  | Excellent                  | Good                     |

Recent benchmarks show NGINX can handle up to four times as many connections as Apache, with lower latency and reduced resource consumption.

## NGINX Editions

NGINX is distributed in two main editions:

| Edition                 | Key Features                                   | Support           | Download URL                                                                   |
| ----------------------- | ---------------------------------------------- | ----------------- | ------------------------------------------------------------------------------ |
| Community (Open Source) | Core HTTP, reverse proxy, load balancing       | Community forum   | [https://nginx.org/download](https://nginx.org/download)                       |
| NGINX Plus (Commercial) | Advanced modules, dashboard, WAF, 24×7 support | Paid subscription | [https://nginx.com/products/nginx-plus](https://nginx.com/products/nginx-plus) |


>  Check the licensing and support terms before deploying NGINX Plus in production environments.

## Official Download Sites

For the open source release, visit nginx.org. To learn about NGINX Plus, head to nginx.com.

## Market Share & Adoption

According to a June 2024 survey, NGINX holds 21% market share among web servers—and 32% when including OpenResty, an enhanced NGINX distribution. It ranks second only to Cloudflare in the top million sites and leads in overall domains and compute infrastructure.

Combined, NGINX and OpenResty power roughly two-thirds of all Internet domains. Major organizations such as GitHub, Cloudflare, LinkedIn, Microsoft, and Netflix rely on NGINX for reliable, scalable performance.

| Web Server | Market Share¹ |
| ---------- | ------------- |
| Cloudflare | 34%           |
| NGINX      | 21%           |
| OpenResty  | 11%           |
| Others     | 34%           |

¹ June 2024 survey data


## Links and References

* Official NGINX Documentation: [https://nginx.org/en/docs/](https://nginx.org/en/docs/)
* NGINX Plus Overview: [https://nginx.com/products/nginx-plus/](https://nginx.com/products/nginx-plus/)
* OpenResty: [https://openresty.org/en/](https://openresty.org/en/)

# Nginx Architecture

In modern web environments, servers must handle thousands of simultaneous connections with minimal latency. Nginx achieves this through a lightweight, non-blocking event-driven architecture paired with a master-worker process model. 

## Event-Driven Architecture Overview

Nginx decouples request acceptance from request processing. It listens for new events, delegates work, and immediately returns to watching for additional activity.

## How Nginx Manages Asynchronous Processing

Under the hood, Nginx uses non-blocking I/O and a single-threaded event loop per worker to juggle connections efficiently:

>  Nginx’s asynchronous event loop ensures that while one request awaits data (disk I/O, upstream response), the worker can serve other clients without delay.


## The Restaurant Analogy: Mapping Requests to Events


In Nginx, each HTTP request follows these steps:

1. **Incoming Request**\
   The client issues an HTTP/S request.
2. **Event Loop**\
   Nginx accepts the connection and returns immediately to monitor other events.
3. **Processing Event**\
   The worker reads files, queries databases, or proxies to an upstream server. If I/O is required, it switches context to serve another request.
4. **Response Sent**\
   Once processing completes, Nginx replies to the client and continues the loop.

## Event Handling Sequence in Nginx

The diagram below outlines how a worker process manages multiple requests simultaneously:

### Step-by-Step Flow

* **Accept**: New connection arrives.
* **Register**: Connection is added to the event loop.
* **Dispatch**: Worker processes available events.
* **I/O Wait**: If blocked, event loop switches to another request.
* **Complete**: Response is sent when processing is done.

## Master and Worker Processes

To leverage multi-core CPUs and isolate failures, Nginx uses a master–worker architecture:

| Process Type | Responsibility                                           | Handles Client Requests? |
| ------------ | -------------------------------------------------------- | ------------------------ |
| **Master**   | Reads configuration, spawns/reloads worker processes     | No                       |
| **Worker**   | Runs event loop, accepts connections, processes requests | Yes                      |

### Master Process

* Supervises worker lifecycle
* Reloads configuration without downtime
* Never blocks on I/O or client handling

### Worker Processes

* Each runs an independent, single-threaded event loop
* Handles connection acceptance, reading, writing, and multiplexing
* Scales across CPU cores by running multiple workers

## Further Reading and References

* [Official Nginx Documentation](https://nginx.org/en/docs/)
* [Event-driven architecture on Wikipedia](https://en.wikipedia.org/wiki/Event-driven_architecture)
* [High Performance Browser Networking](https://hpbn.co/)

By combining non-blocking I/O, efficient event loops, and a robust master–worker model, Nginx delivers the scalability and low latency demanded by today’s Internet services.


# Nginx Use Cases

Nginx is more than a high-performance web server. It can improve your architecture by acting as a load balancer, reverse proxy, forward proxy, or caching layer—boosting scalability, reducing latency, and enhancing security.

Key benefits include:

* Distributing requests for high availability
* Offloading SSL/TLS and request routing
* Caching responses to cut backend load
* Controlling outbound traffic and anonymizing clients

Explore each use case below, complete with configuration examples and best practices.


## Load Balancing

By distributing incoming requests across multiple servers, Nginx prevents any single backend from becoming a bottleneck. You declare an `upstream` block listing your servers, then proxy traffic to it.

```nginx  theme={null}
upstream backend {
    server backend1.example.com;
    server backend2.example.com max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

>  Nginx supports multiple algorithms including `round_robin` (default), `least_conn`, and `ip_hash`. Choose one based on your workload characteristics.

## Reverse Proxy

A reverse proxy accepts client requests, applies routing or SSL offloading, then forwards them to one or more backend servers. This hides your infrastructure behind a single public endpoint.

```nginx  theme={null}
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/nginx/ssl/example.crt;
    ssl_certificate_key /etc/nginx/ssl/example.key;

    location / {
        proxy_pass       http://internal_app;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Load Balancer vs. Reverse Proxy

| Feature          | Load Balancer                     | Reverse Proxy                           |
| ---------------- | --------------------------------- | --------------------------------------- |
| Primary Role     | Distribute traffic across servers | Intercept and forward requests          |
| Backend Servers  | Requires two or more              | Can work with a single server           |
| Common Use Cases | Scaling, failover, health checks  | SSL/TLS termination, path-based routing |


## Forward Proxy

A forward proxy sits between clients and the internet, filtering or anonymizing outbound requests. Configure Nginx to restrict sites or mask client IPs for privacy.

```nginx  theme={null}
server {
    listen 3128;

    resolver 8.8.8.8;
    proxy_pass_request_headers on;

    location / {
        proxy_pass $scheme://$http_host$request_uri;
        proxy_hide_header Proxy-Authorization;
    }
}
```

>  Opening a forward proxy to the public can lead to abuse. Always secure it with `allow`/`deny` or authentication mechanisms.

## Caching

Caching with Nginx reduces response times and eases load on backend services. Define a cache zone, set key parameters, and control how responses are stored.

```nginx  theme={null}
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m 
                 inactive=60m max_size=1g;

server {
    listen 80;
    location / {
        proxy_cache my_cache;
        proxy_pass http://backend;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404 1m;
    }
}
```

>  Monitor cache usage and tune `inactive` and `max_size` to avoid running out of disk space. Use `proxy_cache_bypass` for selective caching.

## Links and References

* [Nginx Official Documentation](https://nginx.org/en/docs/)
* [HTTP Load Balancing with Nginx](https://nginx.org/en/docs/http/load_balancing.html)
* [Using Nginx as a Forward Proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
* [Caching Guide in Nginx](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache)
