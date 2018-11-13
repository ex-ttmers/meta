# What is this?

Queries that might be useful against the data warehouse; they will
also hopefully serve as good examples of how to query for particular
features -- e.g., finding facts by date is weirder than you're used
to.

It also includes a couple of processes that could be useful when running and using a data warehouse of your very own.

## How do I rebuild just my lesson_completed_facts table locally?

You can reload just the lesson_completed_facts in your local warehouse by running this from reporting:

```bash
foreman start -f Procfile
# Then do the following while that is running:
cd etl/initial_load
./load_jobs.rb --farm local --type lesson_completed
```
This will enqueue some sidekiq jobs (this is why we used the regular procfile; it includes all the necessary queues).

## How do I rebuild just the live_help_facts table on a farm?

The following instructions have been taken (and modified) from the `reporting`
repository's readme:

https://github.com/thinkthroughmath/reporting#reloading-reviewrcproduction

Since the steps here are brief and lack expanded explanation, please view the
document above for more information.

You can reload just the live_help_facts in on a farm's warehouse by following these steps:

* Kill sidekiq on the farm. This can be done through `ttmscalr`: `$ ttmscalr shutdown:sidekiq -f dw-staging`
* Through the Scalr web UI, load at least one Data Load role. (You may need expanded permissions to do this).
  * Given a farm ID of `12345`, navigate to to the following page: https://my.scalr.com/#/farms/12345/roles
  * Edit the `DATA LOAD` role.
  * Ensure that you are viewing the edit screen for your farm and the data load role before making any changes.
  * Under the `COST METERING` section, change `MIN INSTANCES` and `MAX_INSTANCES` to 1 and save the farm.
* SSH into the debug role of the farm: `ttmscalr ssh debug.1 -f dw-staging`
* Inside a screen session, run the following commands to start a data load server:

```bash
$ cd /var/www
$ bundle exec foreman start -f Procfile.dataserver
```

**Note:** The previous command might fail if you need to bundle. Despite the warnings,
I've been assured that running `bundle` as `root` will not be an issue.

* Disconnect the screen session using `Ctrl-a d` (so that you can reattach to it later)
* Inside a new screen session, run the following commands to setup for the data load
  (Note: the server will not have your scalr keys. You should copy them from your local machine):

```bash
$ export SCALR_KEY_ID=<your KEY_ID>
$ export SCALR_ACCESS_KEY=<your ACCESS_KEY>
```

* To begin the data load, run the following commands:

```bash
$ cd /var/www/etl/initial_load
$ bundle exec ruby load_jobs.rb --farm dw-staging --type live_help
```

**Note:** If the environment hasn't been setup to run `ttmscalr` you might see
the previous command fail. I've noticed a syntax error in the `.ttm_scalr_aliases.json` file.
You need to edit the file to remove a dangling comma, then retry the previous
command.

* To monitor the progress, navigate to the sidekiq view for your farm:
http://warehouse-staging.thinkthroughmath.com/sidekiq

* Verify results using whichever method suits you (ie. `rails console`, viewing
  reports in the app, etc.)
