# Demo First Website with Nginx

In this you’ll learn how to deploy a simple **Hello World** site using Nginx. By the end, you’ll have a custom virtual host serving your own HTML page.

## 1. Verify and Start Nginx

First, confirm that Nginx is installed and running on your server.

```bash  theme={null}
sudo systemctl status nginx
sudo systemctl start nginx
curl http://localhost
```

You should see the default “Welcome to nginx!” page:

```html  theme={null}
<!DOCTYPE html>
<html>
<head>
  <title>Welcome to nginx!</title>
  <style>
    body { width: 35em; margin: 0 auto; font-family: Tahoma, Verdana, Arial, sans-serif; }
  </style>
</head>
<body>
  <h1>Welcome to nginx!</h1>
  <p>If you see this page, the nginx web server is successfully installed and working. Further configuration is required.</p>
  <!-- ... -->
</body>
</html>
```

## 2. Create a New Site Configuration

1. Switch to the Nginx configuration directory and assume root privileges:

   ```bash  theme={null}
   cd /etc/nginx
   sudo su
   ```

2. Copy the default server block to a new file named `helloworld`:

   ```bash  theme={null}
   cd sites-available
   cp default helloworld
   ```

3. Open `sites-available/helloworld` in your preferred editor and replace its contents with:

   ```nginx  theme={null}
   server {
       listen 80;
       server_name helloworld.com;

       root /var/www/helloworld;
       index index.html index.htm;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

   | Directive    | Purpose                                            |
   | ------------ | -------------------------------------------------- |
   | listen 80;   | Accept HTTP requests on port 80                    |
   | server\_name | Match the `Host` header to `helloworld.com`        |
   | root         | Define the document root for this virtual host     |
   | index        | Specify default files to serve                     |
   | try\_files   | Serve requested files or return a 404 if not found |


Save and exit the editor.

## 3. Create the Site Content

Create the document root and add a simple HTML page:

```bash  theme={null}
mkdir -p /var/www/helloworld
cat > /var/www/helloworld/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Hello World</title>
</head>
<body>
  <h1>Hello World!</h1>
  <p>Your Nginx web server is now serving custom content.</p>
</body>
</html>
EOF
```

## 4. Enable the Site and Reload Nginx

1. Create a symbolic link in `sites-enabled`:

   ```bash  theme={null}
   ln -s /etc/nginx/sites-available/helloworld /etc/nginx/sites-enabled/
   ```

2. Test the syntax and reload Nginx:

   ```bash  theme={null}
   nginx -t
   nginx -s reload
   ```

## 5. Test Your Hello World Site

* To view the default site again:

  ```bash  theme={null}
  curl http://localhost
  ```

* To test your custom site without DNS:

  ```bash  theme={null}
  curl --header "Host: helloworld.com" http://localhost
  ```

  Expected response:

  ```html  theme={null}
  <h1>Hello World!</h1>
  ```

* If you send an unknown host header, Nginx will serve the first server block alphabetically (the default site):

  ```bash  theme={null}
  curl --header "Host: unknown.com" http://localhost
  ```

  You can add an entry to `/etc/hosts` for `helloworld.com` to test in a browser without DNS:

  ```text  theme={null}
  127.0.0.1   helloworld.com
  ```


***

## Links and References

* [Nginx Official Docs](https://nginx.org/en/docs/)
* [Understanding Nginx Server Blocks](https://www.nginx.com/resources/wiki/start/topics/examples/server_blocks/)
* [Using curl to Test Web Servers](https://curl.se/docs/manpage.html)
