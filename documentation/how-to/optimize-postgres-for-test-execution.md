# Test Speedups

By modifying a few Postgres server settings, I was able to reduce
the test execution time of the integration tests from 22 minutes
to 10 minutes.

## Postgres performance gains

The change involves turning off the disk flushing in
Postgres. This is a terrible thing to do if you care about your
data, so this setting should only be used in development and
test. To modify your server so that it might lose data at any
point (but will be really fast!) do this:

**If you installed Postgres via homebrew:**

 1. Edit your postgres config at `/usr/local/var/postgres/postgresql.conf` to include:

        fsync = off               # turns forced synchronization on or off
        synchronous_commit = off  # synchronization level;
        full_page_writes = off    # recover from partial page writes
        bgwriter_lru_maxpages = 0

 2. Restart your Postgres server:

        pg_ctl -D /usr/local/var/postgres stop -s -m fast
        pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start

    Or if you use launch control:

        launchctl unload ~/Library/LaunchAgents/org.postgresql.postgres.plist
        launchctl load ~/Library/LaunchAgents/org.postgresql.postgres.plist

**If you're using Postgres.app:**

 1. Edit your postgres config at `~/Library/Application Support/Postgres/var/postgresql.conf` to include:

        fsync = off               # turns forced synchronization on or off
        synchronous_commit = off  # synchronization level;
        full_page_writes = off    # recover from partial page writes

 2. Restart Postgres.app. _Note: There's an open ticket ([[https://github.com/mattt/PostgresApp/issues/56]]) regarding config changes not taking effect. So, you know, proceed with caution._


**If you're on Linux and installed Postgres via apt-get:**

 1. Edit your postgres config at `/etc/postgresql/9.2/main/postgresql.conf` to include:

        fsync = off               # turns forced synchronization on or off
        synchronous_commit = off  # synchronization level;
        full_page_writes = off    # recover from partial page writes
        bgwriter_lru_maxpages = 0

 2. Restart your Postgres server:

        sudo /etc/init.d/postgresql restart 9.2

## Other performance gains

I am also running a version of Ruby patched for performance. This
patch dramatically reduces rails boot time. Instructions to
install this version are here:

[[Installing a performance patched version of Ruby]]

[Installing a performance patched version of Ruby for those slumming with rbenv](https://gist.github.com/1688857?utm_source=rubyweekly&utm_medium=email)

 
