# HTTP Headers

HTTP headers are metadata exchanged between a client (e.g., your browser) and a web server. Much like the information on an envelope, headers indicate how to process the payload and which instructions to follow.

## Request–Response Flow

When you navigate to a website:

1. Your browser sends an HTTP request with **request headers** describing the desired resource.
2. The server processes the request and replies with an HTTP response, including **response headers** that describe the returned content.

## Anatomy of an HTTP Header

Each header line follows the format `Key: Value`. For example:

```http  theme={null}
GET / HTTP/2
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT) Firefox/103.0
Accept: text/html
Accept-Language: en-US
Connection: keep-alive
Cache-Control: no-cache
```

* **Host**: Specifies the target domain (`example.com`).
* **User-Agent**: Provides details about the client software.
* **Accept**: Indicates which content types the client can process.
* **Cache-Control**: Instructs caching behavior.

>  In HTTP/2, pseudo-headers (like `:method`, `:path`) precede regular headers and always start with `:`.

## Types of HTTP Headers

HTTP headers are grouped by their roles. Below is a summary table.

| Header Category | Purpose                              | Common Example                 |
| --------------- | ------------------------------------ | ------------------------------ |
| General         | Used in both requests and responses  | `Connection`, `Cache-Control`  |
| Request         | Sent by clients to servers           | `User-Agent`, `Accept`         |
| Response        | Sent by servers back to clients      | `Content-Type`, `Server`       |
| Security        | Mitigate web vulnerabilities         | `Content-Security-Policy`      |
| Authentication  | Verify client identity               | `Authorization`                |
| Caching         | Control resource caching             | `Expires`, `ETag`              |
| CORS            | Enable cross-origin resource sharing | `Access-Control-Allow-Origin`  |
| Proxy           | Convey client info through proxies   | `X-Forwarded-For`, `X-Real-IP` |
| Custom          | Application-specific metadata        | `X-Custom-Header`              |

### General Headers

General headers carry metadata applicable to both requests and responses, such as connection management and caching directives.

```
Request URL     https://pep8.org/
Request Method  GET
Status Code     304 Not Modified
Remote Address  13.215.239.219:443
Referrer Policy strict-origin-when-cross-origin
```

### Request Headers

Clients send these headers to the server:

```http  theme={null}
:authority: google.com
:method: GET
:path: /
:scheme: https
Accept: text/html,application/xhtml+xml
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Cache-Control: max-age=0
Cookie: session=abcd1234
Referer: https://www.google.com/
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)
```

* **Accept**: Media types the client prefers.
* **User-Agent**: Browser and OS identification.
* **Accept-Language**: Preferred languages.
* **Cookie**: Session data stored by the browser.

### Response Headers

Servers include these headers in replies:

```http  theme={null}
Age: 387887
Cache-Control: max-age=3600
Content-Encoding: br
Content-Type: text/html
Date: Fri, 17 Jan 2025 18:44:46 GMT
Server: cloudflare
Vary: Accept-Encoding
X-Cache: HIT
```

* **Content-Type**: Media type of the response body.
* **Cache-Control**: Caching policy instructions.
* **Server**: Server software identifier.
* **X-Cache**: Indicates cache hits or misses.

## Security Headers

Security headers guard against common web threats like XSS, clickjacking, and mixed-content issues.

```http  theme={null}
alt-svc: clear
cache-control: no-cache, no-store, max-age=0, must-revalidate
content-encoding: gzip
content-security-policy:
  script-src 'nonce-6tCRsH62nR2cX-vHxm5A' 'strict-dynamic'
  https: 'unsafe-eval'; object-src 'none'; base-uri 'self';
  report-uri https://mail.google.com/mail/cspreport
content-type: text/html; charset=utf-8
strict-transport-security: max-age=1088400; includeSubDomains
x-frame-options: SAMEORIGIN
x-xss-protection: 0
```
>  Improperly configured security headers can break site functionality or leave vulnerabilities. Always test in staging before deploying.


## Authentication Headers

Authenticate requests using credentials or tokens:

