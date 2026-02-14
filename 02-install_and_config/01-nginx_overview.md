# Nginx Overview

This Nginx configuration, focusing on the nginx.conf file and its core components for optimizing web server performance.

The core components of Nginx configuration by exploring the **`nginx.conf`** file typically located at `/etc/nginx/nginx.conf` on Linux systems. You’ll learn how to structure global settings, optimize connection handling, define HTTP parameters, and create virtual hosts (server blocks).

At a glance, an **`nginx.conf`** file is divided into four main sections:

1. **Global settings**
2. **events block**
3. **http block**
4. **server block**



## nginx.conf Structure

1. **Global settings**\
   Define user permissions, worker processes, PID file location, compression, caching, and more.

2. **events block**\
   Controls Nginx’s event model and the maximum number of simultaneous connections per worker.

3. **http block**\
   Contains HTTP directives for logging, timeouts, compression, MIME types, and includes for server blocks.

4. **server block**\
   Configures how Nginx responds to requests for specific domain names or IP addresses (virtual hosts).




## Global Settings, Events, and HTTP Blocks

Below is a pared-down example of an **`nginx.conf`** layout including global directives, the `events` block, and the `http` block. Notice how server blocks are included separately.

```nginx  theme={null}
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    gzip            on;

    log_format main '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent"';

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Include virtual host definitions
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

Key directives explained:

* `user`\
  System user for Nginx worker processes (e.g., `www-data` on Debian/Ubuntu).
* `worker_processes`\
  Number of worker processes—`auto` matches CPU cores.
* `pid`\
  Path to the master process PID file.
* `events → worker_connections`\
  Maximum simultaneous connections each worker can handle.
* `http`\
  Configures HTTP-level settings: compression (`gzip`), timeouts, logging formats, MIME types, and includes server blocks.



## Server Blocks (Virtual Hosts)

Server blocks, or virtual hosts, let you host multiple domains on one Nginx instance. Incoming requests are routed to the matching block based on `server_name` or IP address.


### Example: Basic HTTP Server Block

```nginx  theme={null}
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/example.com/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Directive breakdown:

* `listen`\
  Port for incoming traffic (`80` for HTTP, `443` for HTTPS).
* `server_name`\
  Domain names or IP addresses handled by this block.
* `root`\
  Path to the website’s document root.
* `index`\
  Default file(s) served when a directory is requested.
* `location /`\
  URI matching; `try_files` checks for existing files or directories and returns a 404 if not found.




## Nginx Directory Structure

Nginx uses a standardized directory layout for its configuration files and web content. Below is a quick reference of the most common paths:

| Path                          | Description                                                   |
| ----------------------------- | ------------------------------------------------------------- |
| `/etc/nginx/nginx.conf`       | Main configuration file                                       |
| `/etc/nginx/sites-available/` | Store individual server block files                           |
| `/etc/nginx/sites-enabled/`   | Symbolic links to enabled sites from *sites-available*        |
| `/etc/nginx/conf.d/`          | Additional configuration snippets (e.g., SSL, load balancing) |
| `/var/www/...`                | Default web content roots (Debian/Ubuntu)                     |
| `/usr/share/nginx/html`       | Default web root (RHEL/CentOS)                                |
| `/etc/nginx/mime.types`       | MIME type definitions                                         |
| `/run/nginx.pid`              | PID file location                                             |
| `/var/log/nginx/`             | Access and error logs                                         |




## Essential Nginx Commands

Use these commands to inspect, test, and manage your Nginx server:

| Command                        | Description                             |
| ------------------------------ | --------------------------------------- |
| `nginx -h`                     | Display help and available options      |
| `nginx -v`                     | Show Nginx version                      |
| `nginx -V`                     | Show version and compile-time options   |
| `nginx -t`                     | Check configuration syntax and validity |
| `nginx -T`                     | Dump complete configuration for review  |
| `nginx -s reload`              | Reload configuration without downtime   |
| `nginx -s stop`                | Graceful shutdown                       |
| `nginx -s quit`                | Immediate shutdown                      |
| `nginx -s reopen`              | Reopen log files                        |
| `sudo systemctl reload nginx`  | Reload using systemd (graceful)         |
| `sudo systemctl restart nginx` | Restart Nginx (brief downtime possible) |
| `sudo systemctl status nginx`  | Check Nginx service status              |

Example syntax check:

```bash  theme={null}
sudo nginx -t
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful
```



## Links and References

* [Official Nginx Documentation](https://nginx.org/en/docs/)
* [Nginx Beginner’s Guide](https://nginx.org/en/docs/beginners_guide.html)
* [DigitalOcean Nginx Tutorials](https://www.digitalocean.com/community/tags/nginx)
