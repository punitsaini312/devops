# Nginx Notes --- Revision Handbook

## 1. Nginx Installation

### Ubuntu / Debian

``` bash
sudo apt update
sudo apt install nginx
```

Check installation:

``` bash
nginx -v
```

Check where Nginx is installed:

``` bash
which nginx
```

------------------------------------------------------------------------

## 2. Nginx Service Management

Nginx is normally managed using `systemctl`.

### Start Nginx

``` bash
sudo systemctl start nginx
```

### Stop Nginx

``` bash
sudo systemctl stop nginx
```

### Restart Nginx

``` bash
sudo systemctl restart nginx
```

This stops and starts Nginx again.

### Reload Nginx

``` bash
sudo systemctl reload nginx
```

Reload applies configuration changes without completely stopping Nginx.

**Important:** After changing an Nginx configuration, prefer:

``` bash
sudo nginx -t
sudo systemctl reload nginx
```

### Check status

``` bash
sudo systemctl status nginx
```

### Enable Nginx at boot

``` bash
sudo systemctl enable nginx
```

### Disable Nginx at boot

``` bash
sudo systemctl disable nginx
```

### Check configuration syntax

``` bash
sudo nginx -t
```

Typical successful output:

``` text
syntax is ok
test is successful
```

------------------------------------------------------------------------

# 3. Basic Structure of `nginx.conf`

Main configuration file:

``` text
/etc/nginx/nginx.conf
```

A simplified structure looks like:

``` nginx
events {
    # Connection-related settings
}

http {

    # HTTP-level settings

    server {

        # Website/server configuration

        listen 80;
        server_name example.com;

        location / {
            # Configuration for /
        }

        location /about {
            # Configuration for /about
        }
    }

    server {
        # Another website
    }
}
```

## Important hierarchy

Remember:

``` text
nginx
│
├── events
│
└── http
    │
    ├── server
    │   ├── location /
    │   └── location /about
    │
    └── server
        └── location /
```

### `events`

Contains settings related to how Nginx handles connections.

Example:

``` nginx
events {
    worker_connections 1024;
}
```

### `http`

Contains HTTP/web-server configuration.

For example:

``` nginx
http {
    include /etc/nginx/mime.types;

    server {
        ...
    }
}
```

### `server`

A `server` block represents a virtual server / website.

Example:

``` nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html/demo;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Multiple websites can have multiple `server` blocks.

### `location`

Controls how Nginx handles a particular URL path.

Example:

``` nginx
location / {
    ...
}

location /about {
    ...
}
```

Then:

``` text
http://example.com/
        ↓
location /

http://example.com/about
        ↓
location /about
```

------------------------------------------------------------------------

# 4. Important Nginx Files and Directories

## Main configuration

``` text
/etc/nginx/nginx.conf
```

This is the main Nginx configuration file.

## Sites available

``` text
/etc/nginx/sites-available/
```

Contains server configurations that are available to use.

Example:

``` text
/etc/nginx/sites-available/default
```

## Sites enabled

``` text
/etc/nginx/sites-enabled/
```

Contains configurations that Nginx actually loads.

Usually, files here are symbolic links to files in `sites-available`.

Check:

``` bash
ls -l /etc/nginx/sites-enabled/
```

Typical relationship:

``` text
sites-available/default
        ↑
        │ symbolic link
        │
sites-enabled/default
```

## Default web root

On Ubuntu/Debian, a common default web root is:

``` text
/var/www/html/
```

Example:

``` text
/var/www/html/index.html
```

## Nginx logs

Access log:

``` text
/var/log/nginx/access.log
```

Error log:

``` text
/var/log/nginx/error.log
```

Useful commands:

``` bash
sudo tail -f /var/log/nginx/access.log
```

``` bash
sudo tail -f /var/log/nginx/error.log
```

## Other useful directory

``` text
/etc/nginx/conf.d/
```

Can contain additional `.conf` configuration files.

------------------------------------------------------------------------

# 5. My Practical Nginx Exercise

## Step 1 --- Create a demo website

Create a directory:

``` bash
sudo mkdir -p /var/www/html/demo
```

Create an HTML file:

``` bash
sudo nano /var/www/html/demo/index.html
```

Example:

``` html
<h1>This is my Demo Website</h1>
```

------------------------------------------------------------------------

## Step 2 --- Remove the default `index.html`

The default Nginx page is normally:

``` text
/var/www/html/index.html
```

Remove it:

``` bash
sudo rm /var/www/html/index.html
```

Now the default website content is removed.

------------------------------------------------------------------------

## Step 3 --- Change the Nginx default configuration

Edit:

``` bash
sudo nano /etc/nginx/sites-available/default
```

On Ubuntu/Debian, the enabled configuration is usually linked as:

``` text
/etc/nginx/sites-enabled/default
```

A basic configuration:

``` nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/html/demo;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

