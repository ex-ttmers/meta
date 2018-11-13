Adhoc reports are here: http://warehouse.thinkthroughmath.com/adhoc/new

Check the adhoc queue in http://warehouse.thinkthroughmath.com/sidekiq/workers.

```
ttmscalr psql warehouse -f dw-prod
```

Use pg_stat_activity to find queries that could be it:

```sql
select * from pg_stat_activity where application_name not like '%unicorn%' and application_name not like '%etl%' and state = 'active';
```

If you find a pid that's likely you can kill it:

```sql
select pg_cancel_backend(<pid>);
```

Usage report, for example, spends a lot of time in ruby mapping fields. So it's possible there's no query to kill. In this case, we need to kill and restart the reports sidekiq.

```
ttmscalr restart reports.1 -f dw-prod
```

Make sure that you clear the adhoc queue from [the Sidekiq dashboard](http://warehouse.thinkthroughmath.com/sidekiq/queues) (you can just `Delete` the adhoc queue from there -- it'll come back) after you restart but before sidekiq comes back up, or sidekiq will retry the job that got stuck.

If *THAT* doesn't work, you may have to "clear the workers list". There is a button for that on the "busy" page, but be careful not to clear an ETL job.
