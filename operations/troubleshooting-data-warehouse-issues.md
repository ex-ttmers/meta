## Troubleshooting Data Warehouse Query Timeouts

The warehouse runs hundreds of thousands of queries per day. Most of the time these queries are handled quickly and without, but occasionally queries begin to time out and apangea experience errors. This document describes a series of steps that can be taken in order to identify and resolve the root cause of these slow queries.

If you are seeing errors in production, here is the series of things to check:

### 1) An expensive ad-hoc report is running

- Adhoc reports are here: http://warehouse.thinkthroughmath.com/adhoc/new
- Check the adhoc queue in http://warehouse.thinkthroughmath.com/sidekiq/workers

```
ttmscalr psql warehouse -f dw-prod
```

Use pg_stat_activity to find queries that could be the culprit:

```sql
select * from pg_stat_activity where application_name not like '%unicorn%' and application_name not like '%etl%' and state = 'active';
```

If you find a pid that's likely you can kill it:

```sql
select pg_cancel_backend(<pid>);
```

Some report spend a lot of time in ruby mapping fields. So it's possible there's no query to kill. In this case, we need to kill and restart the reports sidekiq.

```
ttmscalr restart reports.1 -f dw-prod
```

Make sure that you clear the adhoc queue from [the Sidekiq dashboard](http://warehouse.thinkthroughmath.com/sidekiq/queues) (you can just `Delete` the adhoc queue from there -- it'll come back) after you restart but before sidekiq comes back up, or sidekiq will retry the job that got stuck.

If *THAT* doesn't work, you may have to "clear the workers list". There is a button for that on the "busy" page, but be careful not to clear an ETL job.