The important change is:

``` nginx
root /var/www/html/demo;
```

Now when the browser requests:

``` text
http://SERVER_IP/
```

Nginx looks inside:

``` text
/var/www/html/demo/
```

and serves:

``` text
/var/www/html/demo/index.html
```

------------------------------------------------------------------------

## Step 4 --- Test and reload

Always test the configuration first:

``` bash
sudo nginx -t
```

Then:

``` bash
sudo systemctl reload nginx
```

------------------------------------------------------------------------

# 6. Practising URL Paths with `/about`

Now create an `about` directory:

``` bash
sudo mkdir /var/www/html/demo/about
```

Create:

``` bash
sudo nano /var/www/html/demo/about/index.html
```

Example:

``` html
<h1>About Page</h1>
```

Now the directory structure is:

``` text
/var/www/html/demo/
│
├── index.html
│
└── about/
    └── index.html
```

Because the Nginx root is:

``` nginx
root /var/www/html/demo;
```

these URLs map to:

``` text
http://SERVER_IP/
        ↓
/var/www/html/demo/index.html
```

and:

``` text
http://SERVER_IP/about/
        ↓
/var/www/html/demo/about/index.html
```

### Important concept

Nginx combines:

``` text
root + URL path
```

For:

``` text
root = /var/www/html/demo
URL  = /about/
```

Nginx looks approximately at:

``` text
/var/www/html/demo/about/
```

------------------------------------------------------------------------

# 7. Hosting a Second Website

Now create another website directory:

``` bash
sudo mkdir -p /var/www/html/demo/mysecondwebsite
```

Create its HTML file:

``` bash
sudo nano /var/www/html/demo/mysecondwebsite/index.html
```

Example:

``` html
<h1>My Second Website</h1>
```

Directory structure:

``` text
/var/www/html/demo/
│
├── index.html
│
├── about/
│   └── index.html
│
└── mysecondwebsite/
    └── index.html
```

There are two different concepts here:

### Case 1 --- Second website under a URL path

If you simply use the same `root`:

``` nginx
root /var/www/html/demo;

location / {
    try_files $uri $uri/ =404;
}
```

then:

``` text
http://SERVER_IP/mysecondwebsite/
```

serves:

``` text
/var/www/html/demo/mysecondwebsite/index.html
```

This is a **different URL path**, but technically it is still inside the
same `server` block.

------------------------------------------------------------------------

# 8. Adding a New `server` Block

To host a genuinely separate website, create another `server` block.

Example:

``` nginx
server {
    listen 80;
    server_name mysecondwebsite.com;

    root /var/www/html/demo/mysecondwebsite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Now you can have:

``` nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html/demo;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen 80;
    server_name mysecondwebsite.com;

    root /var/www/html/demo/mysecondwebsite;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### How does Nginx decide which server block to use?

The browser sends the hostname:

``` text
Host: example.com
```

or:

``` text
Host: mysecondwebsite.com
```

Nginx uses the `server_name` to select the appropriate `server` block.

Conceptually:

``` text
example.com
      ↓
server_name example.com
      ↓
root /var/www/html/demo


mysecondwebsite.com
      ↓
server_name mysecondwebsite.com
      ↓
root /var/www/html/demo/mysecondwebsite
```

------------------------------------------------------------------------

# 9. Reverse Proxy

A reverse proxy is different from serving static HTML files.

Instead of:

``` text
Browser → Nginx → HTML file
```

Nginx can work like:

``` text
Browser
   │
   ▼
 Nginx
   │
   │ proxy_pass
   ▼
Application
```

For example, suppose your application is running on:

``` text
127.0.0.1:3000
```

You want users to access:

``` text
http://example.com/
```

through Nginx.

Configuration:

``` nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Now:

``` text
Browser
   │
   │ http://example.com
   ▼
Nginx :80
   │
   │ proxy_pass
   ▼
Application :3000
```

## What changed?

For static website hosting:

``` nginx
root /var/www/html/demo;
```

Nginx reads files from the filesystem.

For reverse proxy:

``` nginx
proxy_pass http://127.0.0.1:3000;
```

Nginx forwards the request to another application/server.

### Example with `/api`

Suppose:

``` text
Frontend → Nginx
             │
             ├── /       → frontend/static files
             │
             └── /api/   → backend :8000
```

Configuration:

``` nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html/demo;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Request:

``` text
GET /
```

→ Nginx serves the frontend.

Request:

``` text
GET /api/users
```

→ Nginx forwards it to the backend.

------------------------------------------------------------------------

# 10. Very Important Nginx Concepts

## Process vs Request

A request does **not** normally create a new Nginx process.

Nginx has long-running processes:

``` text
Nginx
│
├── Master process
│
├── Worker 1
├── Worker 2
├── Worker 3
└── Worker 4
```

