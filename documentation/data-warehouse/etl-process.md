## ETL Process

The ETL process defines the movement of data from the production
system to the data warehouse. ETL stands for Extract Transform and
Load, and each step of the process should be considered
separately because they make different tradeoffs -- for example,
Extract and Load are concerned with speed above all else, but
Transform is more interested in data integrity and consistency.

### Quick start

They can be queued into sidekiq from rake:

    $ rake etl:queue_job

__HINT__ -- execute that rake task every 10 minutes from cron and
(as long as you've got a single sidekiq processor, see below)
you've got a serialized scheduled job execution engine.

Or they can be programmatically added from the Rails console:

    cwinters@abita:~/Projects/TTM/reporting$ heroku run console -a ttm-reporting
    Running `console` attached to terminal... up, run.3003
    ...
    Loading production environment (Rails 3.2.11)
    irb(main):001:0> ProcessEtlJobWorker.perform_async
    => "3a126e91ac464c089f71d759"

This last option also enables you to process specific ETL
files. For example, say I wanted to queue a job that only ran the
ETL process for attempt facts:

    irb(main):001:0> options = { 'files' => [ 'etl/attempt_fact.ctl' ] }
    => {"files"=>["etl/attempt_fact.ctl"]}
    irb(main):002:0> ProcessEtlJobWorker.perform_async(options)
    => "3a126e91ac464c089f71d759"

Or say I wanted to only execute attempt facts but redefine the
batch size to 5000 per shard:

    irb(main):001:0> options = {'files' => ['etl/attempt_fact.ctl'], 'env' => {'ATTEMPT_BATCH_SIZE' => 5000}}
    => {"files"=>["etl/attempt_fact.ctl"], "env"=>{"ATTEMPT_BATCH_SIZE"=>5000}}
    irb(main):002:0> ProcessEtlJobWorker.perform_async(options)
    => "3a126e91ac464c089f71d759"

Creating lots of such jobs now becomes really simple. Here we put
60 jobs onto the queue:

    irb(main):001:0> options = { 'files' => [ 'etl/attempt_fact.ctl' ] }
    => {"files"=>["etl/attempt_fact.ctl"]}
    irb(main):002:0> 60.times { ProcessEtlJobWorker.perform_async(options) }
    => 60

### Setup

The sidekiq jobs are put into a 'etl_job' queue. There's a worker
in `Procfile` setup to process this queue:

    etl_job: bundle exec sidekiq -c 1 -q etl_job

The `-c 1` configuration limits sidekiq to only process one job
at a time. __THIS IS IMPORTANT__. If multiple ETL jobs are
running at the same time you run the risk of double-importing
data. __Don't do that.__

## Issues

### ETL: resolving inconsistency eventually

One tricky issue that crops up with data warehousing is syncing
dependent data. For example, what happens if we try to insert an
attempt fact that references a classroom that's not yet in the
system?

In the beginning we'd insert a -1 for the classroom and punt to
resolving it later, hoping it would happen only rarely. (We also
had the idea of inserting a negated classroom ID and retrieving
them later.)

What we settled on was this process:

1. if any dimension is unresolved for a fact we insert the
   original record into a table, along with the fact type, time
   inserted, number of times tried (?), and a string indicating
   which dimensions are unresolved (?)
2. every n minutes (30? 60?) we run a scheduled job THAT MUST BE
   PROCESSED SERIALLY WITH `etl_job` JOBS; that scheduled job will:
    1. collect unresolved records created before the scheduled
       job time and group them by fact type
    2. for each fact type we create a file containing the records
       (including the number of tries) with a known filename; we
       may also create a custom `:source` to read these directly
       from the database
    3. run the ETL control job for each fact type (similar to how
       we can run specific controls in ProcessEtlJobWorker); that
       ETL job looks for the known filename (or uses the custom
       `:source`) to recognize it's processing previously
       unresolved records
    4. at the end of each ETL job we delete ALL the pending
       records created before the ETL job time; if the records
       can still not be processed we just re-add them to the
       holding table and they'll get reprocessed at the next
       scheduled job time

### ETL: fact types

There are two types of facts from the point-of-view of the ETL
process.

1. Immutable facts. These are written once and never modified.
(It's possible the facts are deleted, but we haven't dealt with
that yet.)
2. Sessioned facts. These represent data over a span of time.

#### Immutable facts

The ETL process for tracking immutable facts is easy: since they
never change we can just track the max ID. Records before the ID
have been consumed by the ETL process, records after the ID
haven't and can be __pulled__ to the warehouse. No changes are
necessary to the server.

Two examples of these are `attempts` and `hint_views`. Both of
these are sharded, so we actually track a max ID per shard rather
than overall. In the ETL database's `cross_references` table
you'll see these rows for the `attempt_facts` tracking:

    ttm_etl_development=# select id, table_name, last_id from cross_references
    ttm_etl_development-#  where table_name like 'attempt%';
     id |     table_name      | last_id
    ----+---------------------+----------
      7 | attempt_facts.one   | 78637502
      8 | attempt_facts.two   | 78637950
      9 | attempt_facts.three | 78637582
     10 | attempt_facts.four  | 78637945
    (4 rows)

So the next time it incrementally fetches records it will get
those with ID > 78637502 from shard one, ID > 78637950 from shard
two, and so on.

#### Sessioned facts

The process for tracking sessioned facts is a little more
complex. The server needs to decide when it's okay for the facts
to be __pushed__ to the warehouse. These decisions must be coded
in the lifecycle for each fact. Hopefully it's obvious when that
occurs, but some examples follow.

They all create a record in `data_warehouse_events` via a sidekiq
job to minimize contention.

__Lesson Completed__

Completing a lesson is pretty straightforward. We only mark a
lesson completed in two places:

1. When a pre-quiz is passed and the student can place out
2. When a post-quiz is completed.

__Live Help__

We mark a live help session as completed when:

1. The teacher closes the session. This is an explicit event
   triggered by the user.
2. The session is 'stale', which means it hasn't been closed and
   it was started over two hours ago.

The first is easy to catch -- there's a method
`app/models/live_help_transcript.rb#complete!` to which we add
our behavior.

The second requires a scheduled job to look back over data. That
job is `dw:scan_live_help_sessions` and is run every hour on
production.

__Student Session__

Student sessions were recently refactored to have an explicit
lifecycle, so it was easy to add our behavior to
`app/models/student_session.rb#complete!`.

### ETL: sharded vs unsharded

For facts derived from non-sharded data the process is fairly
straightforward. For example, pulling data from
`live_help_transcripts` we need to:

* Feed data from a SQL query into the ETL pipeline

For sharded data it's more complex. We created the helper
`processors/shard_query_consolidator.rb` to make it easier. It
will take a base SQL query and, for each shard:

* Allow you to modify that query with shard-specific
  information such as its max ID.
* Accumulate the results into a file for all shards
* Feed that file's data into the ETL pipeline.

There's also a variation that will fetch relevant records from
the `data_warehouse_events` table on master and in turn fetch
corresponding records from their associated shard.
