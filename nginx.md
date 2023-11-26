# Nginx
![Alt text](images/image-nginx.png)

NGINX is a powerful web server and uses a non-threaded, event-driven architecture that enables it to outperform Apache if configured correctly. It can also do other important things, such as load balancing, HTTP caching, or be used as a reverse proxy.

<u>Basic Installation</u>

<u>Configuration Settings</u>

The core settings of NGINX are in the `nginx.conf` file, which by default looks like this:
```conf
user www-data;
worker_processes 4;
pid /run/nginx.pid;

events { 
    worker_connections 768;
    # multi_accept on;
}

http {

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    gzip on;
    gzip_disable "msie6";

    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

This file is structured into **Contexts**. The first one is the **events** Context, and the second one is the **http** Context. This structure enables some advanced layering of your configuration as each context can have other nested contexts that inherit everything from their parent but can also override a setting as needed.

Some of the most important pieces of the NGINX config file are:

- **worker_processes**: This setting defines the number of worker processes that NGINX will use. Because NGINX is single threaded, this number should usually be equal to the number of CPU cores.
- **worker_connections**: This is the maximum number of simultaneous connections for each worker process and tells our worker processes how many people can simultaneously be served by NGINX. The bigger it is, the more simultaneous users the NGINX will be able to serve.
- **access_log & error_log**: These are the files that NGINX will use to log any errors and access attempts. These logs are generally reviewed for debugging and troubleshooting.
- **gzip**: These are the setting for GZIP compression of NGINX responses. Enabling this one along with the various sub-settings that by default are commented out will result in a quite a big performance upgrade. From the sub-settings of GZIP, care should be taken for the gzip_comp_level, which is the level of compression and ranges from 1 to 10. Generally, this value should not be above 6 - the gain in terms of size reduction is insignificant, as it needs a lot more CPU usage. gzip_types is a list of response types that compression will be applied on.

Your NGINX install can support far more than a single website, and the files that define your server's sites live in the /etc/nginx/sites-available directory. But NGINX won't actually do anything with them unless they're symlinked into the /etc/nginx/sites-enabled directory - when you're ready for a site to go online, symlink it into sites-enabled and restart NGINX.

The `sites-available` directory includes configurations for virtual hosts. This allows the web server to be configured for multiple sites that have separate configurations. The sites within this directory are not live and are only enabled if we create a symbolic link into the `sites-enabled` folder.

Either create a new file for your application or edit the default one. A typical configuration looks like the below one.

```
upstream remoteApplicationServer {
    server 10.10.10.10;
}

upstream remoteAPIServer {
    server 20.20.20.20;
    server 20.20.20.21;
    server 20.20.20.22;
    server 20.20.20.23;
}

server {
    listen 80;
    server_name www.customapp.com customapp.com
    root /var/www/html;
    index index.html

        location / {
            alias /var/www/html/customapp/;
            try_files $uri $uri/ =404;
        }

        location /remoteapp {
            proxy_set_header    Host            $host:$server_port;
            proxy_set_header    X-Real-IP       $remote_addr;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://remoteAPIServer/;
        }

        location /api/v1/ {
            proxy_pass https://remoteAPIServer/api/v1/;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_redirect http:// https://;
        }
}
```

Much like the `nginx.conf`, this one also uses the concept of nested contexts (and all of these are also nested inside the **HTTP** context of nginx.conf, so they also inherit everything from it).

The **server** context defines a specific virtual server to handle your clients' requests. You can have multiple server blocks, and NGINX will choose between them based on the `listen` and `server_name` directives.

Inside a server block, we define multiple **location** contexts that are used to decide how to handle the client requests. Whenever a request comes in, NGINX will try to match its URI to one of those location definitions and handle it accordingly.

There are many important directives that can be used under the location context, such as:

- **try_files** will try to serve a static file found under the folder that points to the root directive.
- **proxy_pass** will send the request to a specified proxied server.
- **rewrite** will rewrite the incoming URI based on a regular expression so that another location block will be able to handle it.

The **upstream** context defines a pool of servers that NGINX will proxy the requests to. After we create an upstream block and define a server inside it we can then reference it by name inside our location blocks. Furthermore, an upstream context can have many servers assigned under it so that NGINX will do some load balancing when proxying the requests.

#### Start NGINX
we can start up NGINX using the below command:

```bash
sudo service nginx start
```

After that, whenever we change something on our configuration, we only have to reload it (without downtime) using the command:

```bash
service nginx reload
```

Lastly, we can check NGINX's status using the command:

```bash
service nginx status
```