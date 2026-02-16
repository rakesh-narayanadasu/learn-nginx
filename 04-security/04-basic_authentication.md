# Basic Authentication

Nginx provides a method to secure website sections using HTTP basic authentication, allowing access only to users with valid credentials.

Nginx offers a straightforward way to secure parts of your website such as admin panels, staging previews, or premium sections using HTTP basic authentication. Only users with valid credentials can access protected resources, while the rest of your site remains publicly available.

For example, anyone can browse the public pages of `example.com`, but visiting `example.com/admin` will trigger a browser login prompt:

> HTTP basic authentication sends credentials encoded in Base64, not encrypted. Use it only within trusted networks (VPN or internal LAN). For public-facing apps, consider OAuth, JWT, or framework-native authentication for better security and user experience.

## 1. Generate the Password File (`.htpasswd`)

You can create the `.htpasswd` file with either Apache’s utility or OpenSSL. Store this file in a secure directory (e.g., `/etc/nginx/conf.d/`).

| Method            | Requirement                      | Usage Example           |
| ----------------- | -------------------------------- | ----------------------- |
| Apache `htpasswd` | `apache2-utils` or `httpd-tools` | Add users with a prompt |
| OpenSSL           | Built-in on most systems         | Manually append hashes  |

### 1.1 Using Apache’s `htpasswd`

```bash  theme={null}
# Install the tool if needed
sudo apt-get install apache2-utils   # Debian/Ubuntu
sudo yum install httpd-tools         # RHEL/CentOS

# Create a new file and add the first user
sudo htpasswd -c /etc/nginx/conf.d/.htpasswd admin
# Add additional users (omit -c to avoid overwriting)
sudo htpasswd /etc/nginx/conf.d/.htpasswd jsmith
# Enter and confirm password
```

### 1.2 Using OpenSSL

```bash  theme={null}
# For each user, append "username:" then an APR1-hashed password
sudo sh -c "echo -n 'admin:' >> /etc/nginx/conf.d/.htpasswd"
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/conf.d/.htpasswd"
sudo sh -c "echo -n 'jsmith:' >> /etc/nginx/conf.d/.htpasswd"
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/conf.d/.htpasswd"
# Enter and verify password
```
>  Once hashed, plain-text passwords cannot be recovered. Store them securely using a password manager like LastPass or a secrets vault (e.g., HashiCorp Vault).

To confirm your `.htpasswd` entries:

```bash  theme={null}
cat /etc/nginx/conf.d/.htpasswd
# admin:$apr1$egX1fPMK$EXwGqVFsOSBFsQNJMc2iB0
# jsmith:$apr1$L5aCfsuk$XPsXgl1JMTQpd0ihTVyus.
```

## 2. Configure Nginx to Require Authentication

Edit your server block (commonly in `/etc/nginx/sites-available/` or `/etc/nginx/conf.d/`) to protect a specific location, such as `/admin`:

```nginx  theme={null}
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/example.com/html;
    index index.html;

    location /admin {
        auth_basic           "Restricted Content";
        auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
    }
}
```

* **auth\_basic**: Text shown in the browser’s login dialog.
* **auth\_basic\_user\_file**: Path to your `.htpasswd` file.

Save the file, test your configuration, and reload Nginx:

```bash  theme={null}
sudo nginx -t
sudo systemctl reload nginx
```

When you navigate to `http://example.com/admin`, a login popup appears:

## 3. Next Steps & References

A live demo will follow, showcasing each step in action. In the meantime, explore these resources to deepen your understanding:

