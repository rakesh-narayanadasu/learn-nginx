# URL Redirect Rewrite

This explains how to use Nginx return and rewrite directives for URL redirection and transformation.

In this you’ll learn how to use Nginx’s `return` (redirect) and `rewrite` directives to forward requests, enforce HTTPS, and transform URLs on the fly. Whether you’re migrating a domain, cleaning up paths, or preserving legacy links, these patterns will help you maintain SEO value and user experience.

## Why Redirect vs Rewrite?

* **Redirect (`return`)** issues an HTTP status code (e.g., 301) back to the client and changes what appears in their browser’s address bar.
* **Rewrite** silently alters the URI before Nginx processes it, keeping the user’s URL intact.

Use redirects when you want search engines and clients to update bookmarks. Use rewrites to preserve a clean URL structure without exposing internal file paths.


## 1. Redirecting with `return`

### 1.1. Entire Domain Redirect

To forward every request from one domain to another:

```nginx  theme={null}
server {
    listen       80;
    server_name  honda.cars.com;

    return 301 https://cars.honda.com$request_uri;

    root  /var/www/example.com/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
>  Use `301` for permanent redirects to transfer SEO equity. For temporary redirects, replace `301` with `302`.

### 1.2. HTTP → HTTPS Redirect

Force all HTTP traffic to HTTPS:

```nginx  theme={null}
server {
    listen       80;
    server_name  honda.cars.com;

    return 301 https://$host$request_uri;
}

server {
    listen       443 ssl;
    server_name  honda.cars.com;

    ssl_certificate     /etc/ssl/certs/honda.cars.com.pem;
    ssl_certificate_key /etc/ssl/certs/honda.cars.com-key.pem;

    root  /var/www/;
}
```

Here, `$host` becomes the requested domain name, and `$request_uri` includes the full path and query string.

### 1.3. Single-Page Redirect

For path-specific forwarding, wrap only that URI:

```nginx  theme={null}
server {
    listen       80;
    server_name  honda.cars.com;

    root  /var/www/example.com/html;
    index index.html;

    location /civic-type-r {
        return 301 https://cars.honda.com$request_uri;
    }
}
```

## 2. Common HTTP Status Codes

Inspect redirects and responses in your browser’s DevTools (Network tab):

| Status Code | Meaning                    |
| ----------- | -------------------------- |
| 200         | OK                         |
| 301         | Moved Permanently          |
| 302         | Found (Temporary Redirect) |
| 400         | Bad Request                |
| 401         | Unauthorized               |
| 403         | Forbidden                  |
| 404         | Not Found                  |
| 500         | Internal Server Error      |
| 502         | Bad Gateway                |
| 503         | Service Unavailable        |

## 3. Transforming URLs with `rewrite`

Rewrites let you modify incoming URLs before Nginx searches for files or forwards to upstream servers—ideal for cleanup rules or legacy support.

Use rewrites to convert long URIs into friendly, concise paths:

### 3.1. Simple Rewrite Example

Map `/sports-car-civic-type-r` → `/type-r`:

```nginx  theme={null}
server {
    listen       80;
    server_name  honda.cars.com;

    root  /var/www/example.com/html;
    index index.html;

    rewrite ^/sports-car-civic-type-r$ /type-r permanent;
}
```

### 3.2. Rewriting Directory Paths

When you rename a folder from `/pics` → `/images`, keep existing links functional:

```bash  theme={null}
# Before rename
$ tree
|-- 50x.html
|-- index.html
`-- pics
    |-- accord.jpg
    |-- civic.jpg
    `-- type-r.jpg

# After rename
$ tree
|-- 50x.html
|-- index.html
|-- images
|   |-- accord.jpg
|   |-- civic.jpg
|   `-- type-r.jpg
`-- pics
    |-- accord.jpg
    |-- civic.jpg
    `-- type-r.jpg
```

```nginx  theme={null}
server {
    listen       80;
    server_name  honda.cars.com;

    root  /var/www/example.com/html;
    index index.html;

    location /pics {
        rewrite ^/pics/(.*)$ /images/$1 permanent;
    }
}
```

Visitors requesting `/pics/type-r.jpg` will be served `/images/type-r.jpg` without broken links.

>  Misconfigured regex or overlapping `rewrite` rules can cause redirect loops. Test your configuration with `nginx -t` and in a staging environment first.

