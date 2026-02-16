# Security Introduction

This focuses on securing web applications end-to-end through HTTPS, TLS, secure headers, authentication, and access control.

1. Enable HTTPS using TLS certificates
2. Understand TLS protocols and data encryption
3. Implement secure HTTP response headers
4. Protect resources with Nginx Basic Authentication
5. Control access and integrate fail2ban for automated bans

## 1. Enabling HTTPS with Self-Signed Certificates

To get HTTPS running locally, we’ll use [mkcert](https://github.com/FiloSottile/mkcert) for generating a trusted, self-signed certificate without needing a public domain.

mkcert is perfect for development and testing environments because it automatically adds the generated CA to your local trust store. For production, switch to automated certificates from [Let’s Encrypt](https://letsencrypt.org/) using [Certbot](https://certbot.eff.org/).

```bash  theme={null}
# Install mkcert (macOS example)
brew install mkcert nss

# Create a local CA
mkcert -install

# Generate a cert for localhost
mkcert localhost 127.0.0.1 ::1

# You’ll get files: localhost.pem (cert) and localhost-key.pem (key)
```

Next, configure Nginx to use these files:

```nginx  theme={null}
server {
    listen       443 ssl;
    server_name  localhost;

    ssl_certificate     /path/to/localhost.pem;
    ssl_certificate_key /path/to/localhost-key.pem;

    location / {
        root   html;
        index  index.html index.htm;
    }
}
```

## 2. How TLS Protects Data in Transit

TLS ensures data integrity and confidentiality with:

| Feature            | Description                                   |
| ------------------ | --------------------------------------------- |
| Encryption         | Encrypts payload to prevent eavesdropping     |
| Authentication     | Verifies server identity via certificates     |
| Integrity Checking | Detects tampering with message authentication |

***

## 3. Secure HTTP Response Headers

Adding HTTP headers can mitigate common web attacks. Configure these in your Nginx `server` block:

| Header                  | Purpose                         | Example Value                |
| ----------------------- | ------------------------------- | ---------------------------- |
| Content-Security-Policy | Prevents XSS and data injection | `default-src 'self';`        |
| X-Frame-Options         | Stops clickjacking              | `DENY`                       |
| X-Content-Type-Options  | Disallows MIME-type sniffing    | `nosniff`                    |
| Referrer-Policy         | Controls referral information   | `no-referrer-when-downgrade` |

```nginx  theme={null}
add_header Content-Security-Policy "default-src 'self';" always;
add_header X-Frame-Options "DENY" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
```

***

## 4. Protecting Endpoints with Basic Authentication

Nginx’s `auth_basic` module offers a simple user/password prompt for selected locations.

```bash  theme={null}
# Install apache2-utils for htpasswd
sudo apt-get install apache2-utils

# Create credentials file
htpasswd -c /etc/nginx/.htpasswd user1
```

```nginx  theme={null}
server {
    listen 80;
    server_name example.com;

    location /secure {
        auth_basic           "Restricted Area";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

> Basic authentication uses a flat file for credentials and should only be used for low-risk or internal applications. Consider OAuth or LDAP for stronger authentication mechanisms.

## 5. Allowing, Denying, and Automating Bans with fail2ban

Control access with `allow` and `deny` directives, then integrate [fail2ban](https://www.fail2ban.org/) to automatically block repeated offenders:

```nginx  theme={null}
server {
    listen 80;
    server_name example.com;

    # Allow only internal network
    allow 10.0.0.0/24;
    deny all;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Finally, configure fail2ban’s Nginx filter and jail:

```ini  theme={null}
# /etc/fail2ban/jail.local
[nginx-http-auth]
enabled = true
filter  = nginx-http-auth
port    = http,https
logpath = /var/log/nginx/*error*.log
maxretry = 3
```

## Links and References

* [mkcert GitHub Repository](https://github.com/FiloSottile/mkcert)
* [Let’s Encrypt](https://letsencrypt.org/)
* [Certbot — EFF](https://certbot.eff.org/)
* [Nginx Documentation](https://nginx.org/en/docs/)
* [fail2ban Official Site](https://www.fail2ban.org/)
