# How to add new data to be ETLed into the warehouse

* Think about your schema.
  * What kind of data do you have?
    * Is this a table of events that you will only be inserting immutable records into? For example, AttemptFacts-- attempts don't ever get edited after they happen. These should be in tables that end in `_facts` and have local date/time and UTC date/time columns.
    * Is this a table of entities that can be updated over time? For example, ClassroomStudents-- students can move between different classrooms, so there are some classrooms that they have spent time in in the past but are currently in a different set of classrooms. Or Users' names could be updated. These should have 2 tables-- one for the current state, such as `classroom_students` or `users`, and one for the states in the past-- `historical_classroom_students` or `historical_users`. These are dimension tables but seejee got tired of typing `_dimension` all the time so most of these are suffixless.
    * Is this basically an enum of a fixed number of values that will rarely change? For example, AttemptStatuses. These are also dimensions but don't need to be incrementally updated so their ETL will be different.
    * Is this a table of entities that change over time but that we DON'T care about past values for? WE DO NOT CURRENTLY HAVE ANY OF THESE.
  * What would make the sql queries easy for the people who need data out of this table?
  * What indexes are needed to make those queries performant? Don't just index everything, indexes take up space!
  * Put limits on small columns. We store a lot of data in the warehouse and it adds up! `limit` is the number of characters for :string and :text columns and number of bytes for :binary and :integer columns. 
    * 1 byte integers can store numbers up to 255, so this is good for columns like attempt_status_id because attempt_status is a dimension made up of a fixed set of 3 enumerated statuses and this won't grow without us making a conscious change.
    * 2 byte integers can store numbers up to 65535.
    * Date and time IDs should have `limit: 2`. 
* Create a migration to add the new table(s) IN THE WAREHOUSE SUBDIRECTORY of db/migrate and run rake db:migrate.

## ETL for each fact table

Assuming your new fact table is called something like `some_new_facts`, replace `[table_name]` below with `some_new` and `ConstantName` with `SomeNew`.

*  Create `etl/[table_name]_fact.ctl` patterened after recent `*_fact.ctl` files and containing SQL that will be run against apangea to extract the data you want this table to have.
*  Add `run '[table_name]_fact.ctl'` to `etl/incremental_etl.ebf`
*  Create `etl/initial_load/create_[table_name]_indexes.sql` with `CREATE INDEX` statements for all the indexes on this table.
*  Add `[ConstantName]Fact:       { data: '[table_name]_fact.txt', script: '[table_name]_fact.ctl' },` to `etl/initial_load/data_load_client.rb`
* Create `etl/initial_load/drop_[table_name]_indexes.sql` with `DROP INDEX` statements for all the indexes on this table.
* Create `etl/initial_load/extract_[table_name].sql` patterened after recent `extract_*.sql` files, containing mostly the same query you put in `etl/[table_name]_fact.ctl`.
* Add `'[table_name]' => [ConstantName]Reloader.new,` and `jobs << reload_table('[table_name]', false)` to `etl/initial_load/load_jobs.rb`
* To `etl/initial_load/reloaders.rb`, add:

```
class [ConstantName]Reloader < Reloader
  def name
    '[table_name]'
  end

  def fact
    :[ConstantName]Fact
  end

  def partial(options)
    # This SQL gets added to etl/initial_load/extract_[table_name].sql and is how it decides what
    # rows need to be removed when we're reloading a particular school. Here's one example; take
    # a look at the other classes in this file to decide what should go here.
    "
        s.id IN (SELECT students.id from students #{school_deletion_where(options)})
        AND bt.id NOT IN (#{unprocessed_event_ids.join(',')})
    "
  end
end
```
* Add `[ConstantName]Fact` to the list in `lib/deleters.rb`
* Add `[ConstantName]: 'etl/[table_name]_fact.ctl',` to `lib/process_etl_job_worker.rb`

## ETL for each dimension table

### Dimensions that change

* I haven't had to make one of these yet; if you make one, fill in this section in more detail.
* Start from a dimension like students and its partner historical_students and follow the patterns.

### Dimensions that have a fixed set of values

Assuming your new dimension table is called something like `some_thing_types`, replace `[table_name]` below with `some_thing_type` and `ConstantName` with `SomeThingType`.

* Create `[table_name]_dimension.ctl` and pattern it after `attempt_status_dimension.ctl` (as of this writing the `_dimension` suffix is still part of the filenames but this may be removed in the future, follow what pattern is there when you're reading this) to get the values into the database.
* Add `run "[table_name].ctl"` to `etl/dimensions.ebf`

## Testing ETL

### Manually

In order to test your `ctl` file, you can:

* Remove data from the new DW table(s) you're populating
* Delete the row for this table in the `cross_references` table, which keeps track of the last ID the warehouse has seen-- if this row doesn't exist, ETL will get everything.
* Put relevant data in your local apangea
* Run `etl etl/[table_name]_fact.ctl` or `etl etl/[table_name]_dimension.ctl` or whatever your `.ctl` file is called.
* Check that the output says the number of lines read from sources is what you would expect
* Check that the data ended up in your warehouse in the way you'd expect

You can also make sure these work as you would expect:

* `rake etl:incremental`
* `etl/reload_reporting`

### Automatedly

* Add a new test in `spec/etl/[table_name]_fact_spec.rb` patterned after the others.
* If necessary, create data to be tested in the `create_some_facts` method in `spec/etl/scripts/load_helper.rb`
* In `spec/etl/etl_integration_spec.rb`, add an `it` to each `describe` that checks the counts of your new data expected.
* Run `./spec/etl/etl_integration_spec.sh` and get the tests to pass.

## Deploying

If you changed `etl/dimensions.ebf`, you'll need to run that once manually after deploying -- it doesn't get automatically run on every incremental ETL cycle.

Check how many rows are in apangea that will be brought over by your new ETL.

If there's fewer than a few hundred thousand, cool, deploy normally and the normal ETL process will start working on your new rows.

If there's more than that, during off-peak hours:

* turn off etl
* deploy reporting
* run migrations
* for JUST your new facts/dimensions, do a full rebuild using the load_job.rb script
* turn etl back on
