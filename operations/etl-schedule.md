### Incremental ETL: (every three minutes)
- `rake etl:queue_job` puts a `ProcessEtlJobWorker` on the etl queue.
- `ProcessEtlJobWorker` updates all 'base' facts and dimensions.
- An `EtlAggregationWorker` job is put on the aggregate queue.
- Some of the 'aggregate' facts are rebuilt incrementally:
  - `AttemptByDateStudentClassroomAggregate`: two days of data.
  - `LessonCompletedFact`:  current day's data 
  - `LessonLeaderboardAggregate`: current day's data 
  - `PointLeaderboardAggregate`: current days' data
  - `LiveHelpFact`: current day's data
- Arrays are projected to tables.
- `LessonProjectionAndStandardsWorker` jobs are put on the projection queue.

### Nightly ETL
- 1AM UTC / 9PM EDT - Update moved enrollments
  - `rake etl:process_enrollment_fixes` run synchronously on the SystemWatcher role. 
  - Triggered by `FixEnrollmentClassroom` events.
  - Facts have their classroom id updated according to enrollments that were moved to different classrooms in apangea.
- 3AM UTC / 11PM EDT - VACUUM Analyze
  - `rake db:vacuum_analyze` run synchronously on the SystemWatcher role.
- 5AM UTC / 1AM EDT - Rebuild all aggregates.
  - `rake etl:aggregate` run synchronously on the SystemWatcher role.
- 9AM UTC / 5AM EDT - Enqueue an unresolved pathway job to the etl queue
- 10AM UTC / 6AM EDT - Enqueue an unresolved job to the etl queue.

### Weekly ETL
- Sunday at 1PM UTC / 9AM EDT - Rebuild weekly aggregates.
