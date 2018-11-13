
##### Existing queries, categorized:

Report Queries
  - AchievementReportQuery
  - ClassroomOverviewReportQuery (not initialized with a level but takes a level param) (used?)
  - LeaderboardQuery (has a pager, does not use query builder)
      - LessonLeaderQuery
      - PointLeaderQuery
  - OverviewReportQuery - uses query builder, lots of subqueries
  - StandardsReportQuery
  - UsageReportQuery
  - LiveTeachingQuery
  - MathTimeQuery - uses QueryBuilder
  - OverallPerformanceQuery - sets group to ByStudent, gets a level, no querybuilder
  - StartedStudentCountQuery
  - Think30ScoreboardQuery - Data used for both widget + spreadsheet
  - TrendingGradeLevelQuery

Driver Queries
  - GradeLevelAndPerformanceDriverQuery (CommonFilterSet)
  - GradeLevelDriverQuery (CommonFilterSet)
  - SchoolAndGradeLevelDriverQuery

Helper Queries
  - Subqueries / CTEs
    - AttemptByDateStudentClassroomAggregateQuery (group, agg_finder for AttemptByDateStudentClassroomAggregate, CommonFilterSet)
    - AttemptQuery (group, CommonFilterSet)
    - AverageStudentScoresForStandardsQuery (LessonsForStudentQuery - filtering done at that level, group)
    - DonationQuery (CommonFilterSet, Donateable Filter)
    - EnrolledStudentsQuery (initialized with col_name, params, only_active, ignore_start_date; uses CommonFilterSet)
    - GradeLevelQuery(CommonFilterSet)
    - LessonCompletedQuery (CommonFilterSet, agg finder for LessonCompletedFact)
    - LessonsForStudentQuery (bunch of CTEs)
    - PlacementTestQuery
    - ProjectedLessonsQuery
    - RolledUpGradeLevelQuery
    - StandardsReportStudentDetailQuery
    - StudentQuery
    - UniqueCompletedLessonsForStudentQuery
    - UniqueLessonQuizScoreQuery
    - UniqueStudentCountQuery
  - Filter Queries
    - ClassroomQuery - a bunch of queries, builds a classroom filter for each
    - MathLessonStandardsQuery
    - StudentsForTeacherQuery
    - StudentsInHierarchyQuery

  - Wrapper Queries
    - StandardsReportSummaryQuery


Finders - have a join_to
  - ClassroomFinder
  - DistrictFinder

Unsorted
  - DimensionSearch - generic dimension query, used?
  - Monkeys
  - SimpleAverageQuery
  - SimpleCountAndAvgQuery
  - SimpleCountAndSumQuery
  - SimpleCountQuery
  - SleepyReportQuery

Extras
  - CommonTableExpression
  - Groups
  - Pager
  - QueryBuilder
  - SortableQuery
