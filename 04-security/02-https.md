# HTTPS

This explains HTTPS, its importance for security and SEO, and how to implement it on your website.

## Why HTTPS Matters

### 1. Security

When you access a website via HTTPS, all communication between your browser and the server is encrypted. On an unencrypted connection (HTTP), anyone on the same network such as a coffee shop Wi-Fi could intercept your passwords, credit-card numbers, or personal details.

With HTTP, data is sent in plain text. An attacker can easily read it.\
With HTTPS, intercepted data is encrypted and unreadable.

### 2. SEO Benefits

Search engines like Google prioritize secure sites in search rankings. Enabling HTTPS not only protects user data but also improves your site’s visibility and trustworthiness.

## SSL and TLS Protocols

SSL (Secure Sockets Layer) and TLS (Transport Layer Security) are cryptographic protocols that secure data in transit. Although we still colloquially call them “SSL certificates,” modern sites use TLS under the hood.

### SSL vs TLS: A Quick Comparison

| Protocol | Status      | Typical Use Case              |
| -------- | ----------- | ----------------------------- |
| SSL      | Deprecated  | Legacy or unsupported systems |
| TLS 1.2  | Widely Used | Production environments       |
| TLS 1.3  | Recommended | Best performance and security |

## How TLS Works: A Checkout Example

Imagine you’re on an e-commerce checkout page and submit your name, address, and credit-card details. TLS protects this process in four steps:

1. **Connection Initiation**\
   Your browser connects to the server over HTTPS.

2. **Certificate Exchange**\
   The server responds by sending its TLS certificate, which includes its public key and identity details.

3. **Certificate Authority (CA)**\
   A trusted CA—like Let’s Encrypt, DigiCert, or Comodo verifies the domain owner and signs the certificate.

4. **Domain Verification**\
   Your browser checks that the certificate matches the domain (e.g., `https://onlinestore.com`), ensuring you’re communicating with the real site.

## Asymmetric Encryption Explained

TLS employs asymmetric encryption (public-key cryptography), similar to SSH. A public key encrypts data, and only the corresponding private key can decrypt it.

1. The server publishes its public key (an “open padlock”) to your browser.
2. Your browser encrypts sensitive data like credit-card details using that public key.
3. The server applies its private key to decrypt the data, keeping it secure even if intercepted.

## Obtaining TLS Certificates

Choose a tool and provider based on your environment:

| Tool    | Provider      | Best For                         |
| ------- | ------------- | -------------------------------- |
| Certbot | Let’s Encrypt | Free, automated production certs |
| mkcert  | Self-signed   | Local development and testing    |

### 1. Let’s Encrypt + Certbot

Certbot is an ACME client that automates issuance and renewal of free TLS certificates from Let’s Encrypt.

```bash  theme={null}
sudo apt update
sudo apt install certbot

sudo certbot certonly \
  --standalone \
  --preferred-challenges http \
  -d example.com \
  -d www.example.com
```
>  Certbot creates a daily cron job for automatic renewal. Ensure ports 80 and 443 are available.

Certificates are saved at:

* `/etc/letsencrypt/live/example.com/fullchain.pem`
* `/etc/letsencrypt/live/example.com/privkey.pem`

### 2. mkcert (Self-Signed for Local Testing)

mkcert sets up a local CA and issues certificates trusted by your development machine. Not suitable for production.

```bash  theme={null}
sudo apt update
sudo apt install mkcert

# Change to your certificates directory
cd /etc/ssl/private

# Install mkcert’s local CA
mkcert --install

# Generate a wildcard certificate for *.example.com
mkcert *.example.com
```

>  Self-signed certificates from `mkcert` won’t be trusted by remote browsers or services. Use only for local development.

## Configuring Nginx for HTTPS

Once you have your certificate and private key, update your Nginx server block to listen on port 443:

```nginx  theme={null}
server {
    listen 443 ssl;
    server_name honda.cars.com;

    ssl_certificate     /etc/ssl/certs/honda.cars.com.pem;
    ssl_certificate_key /etc/ssl/certs/honda.cars.com-key.pem;

    root  /var/www/honda.cars.com/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Reload Nginx and verify your site at `https://honda.cars.com`.

## Links and References

