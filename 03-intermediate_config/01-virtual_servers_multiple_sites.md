# Intermediate Config introduction

Exploring powerful Nginx capabilities essential for production-grade deployments, including server blocks, redirects, URL rewriting, upstream pools, load balancing, reverse proxy, and caching.

We’ll explore several powerful Nginx capabilities that are essential for production-grade deployments. We’ll focus on the most common patterns and best practices, including:

1. **Hosting Multiple Domains (Server Blocks)**\
   Use `server_name` directives to serve different websites from a single Nginx instance.

2. **Redirects with the `return` Directive**\
   Implement HTTP-to-HTTPS redirects and simple URL redirections using `return`.

3. **URL Rewriting & Regex (Rewrite Module)**\
   Leverage regular expressions and the `rewrite` directive to transform and route incoming requests dynamically.

4. **Defining Upstream Pools**\
   Configure `upstream` blocks to group backend servers and manage proxy targets.

5. **Load Balancing Strategies**\
   Explore Nginx’s built-in algorithms—round robin, weighted, least connections, and more—to distribute traffic efficiently.

6. **Reverse Proxy Configuration**\
   Route client requests to application servers running on different hosts or ports with the `proxy_pass` directive.

7. **Caching for Performance**\
   Enable and tune Nginx’s caching mechanisms (`proxy_cache`, `fastcgi_cache`) to reduce backend load and improve response times.


# Virtual Servers Multiple Sites

To host multiple websites on a single Nginx server using virtual servers for cost reduction and simplified management.

In this you’ll learn how to host multiple websites on a single Nginx server using **virtual servers** (also known as *server blocks* or *virtual hosts*). Imagine your server as a building and each site as an apartment. Visitors arrive at the building address (your server IP/port) and Nginx routes them to the correct apartment (server block) based on the domain name in the `Host` header.

By consolidating sites on one machine, you reduce infrastructure costs and simplify management—just one SSH session to maintain all your domains.

## How Nginx Virtual Servers Work

When a request arrives, Nginx inspects the `Host` header (e.g., `example.com`, `mail.example.com`) and matches it against configured server blocks:

1. Nginx listens on a port (80 or 443 by default).
2. It checks the `Host` header.
3. It routes the request to the server block with the matching `server_name`.

Whether you’re serving a blog, mail client, or map application, each domain or subdomain points to its own configuration within the same Nginx instance.

