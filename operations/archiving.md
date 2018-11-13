Archiving data has two components:

1. At the end of the year, we get to wipe the database and start fresh.
2. During the year, we should work to archive student activity data so
   the database doesn't grow so quickly.

They're two separate sets of problems and concerns, so we'll deal with
them separately.

# End of year: Wiping the database

Unfortunately we don't have the luxury of __actually__ wiping the
database, because of:

* Summer school
* People wanting to do more reports or other data discovery
* Random sales needs

We decided to segment the archives by school. (As of 19-April there
are 13.3k schools in the system.)

We also need to support the ability to UN-archive a school, so any
solution we come up with needs to keep each school's archived data in
a separate place (file, directory, database, whatever).

## Per-school data to include

Every school's archive should include:

### Database: master

Whenever you see 'users where...' below, we'll also capture for them:

* `preferences`
* `user_opted_in_rollup_reports`

Group: __school__

* `schools`
* `taggings`
* `donations` (include records with relevant `school_id`)
* `school_csv_imports`, `school_file_imports` (needed?)
* `users where user.id in school_administrations.user_id` (by school)
* `users where user.id in school_assignments.user_id` (by school)

Group: __classroom__

      SELECT c.*
        FROM classrooms c
       WHERE c.school_id = ?
    ORDER BY c.id

* `classrooms`
* `donations` (include records with relevant `class_id`)
* `classroom_goals`
* `users where user.id in classroom_teachings.user_id`

Group: __student__

      SELECT s.*
        FROM students s
             JOIN classroom_students cs on s.id = cs.student_id
       WHERE cs.classroom_id IN (...)
    ORDER BY s.id

* `students`
* `student_sessions`
* `student_settings`
* `donations` (include records with relevant `student_id`)
* `parents_students`

### Database: shard

Group: __enrollment__

      SELECT e.*
        FROM enrollments e
       WHERE e.classroom_id IN (...)
    ORDER BY e.id

* `enrollments`
* `attempts`
* `lesson_enrollments`
* `hint_views`
* `placement_tests`
* `placement_attempts`
* `enrollment_activities`
* `enrollment_activity_attemptables`

We may want to have another group under `lesson_enrollment` for
`enrollment_activities` and `enrollment_activity_attemptables`
depending on performance.

## Other data

What do we do with non-school data that's not system data? For
example:

* District information
    * `districts`
    * `district_administrators`
    * `donations` (for district)
* Customer information
    * `customers`
    * `customer_administrators`
    * `contests_customers`
    * `customer_settings`
    * `donations` (for customer)
* `charities`
* `contests`

For now we'll likely just keep it around. It's not very much data and
we may actually want to keep it.

## Planning implementation

### Things to check

__Where do we put the files?__

* S3: is this okay for student privacy? Do we need to encrypt?

__Data integrity__

* Is a single user in multiple of `school_administrators`,
  `school_assignments`, or across schools in `classroom_teachings`?
