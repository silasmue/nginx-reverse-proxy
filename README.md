# nginx-reverse-proxy
Guide to setup an nginx server (as reverse proxy).
First setup the server block for your (static) website:
```
server {
        server_name  example.com;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
}
```
After that setup the server block(s) for your services (running in containers):
```
server {
    server_name git.example.com;
        
    location / {
        proxy_pass http://192.168.188.195:3000/;
        include proxy_params;
    }
```
The included `proxy_params:
```
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```
Run `nginx -t` to check the configuration.
Then run `sudo setsebool -P httpd_can_network_relay 1` to enable forwarding to the service(s).
After that you may run certbot --nginx to enable https via [Let's encrypt](https://letsencrypt.org/de/)
Finally you can (re)start your nginx: 
```
systemctl restart nginx
```
# References
https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04
