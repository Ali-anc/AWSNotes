

#  How do I capture client IP addresses in the web server logs behind an ELB?'


> Application Load Balancers and Classic Load Balancers with HTTP/HTTPS Listeners (NGINX)
1.    Open your NGINX configuration file using a text editor. The location is typically /etc/nginx/nginx.conf.
2.    In the LogFormat section, add $http_x_forwarded_for, similar to the following:
```
http {
    ...
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    ...
}
```
```
systemctl reload nginx
```

# How can I redirect HTTP requests to HTTPS using an Application Load Balancer?
```
server {
    listen 80;
    server_name _;
    if ($http_x_forwarded_proto = 'http'){
    return 301 https://$host$request_uri;
    }
}
```
```
systemctl reload nginx
```
