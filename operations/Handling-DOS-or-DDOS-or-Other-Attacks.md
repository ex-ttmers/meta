## Rack Attack
_______
DOS/DDOS Attacks are extremly had to mitigate. It may be next to impossilbe to deter a determined hacker. That being said we
have a gem called [Rack Attack] (https://github.com/kickstarter/rack-attack) that can be used to mitigate small scale attacks.

Rack Attack supports various types of blocking. If you know an IP Address you can configure rack attack to block it or
you can configre it to block access to a specific route. 

You can also configure rack attack to look up IPs from [rails cache] (https://github.com/kickstarter/rack-attack/wiki/Advanced-Configuration). To store an IP for blacklist in the rails cache do the following

``````````
ttmscalr rails_c -f FARM-NAME
Rails.cache.write('block PUT.IP.ADDRESS.HERE', true, expires_in: 3.days)
``````````

Rack Attack also supports throttling. Sometimes a throttle might be more effective than a block.

The configuration is stored in `config/initializers/rack_attack.rb`.

A deploy is required to make the changes active.

## We got some ones old IP
_________
Sometimes when we scale up webservers we might get an IP that was previously used for some service or API. For whatever
reason that IP is still being used by clients of said service even though its now a TTM IP. This might create a flood of unwanted traffic at a particular server.

The easisest way to deal with this is to terminate the server in scalr and let it respawn with a new IP.
