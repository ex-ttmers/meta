We learned this from [Craig Kerstiens](http://www.craigkerstiens.com/).

He enabled `pg_stat_statements` on the database. so now we can run:

`select query, total_time / calls, calls from pg_stat_statements order by 2 desc limit 10;`

This lists the worst offenders in the SQL layer, sorted by
average time to complete.

## Installing pg_stat_statements

__1.__ Edit your `postgresql.conf` to include the following:

    shared_preload_libraries = 'pg_stat_statements'
    pg_stat_statements.max = 10000
    pg_stat_statements.track = all
    
__2.__ Restart postgres

__3.__ Connect to each database on which you want the extension
(typically the master + all shards and run):

    CREATE EXTENSION pg_stat_statements;

## Fixing SHHMAX issue (seen on OSX)

    FATAL:  could not create shared memory segment: Invalid argument
    DETAIL:  Failed system call was shmget(key=5432001, size=15917056, 03600).
    HINT:  This error usually means that PostgreSQL's request for a shared memory segment exceeded your     kernel's SHMMAX parameter.  You can either reduce the request size or reconfigure the kernel with larger SHMMAX.  To reduce the request size (currently 15917056 bytes), reduce PostgreSQL's shared memory usage, perhaps by reducing shared_buffers or max_connections.
	If the request size is already small, it's possible that it is less than your kernel's SHMMIN parameter, in which case raising the request size or reconfiguring SHMMIN is called for.
	The PostgreSQL documentation contains more information about shared memory configuration.

__1.__ Set SHHMAX and SHMALL temporarily

    sudo sysctl -w kern.sysv.shmmax=51539607552
    sudo sysctl -w kern.sysv.shmall=12582912

__2.__ Set SHHMAX and SHMALL to persist after reboots, just write the above lines to /etc/sysctl.conf

source: http://stackoverflow.com/questions/2017004/setting-shmmax-etc-values-on-mac-os-x-10-6-for-postgresql/10629164#10629164
