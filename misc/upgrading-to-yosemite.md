# Getting Apangea to Work After Upgrading to OS X Yosemite

After upgrading to Yosemite, I had to do a few things to get back to a working state.  I am documenting them here to prevent others from having to research these issues.

* For some reason, Yosemite deletes some necessary Postgres directories during the upgrade.  They will need to be re-created (note: not all of these may have been deleted, I think for me only the first and last were; other people online had different experiences, though)

```bash
mkdir /usr/local/var/postgres/pg_tblspc
mkdir /usr/local/var/postgres/pg_twophase
mkdir /usr/local/var/postgres/pg_stat_tmp
```

* You may get messages that Solr can't find Java.  Install a new Java __JDK__ (NOT the JRE).

* If you had set the max number of processes in /etc/launchd.conf, you will have to delete those and do it via sysctl in your .bashrc or something similar.
