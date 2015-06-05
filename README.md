# Caddy image for Docker

Without configuration, this Docker image encapsulates a
[*Caddy*](http://caddyserver.com) HTTP server for static files.
A `/bin/caddy` process listens on the container's `EXPOSE`d TCP
port #2015 and attempts to fulfill requests with files beneath
the container's `/var/www/html/`.

Content should be added by binding over that path with a host volume,
or by `COPY`ing/`ADD`ing files there when `docker build`ing a custom
image based on this one. Adding a `Caddyfile` through the same
mechanisms allows configuration.

## Container file system:
* `/bin/caddy` # HTTP server executable
* `/etc/passwd` # Names user/UID under which caddy runs
* `/var/www/html/` # Server's working directory and HTTP name space root

## Adding Content

There are at least two ways to provide Caddy with content and configuration.

* Bind a host file system path over the container's HTTP name space root:
```
% ls /home/user/site
  index.html
  img/
  [...]
% docker run -p 8080:2015 -v /home/user/site:/var/www/html:ro -d joshix/caddy
```

OR,

* Build the files into an image based on this one:
```
% cd /home/user/site.build
% ls
  Dockerfile
  index.html
  img/logo.png
  [...]
% cat Dockerfile
  FROM joshix/caddy
  COPY . /var/www/html
% docker build -t "com.mysite-caddy" .
% docker run -p 8080:2015 -d com.mysite-caddy
```

## Configuration
To configure Caddy, add `Caddyfile` to the HTTP root in the same fashion:
```
% ls /home/user/site
  Caddyfile
  index.md
  img/
  [...]
% cat Caddyfile
  0.0.0.0:2015
  ext .html .htm .md
  markdown /
  gzip
  [...]
% docker run -p 8080:2015 -v /home/user/site:/var/www/html:ro -d joshix/caddy
```

### TLS
To serve HTTP and HTTP2 over encrypted connections, provide certificate and key
files and a Caddyfile naming them, something like:
```
% ls /home/user/site
  html/
  tls/
% ls /home/user/site/html
  Caddyfile
  index.html
  img/
  [...]
% ls /home/user/site/tls
  site.crt
  site.key
% cat /home/user/site/html/Caddyfile
  0.0.0.0:2015 {
    redir https://site.com # Redirect any HTTP req to HTTPS
  }
  0.0.0.0:2378 {
    tls ../tls/site.crt ../tls/site.key
  }
% docker run -p 80:2015 -p 443:2378 -v /home/user/site:/var/www:ro -d joshix/caddy
```