Learn more in the [Nginx server block documentation](https://nginx.org/en/docs/http/server_names.html).

## Configuring a Basic Virtual Host

Here’s a minimal example for `example.com`:

```nginx  theme={null}
server {
    listen       80;
    server_name  example.com www.example.com;

    root   /var/www/example.com/html;
    index  index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Key directives:

* `listen 80;`\
  Binds this block to port 80 (HTTP).
* `server_name`\
  Lists domains and subdomains handled here.
* `root`\
  Points to the document root where your site files live.
* `index`\
  Specifies the default file to serve.
* `location /`\
  Tries to serve the requested URI or returns `404` if not found.

## Binding to an IP Address

You can serve content directly from an IP address instead of a domain:

```nginx  theme={null}
server {
    listen       80;
    server_name  172.217.22.14;

    root   /var/www/172.217.22.14/html;
    index  index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

This approach is useful for internal test servers or when DNS is not available.

## Serving on a Non-Standard Port

If your application runs on a different port (e.g., Jenkins on 8080), you can expose it:

```nginx  theme={null}
server {
    listen       8080;
    server_name  wiki.example.com;

    root   /var/www/wiki.example.com/html;
    index  index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Users must browse to `http://wiki.example.com:8080`.

For production, it’s best to configure a reverse proxy on ports 80/443 and forward traffic internally to port 8080. This avoids requiring users to specify `:8080` in the URL.

## Managing Multiple Server Blocks

You can place several server blocks in one file, but for maintainability, isolate each site into its own configuration:

```nginx  theme={null}
server {
    listen       80;
    server_name  honda.cars.com;

    root   /var/www/honda.cars.com/html;
    index  index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen       80;
    server_name  toyota.cars.com;

    root   /var/www/toyota.cars.com/html;
    index  index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Keep each site’s configuration in its own file (e.g., `/etc/nginx/sites-available/honda.cars.com` and symlink to `/etc/nginx/sites-enabled/`). A typo in one file won’t take down all your sites when you reload Nginx.


### Quick Reference: Nginx Directives

| Directive    | Purpose                         | Example                                    |
| ------------ | ------------------------------- | ------------------------------------------ |
| listen       | Port for incoming connections   | `listen 80;`                               |
| server\_name | Domain(s) served by this block  | `server_name example.com www.example.com;` |
| root         | Path to website files           | `root /var/www/example.com/html;`          |
| index        | Default file to serve           | `index index.html;`                        |
| location /   | URI handling and fallback rules | `try_files $uri $uri/ =404;`               |

## Links and References

* [Nginx Official Documentation](https://nginx.org/en/docs/)
* [DigitalOcean: How To Set Up Nginx Server Blocks](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-20-04)
* [Stack Overflow: Common Nginx Virtual Host Issues](https://stackoverflow.com/questions/tagged/nginx)


# Demo Configure Multiple Sites

This explains how to host multiple domains on a single Nginx server using virtual hosts.

In this you’ll learn how to host two distinct domains **[www.example1.com](http://www.example1.com)** and \*\*[www.example2.com\*\*—on](http://www.example2.com** on) a single Nginx server. Incoming HTTP requests are routed based on the `Host` header, matching the `server_name` directive within each server block (often called *virtual hosts*). 

## 1. Remove the Default Site

Before adding your own configurations, disable the default Nginx welcome page.

```bash  theme={null}
# Verify the default page
curl localhost
# → Welcome to nginx!
```

Removing the `default` site will cause `localhost` to refuse connections until you enable at least one server block.

```bash  theme={null}
# Disable default configuration
rm /etc/nginx/sites-enabled/default

# Reload Nginx to apply changes
nginx -s reload

# Confirm removal
curl localhost
# → curl: (7) Failed to connect to localhost port 80: Connection refused
```

***

## 2. Create Site Configurations

We’ll copy the default config twice—once for each domain—and then adjust the `root` and `server_name`.

### 2.1 example1.com

```bash  theme={null}
# List available configs
ls -l /etc/nginx/sites-available

# Copy default to example1
cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example1
```

Edit `/etc/nginx/sites-available/example1`:

```nginx  theme={null}
# /etc/nginx/sites-available/example1
server {
    listen 80;
    server_name www.example1.com;
    root /var/www/example1;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 2.2 example2.com

```bash  theme={null}
# Duplicate example1 for example2
cp /etc/nginx/sites-available/example1 /etc/nginx/sites-available/example2
```

Update `/etc/nginx/sites-available/example2`:

```nginx  theme={null}
# /etc/nginx/sites-available/example2
server {
    listen 80;
    server_name www.example2.com;
    root /var/www/example2;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```



## 3. Set Up Document Roots and Index Files

Create directories for each site and add a simple `index.html`:

```bash  theme={null}
mkdir -p /var/www/example1 /var/www/example2

cat << 'EOF' > /var/www/example1/index.html
<h1>Example 1</h1>
EOF

cat << 'EOF' > /var/www/example2/index.html
<h1>Example 2</h1>
EOF

# Ensure Nginx can read the files
chown -R www-data:www-data /var/www/example1 /var/www/example2
```



## 4. Enable Sites and Reload Nginx

Create symbolic links in `sites-enabled`, then test and reload.

```bash  theme={null}
# Enable both sites
ln -s /etc/nginx/sites-available/example1 /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/example2 /etc/nginx/sites-enabled/

# Test configuration
nginx -t
# Reload Nginx
nginx -s reload
```



## 5. Verify Virtual Host Routing

Simulate browser requests using `curl` with custom `Host` headers:

```bash  theme={null}
curl --header "Host: www.example1.com" localhost
curl --header "Host: www.example2.com" localhost
curl --header "Host: unknown.com" localhost
# → <h1>Example 1</h1>  (first enabled server block)
```



## 6. Optional: Single Combined Config

You may merge both server blocks into one file, though we recommend keeping them separate.

```nginx  theme={null}
# /etc/nginx/sites-available/example
server {
    listen 80;
    server_name www.example1.com;
    root /var/www/example1;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen 80;
    server_name www.example2.com;
    root /var/www/example2;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable and reload:

```bash  theme={null}
ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled/
nginx -t && nginx -s reload
```

Keeping separate files in `sites-available` makes it simpler to enable/disable individual domains without affecting others.

## 7. Best Practices

| Practice                     | Benefit                                                                                |
| ---------------------------- | -------------------------------------------------------------------------------------- |
| Separate config per domain   | Isolate changes and minimize risk to other sites.                                      |
| Use descriptive file names   | Quickly identify which domain each configuration serves.                               |
| Regularly test configuration | Run `nginx -t` before reloading to catch syntax errors.                                |
| Limit server-wide directives | Keep domain-specific settings in individual server blocks, not in global `nginx.conf`. |


## References

* [Nginx Server Blocks (Virtual Hosts)](https://docs.nginx.com/nginx/admin-guide/server-blocks/)
* [Ubuntu Nginx Documentation](https://ubuntu.com/server/docs/web-servers-nginx)
* [Curl Manual](https://curl.se/docs/manpage.html)
* [Nginx Try Files Documentation](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)


> ## Documentation Index
> Fetch the complete documentation index at: https://notes.kodekloud.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Demo Configure Multiple Sites

This explains how to host multiple domains on a single Nginx server using virtual hosts.

In this you’ll learn how to host two distinct domains **[www.example1.com](http://www.example1.com)** and \*\*[www.example2.com\*\*—on](http://www.example2.com** on) a single Nginx server. Incoming HTTP requests are routed based on the `Host` header, matching the `server_name` directive within each server block (often called *virtual hosts*).

## 1. Remove the Default Site

Before adding your own configurations, disable the default Nginx welcome page.

```bash  theme={null}
# Verify the default page
curl localhost
# → Welcome to nginx!
```

Removing the `default` site will cause `localhost` to refuse connections until you enable at least one server block.

```bash  theme={null}
# Disable default configuration
rm /etc/nginx/sites-enabled/default

# Reload Nginx to apply changes
nginx -s reload

# Confirm removal
curl localhost
# → curl: (7) Failed to connect to localhost port 80: Connection refused
```

## 2. Create Site Configurations

We’ll copy the default config twice—once for each domain—and then adjust the `root` and `server_name`.

### 2.1 example1.com

```bash  theme={null}
# List available configs
ls -l /etc/nginx/sites-available

# Copy default to example1
cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example1
```

Edit `/etc/nginx/sites-available/example1`:

```nginx  theme={null}
# /etc/nginx/sites-available/example1
server {
    listen 80;
    server_name www.example1.com;
    root /var/www/example1;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 2.2 example2.com

```bash  theme={null}
# Duplicate example1 for example2
cp /etc/nginx/sites-available/example1 /etc/nginx/sites-available/example2
```

Update `/etc/nginx/sites-available/example2`:

```nginx  theme={null}
# /etc/nginx/sites-available/example2
server {
    listen 80;
    server_name www.example2.com;
    root /var/www/example2;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

## 3. Set Up Document Roots and Index Files

Create directories for each site and add a simple `index.html`:

```bash  theme={null}
mkdir -p /var/www/example1 /var/www/example2

cat << 'EOF' > /var/www/example1/index.html
<h1>Example 1</h1>
EOF

cat << 'EOF' > /var/www/example2/index.html
<h1>Example 2</h1>
EOF

# Ensure Nginx can read the files
chown -R www-data:www-data /var/www/example1 /var/www/example2
```

## 4. Enable Sites and Reload Nginx

Create symbolic links in `sites-enabled`, then test and reload.

```bash  theme={null}
# Enable both sites
ln -s /etc/nginx/sites-available/example1 /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/example2 /etc/nginx/sites-enabled/

# Test configuration
nginx -t
# Reload Nginx
nginx -s reload
```

## 5. Verify Virtual Host Routing

Simulate browser requests using `curl` with custom `Host` headers:

```bash  theme={null}
curl --header "Host: www.example1.com" localhost
curl --header "Host: www.example2.com" localhost
curl --header "Host: unknown.com" localhost
# → <h1>Example 1</h1>  (first enabled server block)
```

## 6. Optional: Single Combined Config

You may merge both server blocks into one file, though we recommend keeping them separate.

```nginx  theme={null}
# /etc/nginx/sites-available/example
server {
    listen 80;
    server_name www.example1.com;
    root /var/www/example1;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen 80;
    server_name www.example2.com;
    root /var/www/example2;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable and reload:

```bash  theme={null}
ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled/
nginx -t && nginx -s reload
```

Keeping separate files in `sites-available` makes it simpler to enable/disable individual domains without affecting others.

## 7. Best Practices

| Practice                     | Benefit                                                                                |
| ---------------------------- | -------------------------------------------------------------------------------------- |
| Separate config per domain   | Isolate changes and minimize risk to other sites.                                      |
| Use descriptive file names   | Quickly identify which domain each configuration serves.                               |
| Regularly test configuration | Run `nginx -t` before reloading to catch syntax errors.                                |
| Limit server-wide directives | Keep domain-specific settings in individual server blocks, not in global `nginx.conf`. |
