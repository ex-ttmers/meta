# Reporting Definition Realignment - Technical Notes

## Warehouse schema changes

### Create a new "Type 1" SCD from all "Type 2" SCDs

The current schema of the DW optimizes for reporting on the state of a student *at the time they performed work* rather than their current state.
This is primarly caused by the decision to model most of our dimensions as [Type 2 SCDs](http://en.wikipedia.org/wiki/Slowly_changing_dimension#Type_2).
As students change or move, we write additional rows to the student_dimensions table to represent the new state of that student and leave the original row 
untouched. For example:

**student_dimensions**

| id | source_id | name   | is_active | is_latest | effective_date |
|:--:|:---------:|:------:|:---------:|:---------:|:--------------:|
| 1  | 7         | Joel   | true      | false     | 08-01-2015     |
| 2  | 7         | Joel M | true      | false     | 08-09-2015     |
| 3  | 7         | Joel M | false     | true      | 08-25-2015     |

Here, a single student with apangea ID 7 has been modified three times. They were created on 8/1, had their name changed on 8/9, and were deactivated on
8/25. We have an audit log, but let's see how this makes some seemingly simply queries difficult when we include a fact table.

**lesson_completed_facts**

| student_id | lesson_id | post_quiz_score |
|:----------:|:---------:|:---------------:|
| 1          | 10        | 0.85            |
| 2          | 17        | 0.67            |
| 2          | 77        | 1.0             |

Here, our student has finished 3 lessons. One was done when the first version of the student dimension record was current, and the other two were completed
after the name change (because it is using student_id 2 instead of student_id 1).

Now, let's say we want to run simple query to see how many lessons have been completed by active students. Naively, we'd write the following query:

```SQL
SELECT
    count(*)
FROM
    lesson_completed_facts
    join student_dimensions on student_dimenions.id = lesson_completed_facts.student_id
WHERE
    student_dimensions.is_active = true
```

According to our reporting guidelines, we *want* this query to return nothing, since our only student is currently inactive. However, the answer we get is **3**!
Why? Because we're joining to versions of the student that were active at the time the lesson was completed instead of the most recent version of the student.

To compensate, the warehouse currently goes to great lenghts to compensate for this by doing something like this:

```SQL
SELECT
    count(*)
FROM
    lesson_completed_facts
    join student_dimensions on student_dimenions.id = lesson_completed_facts.student_id
    join student_dimensions latest_student on latest_student.source_id = student_dimensions.source_id and latest_student.is_latest
WHERE
    latest_student.is_active = true
```

In addition to being complex, this is also slow. Also, this isn't applied consistently in all queries because the developer has to remember to do it. We can improve this situtation by introducing another dimension table: a new Type 1 SCD that only contains the most recent version of each record. Then, if the facts were to contain **both** the historical id **and** the source id, most queries would work as intended without having to think about this is_latest complexity. Extending the example above:

If we were to rename **student_dimensions** above to **historical_student_dimensions** and created a new table that looked like:

**student_dimensions**

| id | name   | is_active |
|:--:|:------:|:---------:|
| 7  | Joel M | false     |


And also change the fact table to look like:

**lesson_completed_facts**

| historical_student_id | student_id | lesson_id | post_quiz_score |
|:---------------------:|:----------:|:---------:|:---------------:|
| 1                     | 7          | 10        | 0.85            |
| 2                     | 7          | 17        | 0.67            |
| 2                     | 7          | 77        | 1.0             |

Then our original query:

```SQL
SELECT
    count(*)
FROM
    lesson_completed_facts
    join student_dimensions on student_dimenions.id = lesson_completed_facts.student_id
WHERE
    student_dimensions.is_active = true
```

would correctly return 0 rows, but yet we still preserve the historical student context if it's needed.

Dimensions affected:
- customer_dimensions
- district_dimensions
- school_dimensions
- classroom_dimensions
- student_dimensions
- teacher_dimensions
- donateable_dimensions *
- item_dimensions *
- pathway_dimensions *
- standard_dimensions *

Changes to existing tables:
- Rename existing dimensions to be prefixed with historical_
- Continue to update historical_ tables as type 2 SCDs via ETL.

New Tables:
- Generate new tables that are duplicates of the historical_ tables except they don't have is_latest, effective_start, effective_end columns
- The primary key is the apangea source id, not the surrogate id
- Updated as a Type 1 SCD via ETL

Incremental ideas:
- Include is_latest=true on all non-historical tables to begin with so the queries don't know the difference.
- Have feature flag that swaps which version of the dimensions we join to: historical or regular

Fact changes:
- Rename existing dimension columns on all facts to have historical_ prefix
- Add columns columns to all facts that contain source ids for all these dimensions

Query changes:
- Never join to historical records ever.
- Remove all is_latest stuff, everwhere.
- Remove double joins to get latest student records for filtering

Asterisk tables:

I question the value of keeping history on the dimensions marked with asterisks above, since their history doesn't directly
contribute to any known requirement. If warranted, we could remove the Type 2 historical table for those entities entirely
and just directly update the Type 1 table via ETL.

## Capture move events in apangea

Apangea needs to explicitly capture the following move events:
- Students moving between classrooms (which might be in another school, district)
  - Update classroom_id, school_id, district_id on all affected facts  
- Classrooms moving across schools (which might be in another district)    
  - Update school_id, district_id on all affected facts  
- Schools moving across districts (which might be to no district at all)
  - Update district_id on all affected facts

A simple UI should be developed in apangea so that the move history of a student/classroom/school/district can be 
seen by the dev and support teams.
 
## Consuming apangea move events in the DW

The ETL process needs consume new apangea "entity moved" events, and update all related facts accordingly.

- Students moving between classrooms
  - Update classroom_id, school_id, district_id on all affected facts  
- Classrooms moving across schools    
  - Update school_id, district_id on all affected facts  
- Schools moving across districts
  - Update district_id on all affected facts

Tables affected:
- attempt_facts
- attempt_by_date_student_classoom_aggregates
- **NOT** donation_facts (classroom goal donations should not move)
- hint_view_facts
- lesson_completed_facts
- live_help_facts
- placement_attempt_facts
- placement_test_completed_facts

The aggregate tables are rebuilt every night, and so we should explain that moves will not be reflected in aggregate
until the next school day.

## Students With Work Fact

In order to support the guiding principle that students with work over the configured date range should be denominator for all averages,
we need an easy way to see if a student has completed a single attempt, placement attempt, or benchmark attempt in that date range.

We already have a daily rollup of the attempt_fact table that gets us close. If we extended it to include placement attempts and benchmark attempts
a simple check to see if a row exists in that table over the configured date range for the student in question would answer the question.

- Extend attempt_by_date_student_classoom_aggregates to include the number of placement attempts, benchmark attempts, and lessons completed each day.
- Rename attempt_by_date_student_classoom_aggregates to student_work_by_date (or something)
- We might be able to merge this concept with the PointLeaderboard aggregate table?
- Update overview report query to use this table for the students_with_work count and use it as the denominator in all averages in the overview report.
- Need to do more research to see where else we show averages to see what else will be affected.
