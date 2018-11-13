### Nginx Configuration for Local Development

#### Why do we need this?

When developing locally, typically using `localhost` to address your development servers works just fine. However, once multiple services are in place and once you are making "cross-domain" requests, accessing applications at `localhost` is problematic. Browser security measures will prevent certain types of activities from occurring at `localhost`, so it becomes necessary to run your local development services on a non-localhost domain. Historically, we have used pow for this. While it is simple to set up, it has some drawbacks. For example, it does not support websocket connections. This document outlines how to use nginx to act as a local development proxy instead of a tool like pow.

These instructions were adapted from http://zaiste.net/2013/03/serving_apps_locally_with_nginx_and_pretty_domains/

- Uninstall pow, and install nginx and dnsmasq.

```bash
curl get.pow.cx/uninstall.sh | sh
brew install nginx
brew install dnsmasq
```

- Edit `/usr/local/etc/nginx/nginx.conf`. Change it to run on port 80 and add app to server_name:

```
    server {
        ...
        listen       80;
        server_name  app localhost;
        ...
```

- Use dnsmasq to resolve .dev domain to localhost, and start dnsmasq automatically:

```
echo "address=/.dev/127.0.0.1" > /usr/local/etc/dnsmasq.conf
sudo cp -fv /usr/local/opt/dnsmasq/*.plist /Library/LaunchDaemons
sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist
```

- Add your machine to your list of DNS servers in the System | Network | Advanced tab.

![](https://www.evernote.com/shard/s286/sh/bb466077-85e5-494c-9d30-93c5e38c67e2/ead91dde56fa37d3b4f875b31889754e/deep/0/network_dns.png)

NOTE: Ensure that it is at the top of the list. Also, if you use Wi-Fi and wired connections, you will have to do this for
both interfaces.

- Ensure the DNS resolution is working.

```
$ ping example.dev
PING example.dev (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.030 ms
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.168 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.155 ms
```

- Create a file for each virtual domain you want to use:

`/usr/local/etc/nginx/servers/apangea.dev`

```nginx
upstream apangea {
    server 127.0.0.1:5000;
}

server {
    listen 80;
    server_name apangea.dev;
    root CHANGE_ME_CHANGE_ME; #e.g. /users/seejee/code/apangea/public;

    try_files $uri/index.html $uri.html $uri @app;

    location @app {
      proxy_set_header x-forwarded-for $proxy_add_x_forwarded_for;
      proxy_set_header host $http_host;
      proxy_redirect off;

      proxy_pass http://apangea;
    }
}
```

`/usr/local/etc/nginx/servers/live-teaching.dev`

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

upstream live-teaching.apangea {
    server 127.0.0.1:5050;
}

server {
    listen 80;
    server_name live-teaching.apangea.dev;
    root CHANGE_ME_CHANGE_ME; # e.g. /users/seejee/code/live_teaching/public;

    try_files $uri/index.html $uri.html $uri @app;

    location @app {
      proxy_set_header x-forwarded-for $proxy_add_x_forwarded_for;
      proxy_set_header host $http_host;
      proxy_redirect off;

      proxy_pass http://live-teaching.apangea;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
    }
}
```

NOTE: Be sure to change the "CHANGE_ME" directives above to match your local machine directory structure.

NOTE: The upgrade and connection stuff is to support web socket proxying.

- Start nginx.

```
sudo nginx
```

NOTE: In order to run on port 80, nginx needs root access. I have not been able to get this to work properly when using OSX's launchctl. As a result, you'll need to start nginx manually every time you reboot your machine until we figure out a way to solve
this.

- After making changes to nginx config, you can restart it via:

```
sudo nginx -s reload
```

- Test it!

```
$ curl apangea.dev
<html><body>You are being <a href="http://apangea.dev/users/sign_in">redirected</a>.</body></html>

$ curl live-teaching.apangea.dev
Found. Redirecting to /apangea.dev/users/sign_in
```


