### From http://clock.co.uk/tech-blogs/how-to-create-a-private-npmjs-repository

npmjs.org never goes down
Or so I thought.

I was going to do some development on my local machine, so I fired up npm install <packagename>. Unfortunately, due to a npmjs.org outage, it was not possible for me to get on with my work. So I did what any driven developer would do; I set up a CouchDB replica of npmjs.org. Next time this happens I will be prepared!

As this npmjs.org replica is hosted in the same datacenter as we deploy most of our sites to, it enables a super speedy deployment to testing.

How do we go about this you ask? Well let me tell you.

### Installing CouchDB
Note: These instruction are geared towards Ubuntu 12.04 LTS. But it should be pretty easy to get it up and running on *nix systems, including Mac OS X.

* Install the required packages:

        sudo apt-get install build-essential autoconf automake libtool erlang libicu-dev libmozjs-dev libcurl4   openssl-dev 

* Download CouchDB 1.2:

        wget http://mirrors.ukfast.co.uk/sites/ftp.apache.org/couchdb/releases/1.2.0/apache-couchdb-1.2.0.tar.gz 

* Extract, and relax:

        tar xfv apache-couchdb-1.2.0.tar.gz
 
* Now time to compile:

        cd apache-couchdb-1.2.0
        ./configure
        make
        make install

After all that is done we now want to check that everything's fine and dandy, and that we get the expected output:

        $ couchdb
        Apache CouchDB 1.2.0 (LogLevel=info) is starting.
        Apache CouchDB has started. Time to relax.
        [info] [<0.32.0>] Apache CouchDB has started on http://127.0.0.1:5984/ 
       
Sweet! But are we responding to requests?

        $ curl -X GET http://localhost:5984
        [info] [<0.361.0>] 127.0.0.1 - - GET / 200
Boom, we now have a working CouchDB instance!

You: But wait kind sir, what if we reboot the server? Wouldn't we have to start CouchDB again?

That's correct, we will be adding CouchDB to our init.d scripts, but first we need to create the correct user, group and permissions:

        $ sudo adduser --disabled-login --disabled-password --no-create-home couchdb
        sudo chown -R couchdb:couchdb /usr/local/var/{log,lib,run}/couchdb
        sudo chown -R couchdb:couchdb /usr/local/etc/couchdb/local.ini

We also want CouchDB to use insecure rewrites for a later step. We can turn this off by editing /usr/local/etc/couchdb/local.ini and adding secure_rewrites = false on line 11 in the httpd section.

       $ sudo vim /usr/local/etc/couchdb/local.ini
         [httpd]
         secure_rewrites = false

Ready, set, link, defaults.

        sudo ln -s /usr/local/etc/init.d/couchdb /etc/init.d
        sudo update-rc.d couchdb defaults 

Go!

        sudo /etc/init.d/couchdb start 

YAY!

Replicating npmjs.org
Now to replicate the npm registry.

        curl -X POST http://127.0.0.1:5984/_replicate -d '{"source":"http://isaacs.iriscouch.com/registry/", "target":"registry", "create_target":true}' -H "Content-Type: application/json" 

What this means is that once this is completed your local replication will be an exact copy of the npm registry. However, to ensure we do indeed receive all updates we will add a "continuous":true parameter to the JSON string in our POST request, this utilises CouchDB’s _changes API and will pull any new changes when this API is notified.

        curl -X POST http://127.0.0.1:5984/_replicate -d '{"source":"http://isaacs.iriscouch.com/registry/", "target":"registry", "continuous":true, "create_target":true}' -H "Content-Type: application/json" 

We are now replicating continuously from npmjs.org to our private CouchDB instance! If you ever want to stop these replications you can easily do this by running the same command as before but add a "cancel":true parameter to the JSON POST data.

        curl -X POST http://127.0.0.1:5984/_replicate -d '{"source":"http://isaacs.iriscouch.com/registry/", "target":"registry", "continuous":true, "create_target":true, "cancel":true}' -H "Content-Type: application/json"

And we're almost done! All we need to do now is to set up our own version of the npmjs.org registry and we can relax like the humanoid in the CouchDB logo, or as we do @clock, melt into our beanbags...

Getting npm to work with our replicated CouchDB
Most of the steps can be found on isaacs github in the npmjs.org git repositories README

Of course we need to have nodejs and git installed for this:

        git clone git://github.com/isaacs/npmjs.org.git
        cd npmjs.org
        sudo npm install -g couchapp 
        npm install couchapp 
        npm install semver 
        couchapp push registry/app.js http://localhost:5984/registry 
        couchapp push www/app.js http://localhost:5984/registry 

Boom, we now have a working npm repository, to test this we can run the following command.

       npm --registry http://localhost:5984/registry/_design/scratch/_rewrite login
       npm --registry http://localhost:5984/registry/_design/scratch/_rewrite search

If you are getting results then everything has gone according to plan!

So we now have your own privately hosted npm registry, that keeps itself updated. Pretty neat, eh?

All you have to get up and running on your own subdomain is to modify the [vhosts] section in /usr/local/etc/couchdb/local.ini. Uncomment the example and restart CouchDB.

        $ vim /usr/local/etc/couchdb/local.ini
          [vhosts]
          example.com = /registry/_design/scratch/_rewrite

And while we are at it will lock down the application and prevent unauthorized users from deleting our data.

        $ vim /usr/local/etc/couchdb/local.ini
         [admins]
         admin = password
        $ sudo /etc/init.d/couchdb restart 

Start using your version of npm with the client!
Straight from the npmjs.org README, just replace <registryurl> with your registries url, for example:
http://localhost:5984/registry/_design/app/_rewrite

You can point the npm client at the registry by putting this in your ~/.npmrc file:

registry = <registryurl>
You can also set the npm registry config property like:

npm config set <registryurl>
Or you can simple override the registry config on each call:

npm --registry <registryurl> install <packagename>
Now you can code and install modules even if npmjs.org is down, or if you want to push the boat even further have this running on your local machine and have an up-to-date npm registry just before your flight. ;]
