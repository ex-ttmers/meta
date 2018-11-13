First, update brew and install nginx via brew:

    brew update
    brew install nginx

Build a fake ssl cert and install it (much of this info is taken from [[https://gist.github.com/trcarden/3295935]])

    mkdir /usr/local/etc/nginx/ssl/
    cd /usr/local/etc/nginx/ssl/

Let's make a key! Single command all on one line. You should have two files at the end. localhost.key and localhost.crt

     openssl req -new -newkey rsa:2048 -sha1 -days 365 -nodes -x509 -subj "/C=US/ST=Pennsylvania/L=Pittsburgh/O=SW/CN=localhost" -keyout localhost.key -out localhost.crt

Back up the original config (just in case)

    cd /usr/local/etc/nginx
    cp nginx.conf nginx.conf.orig

Edit nginx.conf to use ssl termination and passthrough: (This config is from TTM Production with some minor edits)

<pre>
# Source - Default nginx config file shipped with unicorn - 
# https://github.com/defunkt/unicorn/blob/master/examples/nginx.conf

# The only setting we feel strongly about is the fail_timeout=0
# directive in the "upstream" block.  max_fails=0 also has the same
# effect as fail_timeout=0 for current versions of nginx and may be
# used in its place.

# you generally only need one nginx worker unless you're serving
# large amounts of static files which require blocking disk reads
worker_processes 1;

# Drop privileges, root is needed on most systems for binding to port 80
# (or anything less than 1024).  Capability-based security may be available for
# your system and worth checking out so you won't need to be root to
# start nginx to bind on 80
# user andre staff;

# Feel free to change all paths to suite your needs here, of course
pid /usr/local/var/run/nginx/nginx.pid;

error_log /usr/local/var/log/nginx/nginx.error.log;

events {
  worker_connections 1024; # increase if you have lots of clients
  accept_mutex off; # "on" if nginx worker_processes > 1
  use kqueue; # enable for Linux 2.6+, use kqueue for FreeBSD, OSX
}

http {
  # nginx will find this file in the config directory set at nginx build time
  include mime.types;

  # fallback in case we can't determine a type
  default_type application/octet-stream;

  # click tracking!
  access_log /usr/local/var/log/nginx/nginx.access.log combined;

  # you generally want to serve static files with nginx since neither
  # Unicorn nor Rainbows! is optimized for it at the moment
  sendfile on;

  tcp_nopush on; # off may be better for *some* Comet/long-poll stuff
  tcp_nodelay off; # on may be better for some Comet/long-poll stuff

  # this can be any application server, not just Unicorn/Rainbows!
  upstream app_server {
    # fail_timeout=0 means we always retry an upstream even if it failed
    # to return a good HTTP response (in case the Unicorn master nukes a
    # single worker for timing out).

    # for TCP setups, point these to your backend servers
    # server 192.168.0.7:8080 fail_timeout=0;
    # server 192.168.0.8:8080 fail_timeout=0;
    # server 192.168.0.9:8080 fail_timeout=0;

    # for UNIX domain socket setups:
    # server unix:/usr/local/var/run/nginx/.unicorn.sock fail_timeout=0;

    server localhost:5000;

   }
 
  server {
    # enable one of the following if you're on Linux or FreeBSD
    # listen 80 default accept_filter=httpready; # for FreeBSD
  
    listen 8080 default;    
    listen 8443 ssl;

    ssl_session_timeout   10m;
    ssl_session_cache   shared:SSL:10m;
    ssl_protocols   SSLv2 SSLv3 TLSv1;
    ssl_ciphers   ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
    ssl_prefer_server_ciphers   on;
    ssl_certificate /usr/local/etc/nginx/ssl/localhost.crt;
    ssl_certificate_key /usr/local/etc/nginx/ssl/localhost.key;
      
    # If you have IPv6, you'll likely want to have two separate listeners.
    # One on IPv4 only (the default), and another on IPv6 only instead
    # of a single dual-stack listener.  A dual-stack listener will make
    # for ugly IPv4 addresses in $remote_addr (e.g ":ffff:10.0.0.1"
    # instead of just "10.0.0.1") and potentially trigger bugs in
    # some software.
    # listen [::]:80 ipv6only=on; # deferred or accept_filter recommended

    client_max_body_size 4G;
    server_name _;

    # ~2 seconds is often enough for most folks to parse HTML/CSS and
    # retrieve needed images/icons/frames, connections are cheap in
    # nginx so increasing this is generally safe...
    keepalive_timeout 5;

    # path for static files
    root /Users/andre/src/apangea/public;

    # Maintenenace Mode
    recursive_error_pages on;
    if (-f /usr/local/var/run/nginx/__maintenance__) {
      rewrite  (version|logo\.png|style\.css)$ /$1 break;
      rewrite ^(.*)$ /maintenance.html break;
    }

    underscores_in_headers on;
        
    # Prefer to serve static files directly from nginx to avoid unnecessary
    # data copies from the application server.
    #
    # try_files directive appeared in in nginx 0.7.27 and has stabilized
    # over time.  Older versions of nginx (e.g. 0.6.x) requires
    # "if (!-f $request_filename)" which was less efficient:
    # http://bogomips.org/unicorn.git/tree/examples/nginx.conf?id=v3.3.1#n127
    try_files $uri/index.html $uri.html $uri @app;

    location @app {
    
      # Allows NewRelic to tract time spent in Nginx
      # https://newrelic.com/docs/features/request-queuing-and-tracking-front-end-time
      # Use Amazon CloudWatch to trac ELB stats
      proxy_set_header X-Request-Start "t=${msec}000";
    
      # an HTTP header important enough to have its own Wikipedia entry:
      #   http://en.wikipedia.org/wiki/X-Forwarded-For
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

      # enable this if you forward HTTPS traffic to unicorn,
      # this helps Rack set the proper URL scheme for doing redirects:
      # proxy_set_header X-Forwarded-Proto $scheme;

      # pass the Host: header from the client right along so redirects
      # can be set properly within the Rack application
      proxy_set_header Host $http_host;

      # The lesson player will set custom XSRF tokens from the cookie on each
      # request. Pass them through to rails.
      proxy_set_header X_XSRF_TOKEN $http_x_xsrf_token;

      # we don't want nginx trying to do something clever with
      # redirects, we set the Host: header above already.
      proxy_redirect off;

      # set "proxy_buffering off" *only* for Rainbows! when doing
      # Comet/long-poll/streaming.  It's also safe to set if you're using
      # only serving fast clients with Unicorn + nginx, but not slow
      # clients.  You normally want nginx to buffer responses to slow
      # clients, even with Rails 3.1 streaming because otherwise a slow
      # client can become a bottleneck of Unicorn.
      #
      # The Rack application may also set "X-Accel-Buffering (yes|no)"
      # in the response headers do disable/enable buffering on a
      # per-response basis.
      # proxy_buffering off;

      proxy_pass http://app_server;
    }

    # Rails error pages
    error_page 500 502 503 504 /500.html;
    location = /500.html {
      root /Users/andre/src/apangea/public;
    }

    location = /maintenance.html {
      root /var/www/public;
      error_page 405 = /maintenance.html;
    }

    location ~ (\.php|.aspx|.asp|.exe|myadmin|health) {
      return 404;
    }

    location ~ /lesson_player/? {
      root /Users/andre/src;
      autoindex on;
    }
  }
}
</pre>

Per user customization:
* To run on port 80 and 443, change the listen lines under server. If you make this change stat nginx with sudo nginx. Otherwise start without sudo. (Port 80 and 443 are privileged ports.)
* Change paths, search for /Users/andre and update as needed.
* For the lesson player make a symlink in your home directory to the dist folder.
 
            ln -s MY_LESSON_PLAYER_GIT_FOLDER/dist lesson_player

Now add nginx to startup and reload

    mkdir -p ~/Library/LaunchAgents
    cp /usr/local/Cellar/nginx/1.2.7/homebrew.mxcl.nginx.plist ~/Library/LaunchAgents/
    launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist

Then you should be able to go to https://localhost:8443, add a security exception, and go!

If you don't want to add nginx to startup

    Start nginx with `$ nginx`
    Stop nginx with `$ nginx -s stop`
    the `-s` switch allows you to send the stop signal whenever you want
    and use sudo if you configured to run on port 80 and 443

See André with questions or issues.