* Begin clean-up by closing your present screen session: `$ exit`
* Remove your Scalr keys from the bash history by editing the `~/.bash_history` file.
* Reattach to your previous screen session: `$ screen -r`
* Kill the data load server: `Ctrl-c`
* Close this screen session: `$ exit`
* Log out of ssh: `$ exit`
* Restart sidekiq role: `$ ttmscalr restart sidekiq -f dw-staging`
* Through the Scalr web UI, terminate the Data Load role for this farm and restore
  the `MIN INSTANCES` and `MAX INSTANCES` values to 0 (using the same navigation steps
  above for loading the `DATA LOAD` role. (You may need expanded permissions to do this).

## What's the impact of changing definition of 'night' on minutes/attempts?

```sql
    SELECT
      SUM(
       CASE dd.is_a_weekday AND NOT td.is_evening
         WHEN true
           THEN af.time_to_solve
         ELSE 0
       END
      ) as current_day_secs,
      SUM(
        CASE dd.is_a_weekday AND NOT td.is_evening
          WHEN true THEN 1 ELSE 0 END
      ) as current_day_attempts,
      SUM(
        CASE NOT dd.is_a_weekday OR td.is_evening
          WHEN true
            THEN af.time_to_solve
          ELSE 0
        END
      ) as current_nw_secs,
      SUM(
        CASE NOT dd.is_a_weekday OR td.is_evening
          WHEN true THEN 1 ELSE 0 END
      ) as current_nw_attempts,
      SUM(
        CASE dd.is_a_weekday AND NOT (td.is_evening OR td.hour = 17 OR td.hour = 16 OR (td.hour = 15 AND td.minute >= 30))
          WHEN true THEN af.time_to_solve ELSE 0  END
      ) as changed_day_secs,
      SUM(
        CASE dd.is_a_weekday AND NOT (td.is_evening OR td.hour = 17 OR td.hour = 16 OR (td.hour = 15 AND td.minute >= 30))
          WHEN true THEN 1 ELSE 0 END
      ) as changed_day_attempts,
      SUM(
        CASE NOT dd.is_a_weekday OR td.is_evening OR td.hour = 17 OR td.hour = 16 OR (td.hour = 15 AND td.minute >= 30)
          WHEN true THEN af.time_to_solve ELSE 0 END
      ) as changed_nw_secs,
      SUM(
        CASE NOT dd.is_a_weekday OR td.is_evening OR td.hour = 17 OR td.hour = 16 OR (td.hour = 15 AND td.minute >= 30)
          WHEN true THEN 1 ELSE 0 END
      ) as changed_nw_attempts
    FROM
      attempt_facts af
      JOIN date_dimensions dd  ON af.local_date_id = dd.id
      JOIN time_dimensions td  ON af.local_time_id = td.id
    WHERE
      af.customer_id = 1460
      AND dd.sql_date BETWEEN '2013-10-14' AND '2013-10-27';

    --   Sample output:
    --    current_day_secs | current_day_attempts | current_nw_secs | current_nw_attempts | changed_day_secs | changed_day_attempts | changed_nw_secs | changed_nw_attempts
    --   ------------------+----------------------+-----------------+---------------------+------------------+----------------------+-----------------+---------------------
    --           845124400 |             38648845 |       159434737 |             5792676 |        749821453 |             34617361 |       254737684 |             9824160
    --
```

Same thing, but summarize by date and for a particular district:

```sql
    SELECT
      dd.date,
      SUM(
       CASE dd.is_a_weekday AND NOT td.is_evening
         WHEN true
           THEN af.time_to_solve
         ELSE 0
       END
      ) as current_day_secs,
      SUM(
        CASE dd.is_a_weekday AND NOT td.is_evening
          WHEN true THEN 1 ELSE 0 END
      ) as current_day_attempts,
      SUM(
        CASE NOT dd.is_a_weekday OR td.is_evening
          WHEN true
            THEN af.time_to_solve
          ELSE 0
        END
      ) as current_nw_secs,
      SUM(
        CASE NOT dd.is_a_weekday OR td.is_evening
          WHEN true THEN 1 ELSE 0 END
      ) as current_nw_attempts,
      SUM(
        CASE dd.is_a_weekday AND NOT (td.is_evening OR td.hour = 17 OR td.hour = 16 OR (td.hour = 15 AND td.minute >= 30))
          WHEN true THEN af.time_to_solve ELSE 0  END
      ) as changed_day_secs,
      SUM(
        CASE dd.is_a_weekday AND NOT (td.is_evening OR td.hour = 17 OR td.hour = 16 OR (td.hour = 15 AND td.minute >= 30))
          WHEN true THEN 1 ELSE 0 END
      ) as changed_day_attempts,
      SUM(
        CASE NOT dd.is_a_weekday OR td.is_evening OR td.hour = 17 OR td.hour = 16 OR (td.hour = 15 AND td.minute >= 30)
          WHEN true THEN af.time_to_solve ELSE 0 END
      ) as changed_nw_secs,
      SUM(
        CASE NOT dd.is_a_weekday OR td.is_evening OR td.hour = 17 OR td.hour = 16 OR (td.hour = 15 AND td.minute >= 30)
          WHEN true THEN 1 ELSE 0 END
      ) as changed_nw_attempts
    FROM
      attempt_facts af
      JOIN date_dimensions dd  ON af.local_date_id = dd.id
      JOIN time_dimensions td  ON af.local_time_id = td.id
    WHERE
      af.district_id = 1319
      AND dd.sql_date BETWEEN '2013-10-01' AND '2013-10-31'
    GROUP BY
      dd.date;
```
## Delete students that have already been deleted from Apangea

First dump all the students from apangea:

    cwinters@abita:~/Projects/TTM/apangea$ ttmscalr psql master -f production
    Executing `psql postgres://who:me@ext.master.postgresql.snozzberry.scalr-dns.net:5432/apangea`
    psql (9.3.1, server 9.2.5)
    SSL connection (cipher: DHE-RSA-AES256-SHA, bits: 256)
    Type "help" for help.
    apangea=> \copy (select id, is_active from students) to 'active_students.txt';
    Time: 8470.370 ms

Then connect to the warehouse:

    cwinters@abita:~/Projects/TTM/apangea$ ttmscalr psql warehouse -f dw-production
    Executing `psql postgres://who:me@ext.master.postgresql.snickerdoodle.scalr-dns.net/warehouse`
    psql (9.3.1, server 9.2.5)
    SSL connection (cipher: DHE-RSA-AES256-SHA, bits: 256)
    Type "help" for help.

Create a temporary table to hold the Apangea students:

    warehouse=> create table apangea_students(source_id int not null primary key, is_active bool);
    CREATE TABLE
    warehouse=> \copy apangea_students from 'active_students.txt'
    COPY

Then a temporary one for the warehouse students, stocking it with the
unique `source_id` of the students it knows about:

    warehouse=> create table warehouse_students(source_id int not null primary key);
    CREATE TABLE
    warehouse=> insert into warehouse_students select distinct source_id from student_dimensions;
    INSERT

Then delete every record from that table for which we have a
`source_id` from Apangea:

    warehouse=> delete from warehouse_students using apangea_students where apangea_students.source_id = warehouse_students.source_id;
    DELETE

Every record leftover is one not found in Apangea. So they can be deleted
from the `student_dimensions` table:

    warehouse=> begin;
    BEGIN
    warehouse=> delete from student_dimensions using warehouse_students where warehouse_students.source_id = student_dimensions.source_id;
    DELETE
    warehouse=> commit;

And cleanup:

    warehouse=> drop table apangea_students;
    DROP TABLE
    warehouse=> drop table warehouse_students;
    DROP TABLE

We can do this with NOT IN queries as well, but I've found the above
is way faster.

Note: if you delete a bunch of records you may want to do the
following after you're done:

    warehouse=> vacuum analyze student_dimensions;

## Query for the student - classroom - school - district - customer hierarchy

Doing this query in the data warehouse is a bit tricky because of customer_dimensions.id vs customer_dimensions.customer_source_id.

Running through some SQL against the DW, pulling the hierarchy for several students in the ```student_dimensions``` table:

```
from
student_dimensions
inner join classroom_dimensions on classroom_dimensions.id = any(student_dimensions.classroom_source_ids)
inner join school_dimensions on student_dimensions.school_source_id = school_dimensions.id
inner join district_dimensions on student_dimensions.district_source_id = district_dimensions.id
inner join customer_dimensions on student_dimensions.customer_source_id = customer_dimensions.id
```

Seems logical, but wait! If you are joining across dimensions, the id fields are internal just like other relational databases. You actually want the dimension's ```_source_id```. You can tell you've made this mistake if all of the results come back as Sallisaw Public Schools which is ```id``` 1025. Texas is ```customer_source_id``` 1025. The joins rewritten:

```
from
student_dimensions
inner join classroom_dimensions on classroom_dimensions.classroom_source_id = any(student_dimensions.classroom_source_ids)
inner join school_dimensions on student_dimensions.school_source_id = school_dimensions.school_source_id
inner join district_dimensions on student_dimensions.district_source_id = district_dimensions.district_source_id
inner join customer_dimensions on student_dimensions.customer_source_id = customer_dimensions.customer_source_id
```

## Recreating benchmark usage facts

If you are missing some StudentBenchmarkUsageAggregates, ClassroomBenchmarkUsageAggregates, or SchoolBenchmarkUsageAggregates you may need to run:
```ruby
rake aggregate:student_benchmark_usage
StudentBenchmarkUsageAggregate.aggregate_all!
```
