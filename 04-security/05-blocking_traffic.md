# Blocking Traffic

Hackers can steal data, distribute spyware or ransomware, and even bring down entire websites. While legitimate bots crawl your pages for search indexing, malicious bots may spam forms, scrape content, or post fake reviews.

How to block unwanted traffic—whether it’s individual IPs, bots, or entire network ranges to protect your site.

## Restricting Access by IP in Nginx

Nginx’s HTTP Access module uses two directives—`allow` and `deny`—to control client access at the server or location level.

| Directive | Description                              | Example                 |
| --------- | ---------------------------------------- | ----------------------- |
| allow     | Grants access to a specific IP or subnet | `allow 192.168.1.0/24;` |
| deny      | Blocks access from an IP or subnet       | `deny 203.0.113.0/24;`  |

>  CIDR notation (`/32` for a single IPv4, `/24` for 256 addresses) lets you include entire IP ranges in one rule.

### Example: Allow Only Two IPs, Block Everything Else

```nginx  theme={null}
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example.com/html;
    index index.html;

    # Permit two individual IPs
    allow 192.168.1.100/32;
    allow 174.0.252.8/32;
    deny all;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

* `/32` mask = single IPv4 address
* `deny all;` blocks every other client

### Example: Deny a Subnet Globally, Allow a Subnet to /admin

```nginx  theme={null}
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example.com/html;
    index index.html;

    # Block the entire 203.0.113.0/24 network site-wide
    deny 203.0.113.0/24;

    location /admin {
        # Only allow 174.0.252.0/24 to access /admin
        allow 174.0.252.0/24;
        deny all;
        try_files $uri $uri/ =404;
    }
}
```

## Why Manual IP Blocking Falls Short

Maintaining long lists of `deny` rules is error-prone attackers simply switch IPs. For automated protection, consider using [Fail2Ban](https://www.fail2ban.org/).

>  Never rely on static IP denylists alone; automated tools help you stay ahead of rotating attackers.

Fail2Ban monitors logs for repeated failures or suspicious patterns, then automatically bans the offending IP for a configurable duration.

```bash  theme={null}
$ sudo tail -f /var/log/fail2ban.log
2024-08-10 19:26:27,469 fail2ban.jail   [7786]: INFO    Creating new jail 'ssh'
2024-08-10 19:26:27,473 fail2ban.filter [7786]: INFO    maxRetry: 2
...
2024-08-10 19:27:40,771 fail2ban.actions: NOTICE  [ssh] Ban 192.168.8.131
```

### Installing Fail2Ban

On Ubuntu/Debian:

```bash  theme={null}
sudo apt update
sudo apt install fail2ban
```

On Red Hat/CentOS:

```bash  theme={null}
sudo yum install fail2ban
```

### Configuring Jails

Copy the default `jail.conf` to `jail.local` and enable only the jails you need:

```bash  theme={null}
cd /etc/fail2ban
sudo cp jail.conf jail.local
sudo vim jail.local
```

#### Protecting Nginx HTTP Authentication

```ini  theme={null}
[nginx-http-auth]
enabled  = true
port     = http,https
filter   = nginx-http-auth
logpath  = /var/log/nginx/access.log
maxretry = 3
findtime = 600
bantime  = 600
```

#### Blocking Bad Bots

```ini  theme={null}
[nginx-badbots]
enabled  = true
port     = http,https
filter   = nginx-badbots
logpath  = /var/log/nginx/access.log
maxretry = 1
bantime  = 48h
```

#### Rate Limiting Excessive Requests

```ini  theme={null}
[nginx-limit-req]
enabled  = true
port     = http,https
filter   = nginx-limit-req
logpath  = /var/log/nginx/access.log
maxretry = 10
findtime = 3600
bantime  = 24h
```

### Built-in Filters

Filters are stored in `/etc/fail2ban/filter.d`. Common patterns include Nginx, SSH, and more:

```bash  theme={null}
$ ls -l /etc/fail2ban/filter.d
-rw-r--r-- 1 root root  474 Nov  9 2022 nginx-bad-request.conf
-rw-r--r-- 1 root root  740 Nov  9 2022 nginx-botsearch.conf
-rw-r--r-- 1 root root 1048 Nov  9 2022 nginx-http-auth.conf
-rw-r--r-- 1 root root 1513 Nov  9 2022 nginx-limit-req.conf
```

To customize, edit or add new `.conf` files under this directory.

### Checking Status and Unbanning

View a jail’s status:

```bash  theme={null}
sudo fail2ban-client status nginx-http-auth
```

Sample output:

```text  theme={null}
Status for the jail: nginx-http-auth
|- Filter
|  |- Currently failed: 0
|  |- Total failed: 3
|  `- File list: /var/log/nginx/access.log
`- Actions
   |- Currently banned: 1
   `- Banned IP list: 108.172.85.62
```

Unban an IP if needed:

```bash  theme={null}
sudo fail2ban-client set nginx-http-auth unbanip 108.172.85.62
```

Fail2Ban operates independently of Nginx or SSH, so there’s no need to restart those services after making changes.

