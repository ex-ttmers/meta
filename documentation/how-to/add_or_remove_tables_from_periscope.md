# Adding / Removing Tables from Periscope

We currently have tables from both apangea (`enrollments`) and the data
warehouse (`warehouse`) in our periscope caches, but not every table is in the
cache.  Sometimes you may need to add or remove tables from the cache.

## Tables in the Cache

First, check to see which tables are in the cache.  You can do this by

1.  Clicking the 'server' icon in the top right of the periscope ui 
2.  Select the database you are interested in looking at a table from (either enrollments or warehouse)
3.  Click the 'cache' tab.
4.  Check the list of tables.  It will tell you about the last time it was cached.

## Adding or removing a table

Periscope adds all the tables it can read to the cache at predetermined intervals.  

-  If you want to add a table that isn't there, you need to `GRANT SELECT` privileges to the periscope user.
-  If you want to remove a table that is there, you need to `REVOKE SELECT` read privileges from the periscope user.
-  You can find the periscope user's info on the 'connection' tab under the server hamburger.

To grant/revoke, just psql into the appropriate prod server and execute the query.

If your table doesn't automagically [dis]appear, you can click the little 'refresh' icon in the 'status' box under the 'connections' tab, and your newly changed table perms should take effect.

## Setting Cache Options

Once your table shows up, it will probably list as 'uncached' in the 'cache' tab.  If you expand the info, you should be able to select a strategy (full, incremental, or both), and this will start the caching.  Details here: https://doc.periscopedata.com/doc/caching-strategies