Workers handle many requests.

Therefore:

``` text
1000 HTTP requests
        ≠
1000 Nginx processes
```

Every request can consume CPU and memory, but the existing Nginx workers
handle the requests.

------------------------------------------------------------------------

# 11. Useful Commands for Practice

Check Nginx version:

``` bash
nginx -v
```

Test configuration:

``` bash
sudo nginx -t
```

See running processes:

``` bash
ps aux | grep nginx
```

Check listening ports:

``` bash
sudo ss -tulpn | grep nginx
```

Check service:

``` bash
sudo systemctl status nginx
```

Reload configuration:

``` bash
sudo systemctl reload nginx
```

View access logs:

``` bash
sudo tail -f /var/log/nginx/access.log
```

View error logs:

``` bash
sudo tail -f /var/log/nginx/error.log
```

------------------------------------------------------------------------

# 12. Interview Questions

## Beginner

### 1. What is Nginx?

Nginx is a web server and reverse proxy server. It can serve static
content, terminate TLS, load-balance traffic, and forward requests to
backend applications.

### 2. What is the difference between a web server and a reverse proxy?

A web server can directly serve content such as HTML, CSS, JavaScript,
and images.

A reverse proxy receives client requests and forwards them to backend
servers/applications.

### 3. What is `nginx.conf`?

It is the main Nginx configuration file.

Usually:

``` text
/etc/nginx/nginx.conf
```

### 4. What is a `server` block?

A `server` block defines configuration for a virtual server/website.

### 5. What is a `location` block?

A `location` block defines how Nginx handles requests matching a
particular URI path.

### 6. What is the purpose of `root`?

`root` tells Nginx where static files are located.

Example:

``` nginx
root /var/www/html/demo;
```

### 7. What is `server_name`?

It specifies the hostname/domain for which a server block should handle
requests.

Example:

``` nginx
server_name example.com;
```

### 8. What is `proxy_pass`?

`proxy_pass` tells Nginx to forward a request to another
server/application.

Example:

``` nginx
proxy_pass http://127.0.0.1:3000;
```

------------------------------------------------------------------------

## Intermediate

### 9. What is the difference between `reload` and `restart`?

`reload` loads the new configuration while keeping the existing Nginx
service running.

`restart` stops and starts the service again.

For normal configuration changes:

``` bash
sudo nginx -t
sudo systemctl reload nginx
```

is generally preferred.

### 10. What are `sites-available` and `sites-enabled`?

`sites-available` contains available site configurations.

`sites-enabled` contains configurations that are enabled and loaded by
Nginx.

On Debian/Ubuntu, `sites-enabled` commonly contains symbolic links to
`sites-available`.

### 11. How do you check whether an Nginx configuration is valid?

``` bash
sudo nginx -t
```

### 12. Where are Nginx logs?

Usually:

``` text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

### 13. How can Nginx host multiple websites?

Using multiple `server` blocks with different `server_name` values.

Example:

``` nginx
server {
    listen 80;
    server_name site1.com;
    ...
}

server {
    listen 80;
    server_name site2.com;
    ...
}
```

### 14. What happens when a user requests `/about`?

Nginx first determines the appropriate `server` block and then selects
the matching `location` block.

If serving static files, the URL is mapped against the configured
`root`.

Example:

``` text
root: /var/www/html/demo
URL:  /about/
```

Result:

``` text
/var/www/html/demo/about/
```

### 15. Why do we run `nginx -t` before reload?

To verify that the configuration syntax is valid before applying it.
This helps avoid reloading a broken configuration.

------------------------------------------------------------------------

# 13. Quick Revision

``` text
Install
   ↓
apt install nginx
   ↓
nginx.conf
   ↓
events + http
   ↓
server
   ↓
location
```

### Static website

``` text
Browser
   ↓
Nginx
   ↓
root
   ↓
HTML/CSS/JS files
```

### Reverse proxy

``` text
Browser
   ↓
Nginx
   ↓
proxy_pass
   ↓
Backend application
```

### Important commands

``` bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl status nginx

sudo nginx -t
```

### Important paths

``` text
/etc/nginx/nginx.conf
/etc/nginx/sites-available/
/etc/nginx/sites-enabled/
/etc/nginx/conf.d/
/var/www/html/
/var/log/nginx/access.log
/var/log/nginx/error.log
```

### Most important mental model

``` text
Client
  ↓
Nginx
  ↓
server block
  ↓
location block
  ↓
 ┌──────────────────┐
 │                  │
root             proxy_pass
 │                  │
 ↓                  ↓
Static files      Backend
```

**Remember:**

> `server` decides **which website/server configuration** handles the
> request.

> `location` decides **how a particular URL/path is handled**.

> `root` tells Nginx **where static files are located**.

> `proxy_pass` tells Nginx **where to forward the request**.
