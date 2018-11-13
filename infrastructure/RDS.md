# Amazon RDS

RDS stands for Relational Database Service, and is their managed database service. Amazon takes care of backups,
maintenance, updates, scaling, support of multiple availability zones, and lots of other fun stuff that we then have
someone to blame for instead of doing it ourselves.

Backups on amazon only take about a half hour (they're using magical internal somethings). The replicas also stay
more in sync, even under high load.

## Why are we using RDS?

Before, we were using Scalr-managed postgres. Scalr uses postgres' built-in Write Ahead Log to manage replication, 
which can get out of sync under high load. They also used amazon's snapshot mechanism to export data, which doesn't
work very well with our data because it takes a diff and, especially the warehouse, we change a LOT so these diffs
were huge. Scalr-managed databases were also all in one availability zone.

Even though we're not using Scalr for our database instances, we're still using Scalr to manage all our other 
machines since amazon doesn't get provide a way to, say, automatically bring up a replacement instance for an 
instance that died.

## Technical details you should be aware of

We take nightly snapshots of all the data in each database, and we keep the last 7 days worth of snapshots. In the 
amazon RDS dashboard, you can list these snapshots and restore them to a newinstance in order to, say, get 
production-like data on a different farm.

When using RDS, you are required to give them a window for maintenance and backups. Our window is at **6 AM 
Sundays**. If you see any database errors around this time, having to do with not being able to connect, the cause
might be amazon cutting over from one availability zone to another (they run maintenance on one at a time).

There is a command-line interface that you can install with `brew install aws`; this will let you do things like spin
up, spin down, etc. But since RDS is just part of our one amazon account, if you have access to one thing, you have 
access to everything and that gives you the possibility of doing bad things to production. If you need this power and
can handle this responsibility, talk to André about getting access.

Monitoring: you can look in Scalr, pick "RDS DB Instances", choose the instance you're interested in, then go to 
Cloud Watch Statistics, you can see various stats like CPU usage, number of connections, etc. 

The non-production farms are not in multiple availability zones. That costs money, Sam.

The RDS database servers have very specific security access groups-- each farm only has access to its database 
server, and the database servers are only accessible from their farms' IP addresses and from the office. In order to 
access the databases when you're not into the office, you can either SSH to the farm's debug role, or set up an SSH 
tunnel with `ttmscalr psql:tunnel`. We do not have ssh access to the database servers.

## PSQL into database instances
1. Run `ttmscalr psql:tunnel -f <FARM_NAME>`
2. In another terminal, run: `ttmscalr psql -f production`.
3. Immediately C-c that; it won't work. We need the username and password from this step for the next step, though.
4. run `psql -u postgres://<username-from-step-3>:<password-from-step-3@localhost:5433/apangea`

###TTM Dev uses this one weird trick to get this to work with pgAdmin
If you use pgAdmin, one way to make this easier is to pre-build the tunnel connections and save them in your server list, in a group called `Tunnels`. Use `ttmscalr config` for the farm to get the credentials and pre-build the connection with `localhost` and port `5433`. Then to connect in, all you need to run is step 1 from above, and then open your connection to the tunneled host in pgAdmin.