These techniques IP-based restrictions in Nginx and automated bans with Fail2Ban can significantly reduce brute-force and bot-driven attacks. In containerized platforms like [Kubernetes](https://kubernetes.io/), you may need different integrations, but the core ideas carry over to any host-based environment.

## Links and References

* [Nginx Documentation](https://nginx.org/en/docs/)
* [Fail2Ban Official Site](https://www.fail2ban.org/)
* [CIDR Cheatsheet](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)
* [Kubernetes Basics](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/)

# Demo Blocking Traffic

This explains how to configure NGINX and Fail2Ban for traffic blocking and security management.

* Configure NGINX to serve `example.com` over HTTP/HTTPS
* Use `allow`/`deny` directives for static IP filtering
* Automate bans on repeated auth failures with Fail2Ban

We’ll use:

* An NGINX server on ports 80 & 443
* Two clients: **node01** (`192.231.128.12`) and **node02** (`192.231.128.3`)

## Prerequisites

* Ubuntu Server with NGINX installed
* Root or sudo access to `/etc/nginx` and `/etc/fail2ban`
* A self‐signed or valid SSL certificate for `example.com`

## 1. Configure NGINX for HTTPS and Basic Auth

Create or edit `/etc/nginx/sites-available/example-https`:

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

    root /var/www/html;
    index index.html index.htm;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
    add_header X-Frame-Options "SAMEORIGIN";
    add_header Content-Security-Policy "default-src 'self'";
    add_header Referrer-Policy origin;

    location / {
        try_files $uri $uri =404;
    }

    location /admin {
        auth_basic           "Restricted Access";
        auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
    }
}
```

Reload NGINX:

```bash  theme={null}
sudo nginx -t && sudo systemctl reload nginx
```

## 2. Test Connectivity from Clients

1. On the NGINX server, find its IP:
   ```bash  theme={null}
   ip a
   # e.g., eth0: inet 192.231.128.10/24
   ```
2. On **node01** and **node02**, add to `/etc/hosts`:
   ```text  theme={null}
   192.231.128.10 example.com
   ```
>  Make sure no conflicting DNS entries exist for `example.com`.

3. Run these tests on each node:

| Test                   | Command                                    | Expected Response           |
| ---------------------- | ------------------------------------------ | --------------------------- |
| HTTP → HTTPS redirect  | `curl http://example.com`                  | `301 Moved Permanently`     |
| HTTPS (self-signed)    | `curl https://example.com`                 | SSL certificate error       |
| Skip cert validation   | `curl -k https://example.com`              | HTML of index.html          |
| Access `/admin` header | `curl -k --head https://example.com/admin` | `HTTP/1.1 401 Unauthorized` |

## 3. Block a Single IP with `deny`

To block **node02** globally, edit the `/` location:

```nginx  theme={null}
location / {
    deny 192.231.128.3/32;  # node02
    try_files $uri $uri =404;
}
```

Reload and verify:

* **node01** (`192.231.128.12`):
  ```bash  theme={null}
  curl -k --head https://example.com
  # HTTP/1.1 200 OK
  ```
* **node02**:
  ```bash  theme={null}
  curl -k --head https://example.com
  # HTTP/1.1 403 Forbidden
  ```

## 4. Restrict `/admin` to a Single Host

Allow only **node01** to hit `/admin`:

```nginx  theme={null}
location /admin {
    allow 192.231.128.12/32;  # node01
    deny all;
    auth_basic           "Restricted Access";
    auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
}
```

Reload and test:

* **node02** → `403 Forbidden`
* **node01** → `401 Unauthorized` (prompt for credentials)

## 5. Allow a CIDR Range

To permit both nodes (in `192.231.128.0/24`) and block everyone else:

```nginx  theme={null}
location / {
    allow 192.231.128.0/24;
    deny all;
    try_files $uri $uri =404;
}

location /admin {
    allow 192.231.128.0/24;
    deny all;
    auth_basic           "Restricted Access";
    auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
}
```

Reload NGINX:

```bash  theme={null}
sudo nginx -t && sudo systemctl reload nginx
```

External clients will now see a web page with a "403 Forbidden" error message, indicating access is denied. It also mentions "nginx/1.18.0 (Ubuntu)" as the server.

## 6. Automate Banning with Fail2Ban

Replace manual IP lists with automatic bans on repeated auth failures.

1. Install Fail2Ban:
   ```bash  theme={null}
   sudo apt update
   sudo apt install fail2ban -y
   ```
>  If prompted to restart due to outdated libraries, choose **Ok**.

2. Copy the default jail configuration:
   ```bash  theme={null}
   sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
   ```

3. Enable the nginx-http-auth jail in `/etc/fail2ban/jail.local`:
   ```ini  theme={null}
   [nginx-http-auth]
   enabled  = true
   port     = http,https
   filter   = nginx-http-auth
   logpath  = %(nginx_error_log)s
   maxretry = 1
   bantime  = 600
   ```

4. Restart Fail2Ban:
   ```bash  theme={null}
   sudo systemctl restart fail2ban
   ```

5. Verify the jail status:
   ```bash  theme={null}
   sudo fail2ban-client status nginx-http-auth
   ```
   You should see the jail enabled with no banned IPs initially.

6. Trigger a ban by entering incorrect credentials in your browser shows a web browser displaying a login page for "example.com" with fields for a username and password, and buttons for "Cancel" and "Sign In.

Then check:

```bash  theme={null}
sudo fail2ban-client status nginx-http-auth
```

Your client IP will appear in the “Banned IP list.”


With NGINX’s static `allow`/`deny` and Fail2Ban’s dynamic banning, you have a robust defense against unwanted access and brute-force attempts.

## Links and References

* [NGINX Official Documentation](https://nginx.org/en/docs/)
* [Fail2Ban Wiki](https://www.fail2ban.org/wiki/index.php/Main_Page)
* [cURL Manual](https://curl.se/docs/manpage.html)
* [NGINX Access Module (`allow`/`deny`)](https://nginx.org/en/docs/http/ngx_http_access_module.html)
