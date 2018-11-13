### Restoring Scalr databases locally

If you have a [my.scalr.com](https://my.scalr.com/) account, you can
log in and download backups from our AWS environments.

1. log in to [my.scalr.com](https://my.scalr.com/)
2. click Farms
3. click the "actions" button for "Production-DB-Master" and select PostgreSQL Status
4. click the manage button in the Database Dump window
5. click the backup that you want, and download all the parts to the same directory
6. open a terminal in the directory that you have the parts in, and run the following commands

        $ cat *.part.* >> fullDB.tar.gz
        $ tar -zxvf fullDB.tar.gz
        $ createdb apangea_development
        $ psql -d apangea_development -c "create role u1rtbpnu2rctb6 with superuser"
        $ psql -f scaler_master.sql -d apangea_development

7. that will get you the master shard. Repeat the steps for the enrollment shards

Note that in addition to time you'll need a lot of diskspace
(~10GB in DB_DIR, and ~100GB in your Postgres data directory).

But what if you already have a set of sharded DB backup files
(say, from a friendly co-worker). You'll need one for each
`database.yml`-configured database. Just put them in a directory
(in this example, named `~/Downloads/shard_dump`) and then run
this rake task to restore them all:

    $ bundle exec rake ttm:pg:restore_sharded DB_DIR='~/Downloads/shard_dump'

This may also take a really long time, but less then downloading
the snapshots yourself.

And at that point you can delete the downloaded snapshots if you
don't need them -- or better yet, archive them to your external
hard drive you have sitting around doing nothing.