* [HTTPS on Wikipedia](https://en.wikipedia.org/wiki/HTTPS)
* [TLS on Wikipedia](https://en.wikipedia.org/wiki/Transport_Layer_Security)
* [SSL on Wikipedia](https://en.wikipedia.org/wiki/Secure_Sockets_Layer)
* [Let’s Encrypt](https://letsencrypt.org)
* [Certbot](https://certbot.eff.org)
* [mkcert](https://mkcert.dev)
* [Nginx](https://nginx.org)

# Demo HTTPS

In this you’ll learn how to:

1. Redirect all HTTP traffic to HTTPS using Nginx.
2. Generate a local SSL certificate for testing.
3. Configure Nginx as an HTTPS reverse proxy forwarding encrypted requests to Apache backends on port 443.

## 1. Simple HTTP → HTTPS Redirect

### 1.1 Open Port 443 in the Firewall

Verify that TCP port 443 is allowed through UFW:

```bash  theme={null}
# Check current UFW status
sudo ufw status

# If 443/tcp is not listed, enable it:
sudo ufw allow 443/tcp
```

### 1.2 Switch from HTTP-Only to HTTPS

Remove the old HTTP site and enable the HTTPS configuration:

```bash  theme={null}
# Disable HTTP site
sudo rm /etc/nginx/sites-enabled/example-http

# List available site configs
ls -l /etc/nginx/sites-available
# Edit the HTTPS site template
sudo vim /etc/nginx/sites-available/example-https
```

Add the following to `example-https`:

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

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable the site and test Nginx:

```bash  theme={null}
sudo ln -s /etc/nginx/sites-available/example-https /etc/nginx/sites-enabled/
sudo nginx -t
```

>  Nginx will report missing certificate files until you generate them in the next step.

### 1.3 Generate a Test SSL Certificate with mkcert

We’ll use [mkcert](https://github.com/FiloSottile/mkcert) to create a local CA and certificate:

```bash  theme={null}
cd /etc/ssl/certs
sudo mkcert example.com
```

Sample output:

```text  theme={null}
Created a new local CA 🏴
Note: the local CA is not installed in the system trust store. Run "mkcert -install" to trust it automatically.
Created a certificate valid for:
 - "example.com"
The certificate is at "./example.com.pem"
The key is at "./example.com-key.pem"
```

Reload Nginx with the new certificates:

```bash  theme={null}
sudo nginx -t
sudo nginx -s reload
```

Now visit:\
[https://example.com](https://example.com)

## 2. HTTPS Reverse Proxy to Apache Backends

### 2.1 Configure Nginx Upstream and proxy\_pass

Edit `/etc/nginx/sites-available/example-https` to add an `upstream` block:

```nginx  theme={null}
# Upstream Apache backends on port 443
upstream example {
    server 192.230.210.3:443;
    server 192.230.210.6:443;
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}

# HTTPS listener forwarding to upstream
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/ssl/certs/example.com.pem;
    ssl_certificate_key /etc/ssl/certs/example.com-key.pem;

    root /var/www/html;
    index index.html index.htm;

    location / {
        proxy_pass https://example;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Test and reload:

```bash  theme={null}
sudo nginx -t
sudo nginx -s reload
```

### 2.2 Configure Apache Backends

On each Apache node:

1. Allow HTTPS in the firewall:\
   `sudo ufw allow 443/tcp`
2. Create `/etc/apache2/sites-available/example.conf`:

```apache  theme={null}
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://%{HTTP_HOST}%{REQUEST_URI}
</VirtualHost>

<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/html

    SSLEngine on
    SSLCertificateFile    /etc/ssl/certs/example.com.pem
    SSLCertificateKeyFile /etc/ssl/certs/example.com-key.pem

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

3. Enable the site and reload Apache:

```bash  theme={null}
sudo a2ensite example.conf
sudo systemctl reload apache2
```

Now, [https://example.com](https://example.com) → Nginx → one of your Apache backends over HTTPS.

***

## Configuration Reference

| File                                      | Purpose                                         |
| ----------------------------------------- | ----------------------------------------------- |
| /etc/nginx/sites-available/example-https  | Nginx HTTP→HTTPS redirect & HTTPS reverse proxy |
| /etc/apache2/sites-available/example.conf | Apache vhosts for HTTP and HTTPS                |


## Links and References

* [mkcert GitHub Repository](https://github.com/FiloSottile/mkcert)
* [Nginx SSL Module](https://nginx.org/en/docs/http/ngx_http_ssl_module.html)
* [Apache SSL/TLS Encryption](https://httpd.apache.org/docs/2.4/ssl/)