| Resource                      | Link                                                                                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Nginx Official Documentation  | [https://nginx.org/en/docs/http/ngx\_http\_auth\_basic\_module.html](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html) |
| Apache `htpasswd` Guide       | [https://httpd.apache.org/docs/2.4/programs/htpasswd.html](https://httpd.apache.org/docs/2.4/programs/htpasswd.html)                 |
| OpenSSL Password Hash Options | [https://www.openssl.org/docs/man1.1.1/man1/openssl-passwd.html](https://www.openssl.org/docs/man1.1.1/man1/openssl-passwd.html)     |
| OAuth 2.0 Overview            | [https://oauth.net/2/](https://oauth.net/2/)                                                                                         |

For further reading:

* [Securing Nginx with HTTPS](https://nginx.org/en/docs/http/configuring_https_servers.html)
* [HashiCorp Vault](https://www.vaultproject.io/)
* [LastPass Password Manager](https://www.lastpass.com/)


> ## Documentation Index
> Fetch the complete documentation index at: https://notes.kodekloud.com/llms.txt
> Use this file to discover all available pages before exploring further.

# Demo Authentication

In this we’ll learn how to secure the `/admin` endpoint of your `example.com` site using HTTP Basic Authentication. The main site remains publicly accessible while only `/admin` requires credentials. We’ll cover:

* Verifying your site setup
* Updating the Nginx configuration
* Generating an `.htpasswd` file
* Testing and reloading Nginx
* (Optional) Protecting the entire site
* Exploring advanced auth solutions

## 1. Verify Your Site

Make sure Nginx is serving your site over HTTPS on port 443:

```bash  theme={null}
# In your browser, visit:
https://example.com
https://example.com/admin
```

Both URLs should load without authentication for now.

## 2. Update Nginx Configuration

Open your SSL-enabled server block (e.g., `/etc/nginx/sites-available/example-https`):

```bash  theme={null}
sudo vim /etc/nginx/sites-available/example-https
```

Locate and update the `server` block to include a dedicated `/admin` location:

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
    root                /var/www/html;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
    add_header X-Frame-Options "SAMEORIGIN";
    add_header Content-Security-Policy "default-src 'self'";
    add_header Referrer-Policy origin;

    index index.html index.htm index.nginx-debian.html;

    # Public content
    location / {
        try_files $uri $uri/ =404;
    }

    # Protect /admin with HTTP Basic Auth
    location /admin {
        auth_basic           "Restricted Access";
        auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
    }
}
```

Save and exit.

>  HTTP Basic Authentication transmits credentials as Base64-encoded text. Always use HTTPS to prevent interception.

## 3. Create the `.htpasswd` File

Generate a username and encrypted password for Basic Auth:

1. Change to the directory for password files:
   ```bash  theme={null}
   cd /etc/nginx/conf.d
   ```

2. Initialize the file and add the `admin` user:
   ```bash  theme={null}
   sudo sh -c "echo -n 'admin:' > .htpasswd"
   ```

3. Append an encrypted password (you’ll be prompted):
   ```bash  theme={null}
   sudo sh -c "openssl passwd -apr1 >> .htpasswd"
   ```

4. Verify the contents:
   ```bash  theme={null}
   ls -la .htpasswd
   cat .htpasswd
   # Example output:
   # admin:$apr1$MASb7ZA.$8LOCauVuqg5nH2AIk72/
   ```

Ensure that `.htpasswd` is readable by the Nginx user but not world-readable:

```bash  theme={null}
sudo chmod 640 /etc/nginx/conf.d/.htpasswd
```

## 4. Test and Reload Nginx

Validate and apply your configuration:

```bash  theme={null}
sudo nginx -t
sudo nginx -s reload
```

Now, refresh in your browser:

* **Public pages** load as before.
* **`/admin`** prompts for credentials.

Enter `admin` and your password to access the admin area.

## 5. (Optional) Password-Protect the Entire Site

If you want every page to require authentication, move the `auth_basic` directives into the `/` block:

```nginx  theme={null}
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate     /etc/ssl/certs/example.com.pem;
    ssl_certificate_key /etc/ssl/certs/example.com-key.pem;
    root                /var/www/html;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
    add_header X-Frame-Options "SAMEORIGIN";
    add_header Content-Security-Policy "default-src 'self'";
    add_header Referrer-Policy origin;

    index index.html index.htm index.nginx-debian.html;

    location / {
        auth_basic           "Restricted Access";
        auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
        try_files $uri $uri/ =404;
    }
}
```

Reload Nginx:

```bash  theme={null}
sudo nginx -t
sudo nginx -s reload
```

Every request to `https://example.com` will now prompt for the Basic Auth credentials. Test it in an incognito/private window.

## 6. Beyond Basic Auth

For scalable, production-grade authentication, consider integrating with identity providers and single sign-on solutions:

| Resource                   | Use Case                         | Link                                                                                                                                       |
| -------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| OAuth 2.0 / OpenID Connect | Token-based authentication flows | [https://oauth.net/](https://oauth.net/) / [https://openid.net/connect/](https://openid.net/connect/)                                      |
| Okta SSO (Nginx Plus)      | Enterprise SSO integration       | [https://www.okta.com/](https://www.okta.com/)                                                                                             |
| Active Directory           | Windows AD integration           | [https://docs.microsoft.com/windows-server/identity/active-directory](https://docs.microsoft.com/windows-server/identity/active-directory) |

### Installing the JavaScript module

Required by some advanced Nginx+ auth integrations:

| OS            | Command                                 |
| ------------- | --------------------------------------- |
| Debian/Ubuntu | `sudo apt install nginx-plus-module-js` |
| RHEL/CentOS   | `sudo yum install nginx-plus-module-js` |

And load it in your main `nginx.conf`:

```nginx  theme={null}
load_module modules/ngx_http_js_module.so;
```

By following these steps, you’ve added HTTP Basic Authentication to your Nginx server and explored more robust alternatives for production. For further reading, see the official [Nginx documentation](https://nginx.org/en/docs/) 