* Are donation criteria exclusive? (That is, a donation with a
  `student_id` won't also have a `class_id`.)
- Are students only in one school?

### Questions

* How to handle schema migrations after the data have been dumped?
* What's the biggest school? (most classrooms + students + activity)
* How long to we expect it to take given n GB? (extract time + restore time)

### How it will work: archiving

__1.__ You run:

    $ ruby archive.rb --school=13 --env=ttm-staging

Other options you can run with:

`--customer=1234`: runs the steps below but for each school within the
customer, first checking the extraction table for each school.

`--env=ttm-staging`: can also be 'local' to run locally with hardcoded
master + shard names.

`--pretend`: do the extraction but don't delete any data or create an
extraction record.

__2.__ Fetch Heroku DB strings for `ttm-staging` environment.

__3.__ Dump data via COPY for school with ID 13 and tables in group school

__4.__ Dump data via COPY for each classroom in school and tables in
group classroom

__5.__ Dump data via COPY for each student in classroom and tables in
group classroom

__6.__ Dump student activity data via COPY for each classroom in group
enrollment (on the school's shard)

__7.__ Pack up the dumped files into a tarball named for the school
(e.g., `2012_school_13.tar.gz`).

__8.__ Generate MD5 checksum for tarball.

__9.__ Create an S3 path and send the tarball there.

__10.__ Run the __Restoring__ steps below using the S3 path + checksum
just created, and against a local database (no shards), recording long
it takes.

__11.__ Delete all the data from steps 3-5.

__12.__ Write an extraction record of the archive into master with the
customer name + ID, school name + ID, S3 path, MD5 checksum of tarball
(step 8), size of the tarball, time to extract (steps 3-6) and time to
restore locally (step 10).

### How it will work: restoring

__1.__ You run:

    $ ruby restore.rb --school=13 --heroku=ttm-staging

Other options you can run with:

`--customer=1234`: runs the steps below but for each school within the
customer, first checking the extraction table for each school.

`--env=ttm-staging`: can also be 'local' to run locally with hardcoded
master + shard names.

`--pretend`: do the fetch and verification but don't load any data or
delete the extraction record.

__2.__ Fetch Heroku DB strings for `ttm-staging` environment.

__3.__ Fetch extraction record in apangea for school ID 13.

__4.__ Fetch tarball from S3 using path in extraction record.

__5.__ Verify tarball with checksum and unpack.

__6.__ Load files to master database via COPY in a set order.

__7.__ Fetch school record just loaded and get shard name.

__8.__ Load files to specified shard database via COPY in a set order.

__9.__ Delete extraction record for this school.

__10.__ Leave file on S3? Replace next time it's extracted?

# During the year: Archiving activity

The idea: since we now have a data warehouse we no longer need to keep
student activity around after a period of time. The process might look
like this:

* At the beginning of every month create a timespan ending the month
  before last (e.g. if it's November find data from September 30 and
  earlier).
* Find all `hint_views` in that timespan and archive
* Find all completed `lesson_enrollments` ending in that timespan and archive
  them and their dependencies:
  * `attempts`
  * `enrollment_activities`
  * `enrollment_activity_attemptables`
* Find all completed `enrollments` ending in that timespan and archive
  them and their dependencies:
  * `placement_attempts`
  * `placement_tests`

## De-sharding!

With the above plan in place we may be able to de-shard! We should so
do while preserving the ability to re-shard if necessary.

# Notes

## Drop tables from production

These probably got there by accident:

On shards:

* `emailed_rollup_reports` + sequence
* `reports_in_emails` + sequence
* `rollup_reports` + sequence
* `user_opted_in_rollup_reports` + sequence

Also:

* `activity_join` + sequence
* `images_tmp`
* `lesson_enrollments_changed_passed`
* `queue_classic_jobs*`
* `tmp_activities_join`

#Meeting Notes

##7/16/13
* Date for archiving 8/17/13, per Services and Support, but subject to change.
* Must also accompany archiving process:
  * Deploy code for SY13-14 data warehouse reporting ("time spent doing math" calculation).
  * Revert auto-enroll pathways/UI.
  * Unlink data warehouse from main database, archive, set up new instance of database, re-link to data warehouse.
* Historically, we have left teachers active and blown away all students.
* There are a handful of customers that will want to keep using last year's data at the cutover point.
  * Tim to get list from Glen/Ryan.
* Chris Winters suggested keeping customers, districts, schools and teachers, but not classrooms and students.
 * There is uncertainty whether operations with customers will change if anything is kept.
   * Tim to vet with Support.
* Team discussed keeping student records and "promoting", but concerns were raised over students not belonging to a classroom and operations around finding and relating students to classrooms vs. starting fresh with new classrooms and student records as in years past.
* Chris Winters suggested archiving now customers who will not be renewing.
  * Tim to get list from Sales/Services/Support.
* Keith suggested using this opportunity to also purge legacy super users and admins.
* Chris Winters reminded the team that archiving is not an instantaneous process, driven by the volume of student activity we are archiving (attempts, enrollments) and the sizes of customers and districts.
* Next steps:
  * Tim to follow up on his action items.
  * Chris Winters to modify archiver to fully nuke vs. not fully nuke.
  * CG to research operations/tasks for data warehouse switchover.


#2013 Archiving Checklist

FYI - it is estimated to take 14.5 seconds per classroom to archive.  With 13771 classrooms it would take max 55 hours.  (2 days and 7 hours)  Bumping the sidekiq up to 4 would, in theory, have this take 15 hours(ish).

* Get the list of schools and/or customers to exclude from Ondo.  Use this to create sidekiq jobs to do the archiving.

* On (Friday Aug 16th, at 5PM)
  * Put the db in Maintenance mode.
  * Backup the database.
  * Turn off Data Ware house - rename it to 2012-2013 - turn etl_job dynos to 0
  * Run the sidekiq jobs to archive and delete the data.
  * Truncate tables feed_items, feed_item_interactions
  * Do vacuum full on each db

  * Rebuild the new warehouse for upcoming year.
  * Point apangea to the new warehouse (env variable)
  * Restart the app (the changing fo the env variable does this already?).
  * Spin down the old warehouse database on heroku to something smaller.
  * Turn off maintenance mode

#2014 Archiving Checklist

### Possible Archiving Dates
- 8/8
- 8/15
- 8/22
- 8/29

### Steps to Archive

1. Apangea Maintenance Mode __AUTOMATE__
 - `ttmscalr maintenance:on -f production`
2. Kill System Watcher __MANUAL__
3. Kill Data Warehouse ETL __AUTOMATE__
 - `ttmscalr shutdown:sidekiq -f warehouse-production`
4. Disable Auto-Backup __MANUAL__
 - Run 5 times (once on each server)
5. DB Backup __MANUAL__
 - Run 5 times (once on each server)
6. Transfer DB Backup __MANUAL__
 - Run 5 times (once on each server)
7. Terminate Slaves __MANUAL__
 - Run 5 times (once on each server)
8. Run Archiving Script __AUTOMATE__
 - Must be run on remote server (DEBUG ROLL)
9. Vacuum Full __AUTOMATE__
 - Run 5 times (once on each server)
 - Put it in Scalr and have it do it
10. Binary Snapshot __MANUAL__
 - Run 5 times (once on each server)
 - Click button in Scalr
11. Activate Slaves __MANUAL__
 - Run 5 times (once on each server)
12. Data warehouse Cut Over __MANUAL__
 - MAKE SURE ETL IS NOT ENABLED ON MULTIPLE DATA WAREHOUSES
 1. New Data warehouse ***Done Before This Process***
 2. Update warehouse.ttm.com to point at new farm
 3. Warehouse-2013.ttm.com to point to old farm
 4. Reduce instance size of old farm
13. Enable Auto-Backup
14. Enable System Watcher
15. Apangea Maintenance Mode Off
 - `ttmscalr maintenenace:off -f production`
16. Data Warehouse rebuild

### Action Items

1. Determine if archiving utility can be leveraged
2. Jim/Rondo tolerance for data lost from day of archiving if students do work in schools that are going to be archived the day of archiving. It will be faster and simpler to use the previous night's database backup rather than having to take a fresh one each time we run the archive/delete script.
3. Ensure it's okay that straggler schools will not have work older than 8/8 in last year's warehouse.
4. Can we just rebuild data warehouse after 8/29? Rollup reports will last years data until 8/29.