```http  theme={null}
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

* **Basic**: Base64-encoded `username:password`.
* **Bearer**: Token-based (e.g., OAuth 2.0, JWT).

## Caching Headers

Manage how clients and proxies cache responses:

```http  theme={null}
Cache-Control: max-age=0, must-revalidate
Expires: Wed, 21 Oct 2025 07:28:00 GMT
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
X-Cache: Miss from cloudfront
```

* **Cache-Control**: Fine-grained caching instructions.
* **Expires**: Absolute expiry date/time.
* **ETag**: Entity tag for version validation.

## CORS Headers

Enable controlled cross-origin requests to protect resources:

```http  theme={null}
Access-Control-Allow-Origin: https://petexchangehk-frontend.vercel.app
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: content-type, authorization
Access-Control-Allow-Credentials: true
```

>  Browsers enforce CORS; servers must explicitly allow desired origins and methods.

## Proxy Headers

When requests pass through load balancers or reverse proxies, these headers preserve client information:

```http  theme={null}
Host: web:8001
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64)
X-Forwarded-For: 192.168.122.4
X-Forwarded-Host: web.domain.com
X-Forwarded-Proto: https
X-Forwarded-Server: 172.19.0.5
X-Real-IP: 192.168.122.4
```

* **X-Forwarded-For**: Original client IP through multiple proxies. To pass the original IP when the request goes through multiple proxies.
* **X-Forwarded-Host**: Original `Host` header.
* **X-Forwarded-Proto**: Protocol used by the client. To get the original protocol request.
* **X-Real-IP**: Direct client IP (single proxy scenario), is to get the original IP when the request goes through one proxy.

## Custom Headers

Define your own headers for feature flags, tracing, or legacy integrations:

```http  theme={null}
GET /api HTTP/1.1
Host: example.com
User-Agent: curl/8.1.2
Accept: */*
X-Custom-Header1: Value1
```

Custom headers must begin with `X-` (for legacy) or a vendor namespace.

## Nginx Built-in Variables

Nginx uses `$`-prefixed variables to access request and server data dynamically.

Common variables:

* `$remote_addr`: THe IP address of the client making the request
* `$remote_port`: Used to get the port of the client making the request
* `$server_addr`: IP address of the Nginx server handling the request
* `$request`: Full request line including method, eg., GET /index.html HTTP1.1
* `$url`: Includes the current path without ant arguments eg., /index.html
* `$host`: Host header value
* `$scheme`: Protocol (`http` or `https`)
* `$request_uri`: Full request URI including query string
* `$request_method`: Usually GET or POST
* `$args`: Used for the query strings eg., id=1234&name=example
* `$http_<header_name>`: Accept any HTTP header eg., $http_user_agent
* `$http_referer`: The value of the Referer, indicating the URL of the page that made the request
* `$status`: Represents the response code sent from the web server eg., 200, 404, 301
* `$upstream_addr`: For the IP address or hostname of the upstream that handled the request
* `$scheme`: For the protocol eg., http or https
* `$host`: To pass the origin host eg., www.google.com
* `$ssl_protocol`: Used for the TLS connection that was used eg., TLSv1.2, TLSv1.3
* `$ssl_cipher`: Used to pass in the code that was in the connection eg., AES128-SHA:AES256:SHA
* `$http_cookie`: The cookie header sent by the client

Combine variables to reconstruct the full URL:

```text  theme={null}
$scheme://$host$request_uri
```

## Example Nginx Configuration

Below is a sample Nginx `server` block that sets security headers and proxies traffic to an upstream backend:

```nginx  theme={null}
server {
    listen       80;
    server_name  honda.cars.com;

    root   /var/www/honda.cars.com/html;
    index  index.html;

    # Security response headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
    add_header X-Frame-Options "SAMEORIGIN";
    add_header Content-Security-Policy "default-src 'self'";
    add_header Referrer-Policy "origin";

    location / {
        # Preserve original client headers
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass         http://backend_upstream;
    }
}
```

* **add\_header**: Appends headers to outbound responses.
* **proxy\_set\_header**: Customizes headers sent to upstream servers.

## Links and References

* [MDN Web Docs: HTTP Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
* [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)


# Demo HTTP Headers

In this we’ll configure an NGINX server to send custom HTTP headers, improve security, and then turn it into a load balancer that forwards client details to Apache backends.

First, let’s inspect the default response from our site before any custom headers are added.

## Testing the Default Response

1. Map `example.com` to `127.0.0.1` by editing `/etc/hosts`:
   ```bash  theme={null}
   cat /etc/hosts
   ```
   ```text  theme={null}
   127.0.0.1   localhost
   ::1         localhost ip6-localhost ip6-loopback
   ...
   127.0.0.1   example.com
   ```

2. Curl over HTTP to see the redirect:
   ```bash  theme={null}
   curl -I http://example.com
   ```
   ```text  theme={null}
   HTTP/1.1 301 Moved Permanently
   Server: nginx/1.18.0 (Ubuntu)
   Date: Wed, 12 Feb 2025 19:24:14 GMT
   ...
   ```

3. Fetch only headers from HTTPS:
   ```bash  theme={null}
   curl --head https://example.com
   ```
   ```text  theme={null}
   HTTP/1.1 200 OK
   Server: nginx/1.18.0 (Ubuntu)
   Date: Wed, 12 Feb 2025 19:24:14 GMT
   Content-Type: text/html
   Content-Length: 8710
   Last-Modified: Wed, 12 Feb 2025 18:42:19 GMT
   Connection: keep-alive
   ETag: "67ace8b-2206"
   Accept-Ranges: bytes
   ```

4. Open your browser’s developer tools (Network tab) and refresh. You’ll see only default headers like `Server`, `Date`, etc.

## Adding Security Headers

Edit your HTTPS server block (e.g., `/etc/nginx/sites-available/example-https`):

```nginx  theme={null}
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/ssl/certs/example.com.pem;
    ssl_certificate_key /etc/ssl/certs/example.com-key.pem;

    root  /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Within the `server { ... }` listening on port 443, add these headers:

```nginx  theme={null}
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
    add_header X-Frame-Options "SAMEORIGIN";
    add_header Content-Security-Policy "default-src 'self'";
    add_header Referrer-Policy "origin";
```

The complete HTTPS block:

```nginx  theme={null}
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/ssl/certs/example.com.pem;
    ssl_certificate_key /etc/ssl/certs/example.com-key.pem;

    root  /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
    add_header X-Frame-Options "SAMEORIGIN";
    add_header Content-Security-Policy "default-src 'self'";
    add_header Referrer-Policy "origin";

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Always test your configuration before reloading:

```bash  theme={null}
nginx -t
nginx -s reload
```


Verify with curl:

```bash  theme={null}
curl --head https://example.com
```

```text  theme={null}
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Wed, 12 Feb 2025 19:28:15 GMT
Content-Type: text/html
Content-Length: 8710
Last-Modified: Wed, 12 Feb 2025 18:42:19 GMT
Connection: keep-alive
ETag: "67acebb-2206"
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Frame-Options: SAMEORIGIN
Content-Security-Policy: default-src 'self'
Referrer-Policy: origin
Accept-Ranges: bytes
```

### Summary of Security Headers

| Header                    | Purpose                                  |
| ------------------------- | ---------------------------------------- |
| Strict-Transport-Security | Enforce HTTPS connections                |
| X-Frame-Options           | Prevent hackers from being able to embed your website into theirs to trick people.                     |
| Content-Security-Policy   | Restrict resource loading to same origin, protects your website against cross-site scripting (XSS). |
| Referrer-Policy           | Control Referer header information, used to send other websites information about where the request came from.       |


## Configuring NGINX as a Load Balancer

Now we’ll distribute traffic to two Apache backends and forward client information.

### 1. Define the Upstream

Add at the top of `nginx.conf` or inside your site file:

```nginx  theme={null}
upstream example {
    server node01:443;
    server node02:443;
}
```

### 2. Initial Proxy Test (No Headers)

Replace the `location /` block:

```nginx  theme={null}
location / {
    proxy_pass https://example;
}
```

Reload NGINX and refresh. You’ll see alternating “Node01” and “Node02”, but Apache logs will record only the load balancer’s IP.

### 3. View Apache Access Logs

```bash  theme={null}
# On node01
tail -f /var/log/apache2/access.log
```

You’ll see entries like:

```text  theme={null}
example.com:443 127.0.0.1 - - [12/Feb/2025:19:30:42 +0000] "GET / HTTP/1.0" 200 3621 ... "curl/7.81.0"
```

## Passing Proxy Headers

Update the same `location /` block to forward client data:

```nginx  theme={null}
location / {
    proxy_set_header X-Real-IP           $remote_addr;
    proxy_set_header Host                $host;
    proxy_set_header X-Forwarded-For     $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto   $scheme;

    proxy_pass https://example;
}
```

Reload NGINX:

```bash  theme={null}
nginx -t
nginx -s reload
```

After changing log formats, always test Apache before restarting:

```bash  theme={null}
apachectl -t
systemctl restart apache2
```

### 4. Update Apache Log Format (on node01)

In `/etc/apache2/apache2.conf` or your vhost:

```apache  theme={null}
LogFormat "%v:%p \"%{X-Real-IP}i\" \"%{X-Forwarded-For}i\" \"%{X-Forwarded-Proto}i\" %h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

Reload Apache:

```bash  theme={null}
apachectl -t
systemctl restart apache2
```

### 5. Compare Logs

* **node02** (unchanged): logs show only the load balancer’s IP.
* **node01**: logs include:
  * `X-Real-IP`: immediate client IP
  * `X-Forwarded-For`: full proxy chain
  * `X-Forwarded-Proto`: original protocol

Example:

```text  theme={null}
example.com:443 "192.231.70.4" "174.0.252.84, 34.117.152.159, 169.254.169.126, 192.168.1.144, 192.231.70.4" "https" 192.231.70.4 - - [12/Feb/2025:19:37:22 +0000] "GET / HTTP/1.0" 200 3621 "https://..." "Mozilla/5.0..."
```

***

## Recap

1. Tested default NGINX headers.
2. Added security headers (`HSTS`, `X-Frame-Options`, `CSP`, `Referrer-Policy`).
3. Configured NGINX as an SSL-terminating load balancer.
4. Forwarded proxy headers (`X-Real-IP`, `Host`, `X-Forwarded-For`, `X-Forwarded-Proto`).
5. Updated Apache `LogFormat` to capture these headers.

From here, explore authentication, caching, compression, and more.

## Links and References

* [NGINX Documentation](https://nginx.org/en/docs/)
* [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
* [HTTP Header Security Guide (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
