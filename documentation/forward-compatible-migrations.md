# Forward Compatible Migrations

Our [archiving strategy](Archiving-data) is fairly robust with
regards to database schema changes. It stores with every archive the
fields in each table that were present at the time of the data
extraction and uses those to restore data.

But this technique is not perfect, and there are a handful of change
types it cannot accommodate. We'll talk about those here and
strategies to mitigate them.

## Quick overview: how it works now

The archiver has a list of tables and constraints for pulling data
from them. For example we have the following declaration for
donations by school:

    class SchoolDonations < QuerySpecs
      def from; 'donations'; end
      def where; 'school_id = %s'; end
    end


With the parent class that's sufficient information to be able to
extract and delete data from the 'donations' table for schools. We do
so with a query like this, though this one is formatted better:

    COPY (
      SELECT
        id, donateable_id, donateable_type, student_id, class_id,
        school_id, district_id, customer_id, points, dollars,
        created_at, updated_at
      FROM
        donations
      WHERE
        school_id = 2019
    ) TO STDOUT


The archiver then captures the results from the query's `STDOUT` and
streams them to a file. It also captures the columns used in the query
and serializes them in the `metadata.json` file stored with the
archive, which looks something like this:

    {
      "school_id": "2019",
      "school_name": "Norwalk-La Mirada CDS (Visions)",
      "school_shard": "enrollments_shard_four",
      "schema_version": "20130604190847",
      "customer_id": "142",
      "db" : {
        ...
        "donations": ["id", "donateable_id", "donateable_type", "student_id", "class_id",
                      "school_id", "district_id", "customer_id", "points", "dollars"",
                      "created_at", "updated_at"],
        ...
      }
    }

When restoring using `COPY` you typically just do:

    COPY donations FROM 'donations.txt'

But as part of making the restore process more robust we explicitly
list the columns from that `metadata.json` file, making a query like
this:

    COPY donations(
           id, donateable_id, donateable_type, student_id, class_id,
           school_id, district_id, customer_id, points, dollars,
           created_at, updated_at
         )
    FROM STDIN

Creating adaptable migrations is in large part understanding this process.

## Identifying tables used in archive

For a number of the migration types below you need to check to see if
you're operating against it's one of the tables used in
archiving. Just do this:

    cwinters@abita:~/Projects/TTM/apangea/db/archiving$ ruby school_restorer.rb tables
    Restore tables:
    ===============
      attempts
      charities_customers
      classroom_goals
      classroom_students
      classroom_teachings
      classrooms
      ...
      user_opted_in_rollup_reports
      users
    -- COMPLETE --

Note that this listing merges tables on the master shard and those on
the enrollments shards -- we can separate them if you think it's
useful.

## Migration: creating a table

On the surface, this is no problem! As long as the data being restored
doesn't need to be moved into the new table you're good.

If we do find the need to do such a migration we'll have to build that
into the archiver. Since it's not something we've needed so far it's
not there yet, but if we can express the migration in SQL it would be
pretty straightforward to have a set of SQL scripts that need to be
run given a range of source and target schema versions.

## Migration: dropping a table

If you need to drop a table and it's not among those used in an
archive, no problem!

If your table is used, __do not drop it!__ Removing the ActiveRecord
class should be sufficient for the application's purposes, but if you
remove the table entirely the restore will fail.

Also, see notes under 'creating a table' above regarding migrating
data into new table/s, if that's necessary.

## Migration: adding a column

Adding a column to one of the tables we use should be fine, with one
caveat: if the column does not allow null (has a `NOT NULL`
constraint) then the import will fail with a constraint
violation. Here's an example of what's being done behind the scenes:

    apangea_development=# create table null_on_insert ( id serial, name varchar(25) );
    NOTICE:  CREATE TABLE will create implicit sequence "null_on_insert_id_seq" for serial column "null_on_insert.id"
    CREATE TABLE
    Time: 8.130 ms
    apangea_development=# insert into null_on_insert (id, name) values (2, 'Mulan');
    INSERT 0 1
    Time: 11.004 ms
    apangea_development=# select * from null_on_insert
    apangea_development-# ;
     id | name
    ----+-------
      2 | Mulan
    (1 row)

    Time: 0.812 ms
    apangea_development=# alter table null_on_insert add column age int;
    ALTER TABLE
    Time: 1.582 ms
    apangea_development=# update null_on_insert set age = 23;
    UPDATE 1
    Time: 1.444 ms
    apangea_development=# alter table null_on_insert alter column age set not null;
    ALTER TABLE
    Time: 1.149 ms
    apangea_development=# insert into null_on_insert (id, name) values (3, 'Merida');
    ERROR:  null value in column "age" violates not-null constraint
    DETAIL:  Failing row contains (3, Merida, null).
    Time: 1.489 ms

From a strict data integrity perspective not-null constraints are
hugely useful. But this is a sacrifice you have to make when your
database has to act as a repository across software versions. It will
likely require additional validations on the application side to
accommodate.

A useful strategy for this is to add a DEFAULT value, so when we do a
COPY into the table without a value for that column it'll get a
reasonable value.

## Migration: removing a column

__Do not__ remove a column from one of the tables used in an
archive. If we no longer use the column in code then there's no harm
in keeping it. But if we need to explicitly ignore the column in AR
there are a couple of strategies out there, two of which are listed in
[this github issue](https://github.com/rails/rails/pull/7417):

1. Override the class method `columns` and elide the column.
2. Use the [ignorable gem](https://github.com/nthj/ignorable)

## Migration: renaming a column

__Do not__ rename a column from one of the tables we use. This may
mean there's a semantic dissonance between the name of the field and
its use, but that's the cost of maintaining compatibility.

If the dissonance is too great then you might consider keeping the
original field and creating a new field. It might be a useful strategy
to default the new column to the value of the old column, but you'll
have to create a `BEFORE INSERT` trigger since you cannot use column
expressions as a `DEFAULT` value.

## Workarounds

There are some workarounds for when we don't follow these
strategies. They're generally only appropriate for restoring locally,
or to some other sort of testing system.

### Restore to an earlier version, then migrate

The primary workaround is to follow this process:

1. Checkout code at or around the time the archive was created.
2. Run `db/rebuild_with_production_content.sh`
3. Restore schools/customers as needed
4. Switch to a current branch
5. Run `rake db:migrate` to run through the migrations as normal.