### 3.3. Regex Cheat Sheet

| Symbol | Description                         | Example                  |
| ------ | ----------------------------------- | ------------------------ |
| ^      | Start of string                     | `^/old` matches `/old`   |
| \$     | End of string                       | `/page$` matches `/page` |
| .      | Any single character                | `a.b` matches `acb`      |
| \*     | Zero or more of the preceding token | `.*` captures anything   |
| \[]    | Character class                     | `[a-z]`                  |
| ()     | Capture group                       | `(.*)`                   |

For interactive testing, try [regex101](https://regex101.com).

# Demo Configure URL Redirect

In this you will learn how to enforce HTTPS by redirecting all HTTP requests (port 80) to HTTPS (port 443) using NGINX. Demonstrate this on a simple diner app currently served over HTTP.

>  Before you begin, ensure NGINX is installed and your TLS certificates (`.pem` and `.key`) are available in `/etc/ssl/certs/`.

## 1. Verify the Current Setup

1. Open the diner app in your browser at `http://localhost`.
2. Check your firewall rules:

```bash  theme={null}
ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
```

Port 443 is not yet allowed, so accessing HTTPS returns a 502 Gateway error:

3. Allow HTTPS traffic:

```bash  theme={null}
ufw allow 443/tcp
ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)
```

Your firewall now permits port 443, but NGINX isn’t listening there yet.

***

## 2. Review Existing NGINX Configuration

List enabled sites:

```bash  theme={null}
ls -l /etc/nginx/sites-enabled
total 4
lrwxrwxrwx 1 root root 32 Feb  7 00:51 diner -> /etc/nginx/sites-available/diner
```

Open `/etc/nginx/sites-available/diner`—it currently listens only on HTTP:

```nginx  theme={null}
server {
    listen 80;
    server_name diner.com;

    root /var/www/diner;
    index index.html index.htm index.nginx-debian.html;

    location / {
        # First attempt to serve request as file,
        # then as directory, then return a 404.
        try_files $uri $uri/ =404;
    }
}
```

## 3. Create the HTTPS Configuration

Create (or edit) `/etc/nginx/sites-available/diner-https` with **two** server blocks:

```nginx  theme={null}
server {
    listen 80;
    server_name diner.com;

    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name diner.com;

    # SSL certificates (already present on the system)
    ssl_certificate     /etc/ssl/certs/diner.com.pem;
    ssl_certificate_key /etc/ssl/certs/diner.com-key.pem;

    root /var/www/diner;
    index index.html index.htm index.nginx-debian.html;

    location / {
        # First attempt to serve request as file,
        # then as directory, then return a 404.
        try_files $uri $uri/ =404;
    }
}
```

test using below command:
```
curl --head -H "Host: diner.com" localhost

# or

curl -I -H "Host: diner.com" localhost
```

| Server Block   | Purpose                                              | Port |
| -------------- | ---------------------------------------------------- | ---- |
| HTTP → HTTPS   | Permanent redirect (`301`) to the same URI over TLS  | 80   |
| HTTPS with SSL | Serves encrypted content using provided certificates | 443  |

***

## 4. Enable and Test the New Configuration

1. Disable the old site and enable the new one:
   ```bash  theme={null}
   sudo rm /etc/nginx/sites-enabled/diner
   sudo ln -s \
       /etc/nginx/sites-available/diner-https \
       /etc/nginx/sites-enabled/diner-https
   ```

2. Test NGINX syntax and reload:
   ```bash  theme={null}
   nginx -t
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successful

   nginx -s reload
   ```

>  If NGINX fails to reload, check for syntax errors in all files under `/etc/nginx/` and confirm your certificate paths are correct.

3. Verify the redirect and HTTPS response:

   * **HTTP → HTTPS redirect:**
     ```bash  theme={null}
     curl -I http://localhost
     HTTP/1.1 301 Moved Permanently
     Location: https://localhost/
     Server: nginx/1.18.0 (Ubuntu)
     ```

   * **Serving content over HTTPS:**
     ```bash  theme={null}
     curl -I https://localhost --insecure
     HTTP/1.1 200 OK
     Server: nginx/1.18.0 (Ubuntu)
     ```

All HTTP requests on port 80 now automatically redirect to HTTPS on port 443, ensuring encrypted connections.



## Links and References

* [NGINX HTTP to HTTPS Redirect](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#return)
* [UFW Documentation](https://help.ubuntu.com/community/UFW)
* [NGINX Official Documentation](https://nginx.org/en/docs/)

# Demo Configure URL Rewrite

In this you’ll learn how to use Nginx’s `rewrite` directive to transparently redirect an old URL path to a new one. This approach preserves existing bookmarks and SEO value by issuing a permanent (301) redirect.

## Scenario

Your site `example.com` originally served images from:

```text  theme={null}
https://example.com/images/pic01.jpg
```

After renaming the folder from `/images` to `/pics`, you want to redirect all traffic from `/images/...` to `/pics/...` without breaking links. A suitable rewrite rule is:

```nginx  theme={null}
rewrite ^/images/(.*)$ /pics/$1 permanent;
```

This captures anything after `/images/` and issues a 301 redirect to `/pics/...`.

***

## Prerequisites

* A running Nginx server (≥1.14)
* SSH access to the server
* Root or sudo privileges
* A staging environment for testing

>  Always test changes in a staging environment before deploying to production to avoid downtime.

## 1. Prepare the Content Directory

On the Nginx server, duplicate the `images/` folder under the document root (`/var/www/html`):

```bash  theme={null}
cd /var/www/html
cp -R images/ pics/
```

Verify both directories exist:

```bash  theme={null}
ls -l /var/www/html
```

Expected output:

```plaintext  theme={null}
drwxr-xr-x 1 root root 4096 Feb 18 16:14 images/
drwxr-xr-x 1 root root 4096 Feb 18 16:29 pics/
```

Both directories should contain the same image files:

```bash  theme={null}
ls -l images/
```

```plaintext  theme={null}
pic01.jpg  pic02.jpg  …  pic15.jpg
```

## 2. Test Before Rewriting

Ensure Nginx is running and serving files without any rewrite rules:

1. Open your browser and visit:\
   `http://example.com/images/pic10.jpg`
2. You should see the image load directly from `/images/pic10.jpg`.

## 3. Review Initial Nginx Configuration

Open your server block configuration for `example.com` (commonly in `/etc/nginx/sites-available/example`):

```nginx  theme={null}
server {
    listen 80;
    server_name example.com;
    root /var/www/html;

    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Validate and reload Nginx:

```bash  theme={null}
sudo nginx -t
sudo systemctl reload nginx
```

## 4. Add the Rewrite Directive

Edit the same `server` block and insert the `rewrite` rule **before** `try_files`:

```nginx  theme={null}
server {
    listen 80;
    server_name example.com;
    root /var/www/html;

    index index.html index.htm index.nginx-debian.html;

    location / {
        # Redirect all /images/... requests to /pics/...
        rewrite ^/images/(.*)$ /pics/$1 permanent;

        # Then attempt to serve the request
        try_files $uri $uri/ =404;
    }
}
```

Save the file, then test and reload:

```bash  theme={null}
sudo nginx -t
sudo systemctl reload nginx
```
>  A typo in the regex or placement of `rewrite` can lead to redirect loops. Double-check the pattern and test thoroughly.

## 5. Verify the Rewrite

To avoid cached redirects, open an incognito/private browser window and navigate to:

```text  theme={null}
http://example.com/images/pic08.jpg
```

You should be permanently redirected (301) to:

```text  theme={null}
http://example.com/pics/pic08.jpg
```

and see the image served from the new location.

## Quick Reference

| Step | Action                               | Command / Pattern                            |
| ---- | ------------------------------------ | -------------------------------------------- |
| 1    | Duplicate directory                  | `cp -R images/ pics/`                        |
| 2    | Verify directories                   | `ls -l /var/www/html`                        |
| 3    | Review current Nginx server block    | `/etc/nginx/sites-available/example`         |
| 4    | Add rewrite rule before `try_files`  | `rewrite ^/images/(.*)$ /pics/$1 permanent;` |
| 5    | Test the redirect in private browser | Visit `/images/pic08.jpg`                    |

## Conclusion

Using Nginx’s `rewrite` directive with regular expressions allows seamless URL remapping and preserves SEO by issuing 301 redirects. Always:

* Validate your Nginx configuration (`nginx -t`)
* Reload Nginx to apply changes
* Test rewrites in a staging environment

## Links and References

* [Nginx Rewrite Module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html)
* [Regular Expression HOWTO](https://www.regular-expressions.info/)
* [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
