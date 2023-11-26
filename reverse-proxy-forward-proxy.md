# Reverses proxy / Forward proxy

> a forward proxy hides the identities of clients, a reverse proxy hides the identities of servers.

## Forward Proxy
A forward proxy provides proxy services to a client or a group of clients. Moreover, these clients belong to a common internal network. In other words, it retrieves data from another website on behalf of the original requestee.

Whether or not this request is allowed depends on how the proxy server's (i.e forward proxy) settings are configured. If the request is allowed — the client is able to transfer the file over the internet. If the request is denied, it is dropped by the proxy server.

In essence, it is the proxy server here that issued the request and not the client. So when the server responded, it addresses its response to the proxy.

In a scenario where content is downloaded frequently, it can also act as a cache server where the proxy can cache the content on the server.

## Reverse Proxy
While a forward proxy works on behalf of the clients, a reverse proxy *proxies* on behalf of the backend servers. Reverse proxies were basically used to balance load/traffic and to achieve HA (i.e. High Availability). It receives the external request from frontend/clients on behalf of servers behind the proxies.

When a reverse proxy performs load balancing, it distributes incoming requests to a cluster of servers, all providing the same kind of service.

## Create the Nginx Reverse Proxy:
Once you have installed Nginx, follow the below command to disable virtual host:

```bash
sudo unlink /etc/nginx/sites-enabled/default
```

After disabling the virtual host, we need to create a file called **reverse-proxy.conf** within the **etc/nginx/sites-available** directory to keep reverse proxy information.

```bash
cd etc/nginx/sites-available/
vi reverse-proxy.conf
```

In the file we need to paste in these strings:

```conf
server {
    listen 80;
    location / {
        proxy_pass http://192.x.x.2;
    }
}
```

To pass information to other servers, you can use the `ngx_http_proxy_module` in the terminal.

Now, activate the directives by linking to `/sites-enabled/` using the following command:

```bash
sudo ln -s /etc/nginx/sites-available/reverse-proxy.conf /etc/nginx/sites-enabled/reverse-proxy.conf
```

Lastly, we need to run an Nginx configuration test and restart Nginx to check its performance

```bash
service nginx configtest
service nginx restart
```