These are a bunch of SQL queries you may find useful in practice
as well as to see the data model being used in context.

Also, see Heroku's [https://dataclips.heroku.com/](Dataclips)
feature, which allows us to store queries and rerun them
occasionally to look at updates. In particular the database meta
queries are useful to revisit to see, for example, which indexes
we might safely remove, or which statements have the highest
average execution time.

## Database meta

### Statement performance

      SELECT TO_CHAR(total_time / 1000 / 60, '9990.99') as minutes,
             TO_CHAR(total_time/calls, '990.999')       as ms_per_call,
             calls,
             rows,
             CASE rows
               WHEN 0 THEN 'n/a'
               ELSE to_char(rows/calls, '9990.9') END   as rows_per_call,
             query
        FROM pg_stat_statements
    ORDER BY total_time/calls desc;

### Table reads and cache hit ratio (overall)

    SELECT sum(heap_blks_read) as heap_read,
           sum(heap_blks_hit)  as heap_hit,
           (sum(heap_blks_hit) - sum(heap_blks_read)) / sum(heap_blks_hit) as ratio
      FROM pg_statio_user_tables;

### Which indexes used?

Filter out 'anchor_*' tables and primary keys, and order by the
ascending count of index scans so we can pick out those that
aren't being used.

        SELECT relname,
               indexrelname,
               idx_scan,
               pg_size_pretty(pg_relation_size(i.indexrelid)) AS index_size
          FROM pg_stat_user_indexes i
               JOIN pg_index USING (indexrelid)
         WHERE indisprimary IS false
               and relname not like 'anchor%'
      ORDER BY idx_scan, relname, indexrelname;

### Per table percent index use

Filter out tables that aren't queried

      SELECT relname,
             100 * idx_scan / (seq_scan + idx_scan) as percent_of_times_index_used,
             n_live_tup                             as rows_in_table
        FROM pg_stat_user_tables
       WHERE seq_scan + idx_scan > 0
    ORDER BY n_live_tup DESC;

### A view of index hit rates:

       SELECT 'index hit rate' as name,
              (sum(idx_blks_hit) - sum(idx_blks_read)) / sum(idx_blks_hit) as ratio
         FROM pg_statio_user_indexes
    UNION ALL
       SELECT 'cache hit rate' as name,
              case sum(idx_blks_hit)
                when 0 then 'NaN'::numeric
                else to_char((sum(idx_blks_hit) - sum(idx_blks_read)) / sum(idx_blks_hit), '99.99')::numeric
              end as ratio
         FROM pg_statio_user_indexes;

### Index scans vs sequence scans

This filters out tables with fewer than 500 rows because they
really don't matter -- pg will often choose to do a sequence scan
on them even if an index applies.

      SELECT statall.relname,
             statall.idx_scan,
             statall.seq_scan,
             (statall.idx_scan-statall.seq_scan) as difference,
             statuser.n_live_tup                 as rows_in_table
        FROM pg_stat_all_tables statall
             JOIN pg_stat_user_tables statuser
                  ON statuser.relname = statall.relname
                     AND statuser.schemaname = statall.schemaname
       WHERE statall.relname NOT LIKE 'pg%'
             AND statuser.n_live_tup > 500
    ORDER BY statall.seq_scan DESC
       LIMIT 50;

### Columns in a table

    SELECT
      attname
    FROM
      pg_attribute
      JOIN pg_class ON pg_attribute.attrelid = pg_class.oid
    WHERE
      relname = 'pathways'
      AND attnum > 0
      AND attisdropped = false

### Database sizes

Databases that the current user does not have access to will be infinitely large.

    SELECT
      d.datname AS Name,  pg_catalog.pg_get_userbyid(d.datdba) AS Owner,
      CASE WHEN pg_catalog.has_database_privilege(d.datname, 'CONNECT')
           THEN pg_catalog.pg_size_pretty(pg_catalog.pg_database_size(d.datname))
           ELSE 'No Access'
      END AS Size
    FROM
      pg_catalog.pg_database d
    ORDER BY
      CASE WHEN pg_catalog.has_database_privilege(d.datname, 'CONNECT')
           THEN pg_catalog.pg_database_size(d.datname)
           ELSE NULL
      END DESC -- nulls first
    LIMIT 20

### Relation sizes

    SELECT
      nspname || '.' || relname AS "relation",
      pg_size_pretty(pg_relation_size(C.oid)) AS "size"
    FROM
      pg_class C
      LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
    WHERE
      nspname NOT IN ('pg_catalog', 'information_schema')
    ORDER BY
      pg_relation_size(C.oid) DESC
    LIMIT
      20;


just for pg_toast relations:

    SELECT
      nspname || '.' || C.relname AS "relation",
      toasted.relname as toasted_table,
      pg_size_pretty(pg_relation_size(C.oid)) AS "size"
    FROM
      pg_class C
      LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
      JOIN pg_class toasted ON toasted.reltoastrelid = C.oid
    WHERE
      nspname NOT IN ('pg_catalog', 'information_schema')
    ORDER BY
      pg_relation_size(C.oid) DESC
    LIMIT
      20;

Find toast relations:

    SELECT
      T.relname
    FROM
      pg_class T, pg_class R
    WHERE
      R.relname LIKE '%report%'
      AND R.reltoastrelid = T.oid;

## General

### Standards info by state standard

Suitable for dumping to a spreadsheet; just remove the WHERE clause to
dump for all states.

       SELECT
         states.code                                              as state,
         state_standards.name                                     as state_standard_name,
         regexp_replace(standard_categories.name, '\t', ' ', 'g') as category_name,
         standards.id                                             as standard_id,
         standards.code                                           as standard_code,
         regexp_replace(regexp_replace(substring(standards.details from 1 for 120), '[\n\r]', '', 'g'), '\|', '/', 'g') as standard_details
       FROM
         state_standards
         JOIN states
              ON states.id = state_standards.state_id
         JOIN standard_categories
              ON standard_categories.state_standard_id = state_standards.id
         JOIN standards
              ON standards.standard_category_id = standard_categories.id
      WHERE
        states.code IN ('TX', 'OK', 'WV', 'PA', 'ID', 'TN')
      ORDER BY
        state, standard_categories.position, standards.position;

Note that the above assumes you're using a field separator of '|', as
the query replaces that in the standards details with '/'. To do this
in psql:

    apangea=> \a
    Output format is unaligned.
    apangea=> \f '|'
    Field separator is "|".
    apangea=> \o my_output_file.csv
    (execute your query, will be dumped to the named CSV)


### Standards covered in a state's default pathway

Given a state ID and grade level ID:

    WITH default_pathway_lessons(lesson_id, state_id) AS (
      SELECT
        pathway_lessons.lesson_id, pathways.state_id
      FROM
        pathways
        JOIN pathway_lessons ON pathway_lessons.pathway_id = pathways.id
        JOIN grade_levels ON grade_levels.id = pathways.grade_level_id
        JOIN lessons ON lessons.id = pathway_lessons.lesson_id
      WHERE
        pathways.state_id = 1
        AND pathways.grade_level_id = 3
        AND pathways.published = true
        AND pathways.customer_id IS NULL -- created by TTM
        AND lessons.active = true
    )
    SELECT
      lessons.id                  as l_id,
      lessons.name                as lesson_name,
      standard_categories.id      as cat_id,
      standard_categories.name    as category_name,
      standards.id                as std_id,
      standards.code              as standard_code,
      substring(standards.details from 1 for 60) as details
    FROM
      standards
      JOIN standard_categories
           ON standards.standard_category_id = standard_categories.id
      JOIN state_standards
           ON state_standards.id = standard_categories.state_standard_id
      JOIN lessons_standards
           ON standards.id = lessons_standards.standard_id
      JOIN lessons
           ON lessons.id = lessons_standards.lesson_id
      JOIN default_pathway_lessons
           ON default_pathway_lessons.lesson_id = lessons.id
              AND default_pathway_lessons.state_id = state_standards.state_id
    GROUP BY
      lessons.id,
      lessons.name,
      standard_categories.id,
      standard_categories.name,
      standards.id,
      standards.code,
      standards.details
    ORDER BY
      lessons.id,
      standard_categories.name,
      standards.code

### Permute distinct values of two columns

    WITH
     r(reason) AS (
       SELECT distinct(reason)
         FROM live_help_transcripts
     ),
     s(session_type) AS (
       SELECT distinct(session_type)
         FROM live_help_transcripts
     )
     SELECT r.reason, s.session_type
       FROM reasons, sessions;


### Big Attemptable Query (OLD, pre-lazy enrollment)

    SELECT lesson_enrollment_id,
           activity_id,
           attemptable_id,
           attemptable_type
      FROM (
         SELECT lesson_enrollments.id as lesson_enrollment_id,
                activities.id as activity_id,
                item_steps.id AS attemptable_id,
                'ItemStep' AS attemptable_type,
                lesson_enrollments.position as le_position,
                activities.position as a_position,
                activity_items.position AS activity_item_position,
                item_step_groups.position as item_step_groups_position,
                item_steps.position as attemptable_position
           FROM lesson_enrollments
                  JOIN activities
                       ON lesson_enrollments.lesson_id = activities.lesson_id
                  LEFT JOIN activity_exclusions
                       ON lesson_enrollments.id = activity_exclusions.lesson_enrollment_id
                          AND activities.id = activity_exclusions.activity_id
                  JOIN items
                       ON items.state = 'Published'
                          AND items.active is true
                  JOIN activity_items
                       ON activity_items.item_id = items.id
                          AND activity_items.activity_id = activities.id
                          AND activities.state = 'Published'
                  JOIN item_steps
                       ON item_steps.item_id = activity_items.item_id
                  LEFT JOIN attempts
                       ON lesson_enrollments.id = attempts.lesson_enrollment_id
                          AND attempts.activity_id = activities.id
                          AND attempts.attemptable_type = 'ItemStep'
                          AND attempts.attemptable_id = item_steps.id
                          AND attempts.final is true
                 LEFT JOIN item_step_groups
                         ON item_steps.item_step_group_id = item_step_groups.id
          WHERE lesson_enrollments.enrollment_id = $1
                AND activity_exclusions.id IS NULL
                AND attempts.id IS NULL
                AND lesson_enrollments.passed IS NULL
          UNION ALL
          SELECT lesson_enrollments.id as lesson_enrollment_id,
                 activities.id as activity_id,
                 chapters.id AS attemptable_id,
                 'Chapter' AS attemptable_type,
                 lesson_enrollments.position as le_position,
                 activities.position as a_position,
                 1 AS activity_item_position,
                 1 AS item_step_groups_position,
                 chapters.position as attemptable_position
            FROM lesson_enrollments
                 JOIN activities
                      ON lesson_enrollments.lesson_id = activities.lesson_id
                 LEFT JOIN activity_exclusions
                      ON lesson_enrollments.id = activity_exclusions.lesson_enrollment_id
                         AND activities.id = activity_exclusions.activity_id
                 JOIN chapters
                      ON chapters.activity_id = activities.id
                         AND activities.state = 'Published'
                 LEFT JOIN attempts
                      ON lesson_enrollments.id = attempts.lesson_enrollment_id
                         AND activities.id = attempts.activity_id
                         AND 'Chapter' = attempts.attemptable_type
                         AND chapters.id = attempts.attemptable_id
                         AND attempts.final is true
          WHERE lesson_enrollments.enrollment_id = $1
                AND activity_exclusions.id IS NULL
                AND attempts.id IS NULL
                AND lesson_enrollments.passed IS NULL
        ) tbl0

        ORDER BY tbl0.le_position,
                 tbl0.a_position,
                 activity_item_position,
                 item_step_groups_position,
                 attemptable_position
        limit 1





## Enrollments, Lesson enrollments, Enrollment activities

### Enrollments with 1 lesson left in pathway

This is a rough guess, as it assumes the student has passed all but
one pathway lessons, so it won't include enrollments with deferred
lessons. It makes other assumptions that shouldn't impact it too much
(pathway lessons haven't been changed, and lessons haven't been
deactivated).

    WITH enrollment_pathway_lesson_count(enrollment_id, student_id, classroom_id, in_process, total, created_at) AS (
           SELECT e.id as enrollment_id,
                  e.student_id,
                  e.classroom_id,
                  (SELECT COUNT(*)
                     FROM lesson_enrollments le
                          JOIN pathway_lessons pl
                               ON pl.lesson_id = le.lesson_id
                                  AND pl.pathway_id = e.pathway_id
                    WHERE le.enrollment_id = e.id
                      AND le.passed = true) as in_process,
                  (SELECT COUNT(*)
                     FROM pathway_lessons pl
                    WHERE pl.pathway_id = e.pathway_id) as total,
                  e.created_at
             FROM enrollments e
                  JOIN pathways p ON e.pathway_id = p.id
            WHERE p.state_id IN (1, 45)
         ORDER BY 4 DESC
     )
     SELECT ec.*
       FROM enrollment_pathway_lesson_count ec
      WHERE ec.in_process + 1 = ec.total;

### Deferred in enrollment

        SELECT le.lesson_id,
               MIN(le.position) as low_position
          FROM lesson_enrollments le
         WHERE le.enrollment_id = 339907
               AND le.passed IS false                      -- Failed twice: I was failed...
               AND le.other_enrollments_failed_count = 1   -- ...and the same lesson was before me was too!
      GROUP BY le.lesson_id
        HAVING MAX(le.other_enrollments_failed_count) = 1  -- ...but not too many (> 2 means skip => see #32019291)
      ORDER BY MIN(le.position)

### Lessons + status for enrollment

      SELECT le.position, le.reason,
             CASE ( SELECT COUNT(*) FROM pathway_lessons pl WHERE pl.pathway_id = e.pathway_id AND pl.lesson_id = le.lesson_id )
                  WHEN 0 THEN 'NO'
                  ELSE 'YES' END as on_pathway,
             CASE le.passed
                  WHEN true THEN 'PASS'
                  WHEN false THEN 'FAIL'
                  ELSE 'N/A' END as passed,
             CASE le.pre_quiz_score
                  WHEN null THEN '-'
                  ELSE to_char(le.pre_quiz_score*100, '999.9') END as prequiz,
             CASE le.post_quiz_score
                  WHEN null THEN '-'
                  ELSE to_char(le.post_quiz_score*100, '999.9') END as postquiz,
             l.id as lesson_id,
             CASE l.active
                  WHEN true THEN 'YES'
                  ELSE 'NO' END as lesson_active,
             le.other_enrollments_failed_count as prev_failed,
             l.name
        FROM enrollments e
             JOIN lesson_enrollments le on le.enrollment_id = e.id
             JOIN lessons l on l.id = le.lesson_id
       WHERE e.id = 1926001
    ORDER BY le.position;


### List activities, items and item steps for lesson enrollment (using EnrollmentActivity)

      SELECT le.id as le_id,
             le.position as le_pos,
             act.id as a_id,
             ea.position as ea_pos,
             ea.completed as ea_cmplt,
             ea.excluded as ea_excl,
             --substring( act.name from 1 for 8 ) || '...' as aname,
             act.activity_type as atype,
             eaa.position as attm_pos,
             eaa.completed as attm_complt,
             ite.id as i_id,
             ist.id as s_id,
             substring( ist.stem from 1 for 20 ) as stem
        FROM enrollments e
             JOIN lesson_enrollments le ON le.enrollment_id = e.id
             JOIN enrollment_activities ea
                  ON ea.lesson_enrollment_id = le.id
             JOIN enrollment_activity_attemptables eaa
                  ON eaa.enrollment_activity_id = ea.id
             JOIN activities act
                  ON act.id = ea.activity_id
             JOIN activity_items ai
                  ON ai.item_id = eaa.item_id
                     AND ai.activity_id = act.id
             JOIN items ite
                  ON ite.id = eaa.item_id
             JOIN item_steps ist
                  ON ist.id = eaa.attemptable_id
       WHERE e.id = 36660
    ORDER BY le_pos, ea_pos, attm_pos;

### List activities, items and item steps for lesson enrollment (using Lesson)

       SELECT le.id as le_id,
              act.id as a_id,
              act.position as apos,
              --substring( act.name from 1 for 8 ) || '...' as aname,
              act.activity_type as atype,
              ai.position as attm_pos,
              ite.id as i_id,
              ite.active as i_active,
              ist.id as s_id,
              substring( ist.stem from 1 for 18 ) as stem
         FROM item_steps ist
              JOIN items ite ON ite.id = ist.item_id
              JOIN activity_items ai ON ai.item_id = ite.id
              JOIN activities act ON act.id = ai.activity_id
              JOIN lessons l ON l.id = act.lesson_id
              JOIN lesson_enrollments le ON le.lesson_id = l.id
        WHERE le.id = 69379636
     ORDER BY apos, attm_pos;

### Find incomplete lesson enrollments where all activities are done

This shouldn't happen anymore, but...

    SELECT le.id, le.position
      FROM lesson_enrollments le
     WHERE le.passed is null
           AND le.tested_out is null
           AND ( select count(*) from enrollment_activities ea
                  where ea.lesson_enrollment_id = le.id ) =
               ( select count(*) from enrollment_activities ea
                  where ea.lesson_enrollment_id = le.id
                        AND ea.completed is true );

## EnrollmentActivityAttemptable

### Attemptables remaining within a lesson enrollment

    SELECT activities.name, activities.activity_type,
           ea.position as activity_position,
           att.attemptable_type || ' ' || att.attemptable_id as attemptable,
           att.position as attemptable_position
      FROM enrollment_activities ea
           JOIN activities ON activities.id = ea.activity_id
           JOIN enrollment_activity_attemptables att ON att.enrollment_activity_id = ea.id
     WHERE ea.lesson_enrollment_id = 56355697
           AND ea.completed = 'f'
           AND att.completed = 'f'
     ORDER BY ea.position, att.position;

### All attemptables within a Lesson Enrollment

    SELECT le.id as le_id,
           substring(lessons.name from 1 for 12) as lesson,
           le.reason,
           le.position as le_pos,
           le.passed as le_pass, le.tested_out as le_tst,
           activities.id as actv_id,
           substring(activities.name from 1 for 12 ) as actv_name,
           activities.activity_type as actv_type,
           ea.position as ea_pos,
           ea.completed as ea_cmp,
           att.item_id,
           att.attemptable_type || ' ' || att.attemptable_id as att,
           att.position as att_pos,
           att.completed as att_cmp, att.final_attempt_id as attempt
      FROM lesson_enrollments le
           JOIN enrollment_activities ea ON ea.lesson_enrollment_id = le.id
           JOIN activities ON activities.id = ea.activity_id
           JOIN enrollment_activity_attemptables att ON att.enrollment_activity_id = ea.id
           JOIN lessons ON lessons.id = le.lesson_id
     WHERE le.id = 65728751
     ORDER BY le.position, ea.position, att.position;


### All attemptables within an enrollment

    SELECT le.id as le_id,
           substring(lessons.name from 1 for 12) as lesson,
           le.reason,
           le.position as le_pos,
           le.passed as le_pass, le.tested_out as le_tst,
           activities.id as actv_id,
           substring(activities.name from 1 for 12 ) as actv_name,
           activities.activity_type as actv_type,
           ea.position as ea_pos,
           ea.completed as ea_cmp,
           att.item_id,
           att.attemptable_type || ' ' || att.attemptable_id as att,
           att.position as att_pos,
           att.completed as att_cmp, att.final_attempt_id as attempt
      FROM lesson_enrollments le
           JOIN enrollment_activities ea ON ea.lesson_enrollment_id = le.id
           JOIN activities ON activities.id = ea.activity_id
           JOIN enrollment_activity_attemptables att ON att.enrollment_activity_id = ea.id
           JOIN lessons ON lessons.id = le.lesson_id
     WHERE le.enrollment_id = 1580616
     ORDER BY le.position, ea.position, att.position;




## Attempts

### List attempts for a classroom's enrollments

     SELECT a.created_at, a.id, a.correct, a.final, a.student_id,
            a.earned_points, a.possible_points
       FROM attempts a
            JOIN lesson_enrollments le ON le.id = a.lesson_enrollment_id
            JOIN enrollments e ON e.id = le.enrollment_id
      WHERE e.classroom_id = 92760;

### List attempts + activity numbers for a lesson enrollment

       SELECT att.id as att_id,
              att.correct as att_ok,
              att.attemptable_type as att_type,
              att.attemptable_id as att_att_id,
              resp.id as r_id,
              substring( resp.response_text from 1 for 15 ) as r_text,
              act.id as act_id,
              act.position as act_pos,
              substring( act.name from 1 for 8 ) || '...' as act_name,
              substring( act.activity_type from 1 for 8 ) || '...' as act_type,
              its.id as s_id,
              substring( its.stem from 1 for 10 ) as stem
         FROM attempts att
              JOIN responses resp ON att.response_id = resp.id
              JOIN item_steps its ON its.id = resp.item_step_id
              JOIN items ite ON ite.id = its.item_id
              JOIN activities act ON act.id = att.activity_id
              JOIN lessons l ON l.id = act.lesson_id
        WHERE att.lesson_enrollment_id = 530263
     ORDER BY att_id;

### Find latest attempt by classroom

Warning, this will take a while to run on a full production system!

    SELECT
      enrollments.classroom_id, max(attempts.created_at)
    FROM
      attempts
      JOIN enrollments ON enrollments.id = attempts.enrollment_id
    GROUP BY
      enrollments.classroom_id


## Pathways, Lessons + pre-requisites, Activities

## Lesson grades within all units

    SELECT
      units.title, array_agg(DISTINCT grade_levels.name) as grade
    FROM
      units
      JOIN lessons on lessons.unit_id = units.id
      JOIN grade_levels on grade_levels.id = lessons.grade_level_id
    GROUP BY
      units.title;

### Lessons + lesson state in pathway

      SELECT p.name, pl.position, l.id as lesson_id,
             CASE l.active
                  WHEN true THEN 'YES'
                  ELSE 'NO' END as lesson_active,
             l.name,
             gl.name as grade_level
        FROM pathways p
             JOIN pathway_lessons pl ON pl.pathway_id = p.id
             JOIN lessons l on l.id = pl.lesson_id
             JOIN grade_levels gl on gl.id = l.grade_level_id
       WHERE p.id = 2
    ORDER BY pl.position;

### Prerequisites for all grades in all published pathways

This one does every grade separately:

    SELECT
      p.id as pathway_id,
      pgl.number as pathway_grade,
      rgl.number as req_grade,
      array_length( array_agg( DISTINCT rln.id ), 1 ) as prereq_len,
      array_agg( DISTINCT rln.id ) as prereqs
    FROM
      lesson_requisites reqs
      JOIN pathway_lessons pl ON pl.lesson_id = reqs.descendant_id
      JOIN pathways p ON p.id = pl.pathway_id
      JOIN lessons pln ON pln.id = pl.lesson_id
      JOIN lessons rln ON rln.id = reqs.ancestor_id
      JOIN grade_levels pgl ON pgl.id = p.grade_level_id
      JOIN grade_levels rgl ON rgl.id = rln.grade_level_id
    WHERE
      p.demo = false
      AND p.published = true
      AND rgl.number < pgl.number
      AND pln.active = true
      AND rln.active = true
    GROUP BY
      p.id, pgl.number

### First TTM-created, published, non-demo pathway for a state and grade

      SELECT DISTINCT
        states.code,
        gl.number,
        first_value(pathways.id)
      OVER(PARTITION BY states.code, gl.number ORDER BY pathways.created_at)
      FROM
        pathways
        JOIN grade_levels gl ON gl.id = pathways.grade_level_id
        JOIN states ON states.id = pathways.state_id
      WHERE
        pathways.published = true
        AND pathways.demo = false
        AND pathways.customer_id IS NULL
      ORDER BY
        states.code, gl.number;

### Name and grade for lesson

by id

       SELECT l.id as lsn_id,
              l.active, l.unit_id,
              substring( l.name from 1 for 30 ) as name,
              gl.name || '/' || gl.number as grade
         FROM lessons l
              JOIN grade_levels gl ON gl.id = l.grade_level_id
        WHERE l.id = 352;

by name:

       SELECT l.id as lsn_id,
              l.active, l.unit_id,
              substring( l.name from 1 for 30 ) as name,
              gl.name || '/' || gl.number as grade
         FROM lessons l
              JOIN grade_levels gl ON gl.id = l.grade_level_id
        WHERE l.name like 'Bar charts%';

### Name and info for pathway

       SELECT p.id, p.name,
              CASE p.use_placement_test
                   WHEN true THEN 'YES'
                   ELSE 'NO' END as placement,
              CASE p.use_remediation
                   WHEN true THEN 'YES'
                   ELSE 'NO' END as remediate,
              CASE p.allow_test_out
                   WHEN true THEN 'YES'
                   ELSE 'NO' END as test_out,
              CASE p.require_pre_quiz
                   WHEN true THEN 'YES'
                   ELSE 'NO' END as prequiz_req,
              gl.name as grade_name,
              gl.number as grade_number
         FROM pathways p
              JOIN grade_levels gl ON p.grade_level_id = gl.id
        WHERE p.id = 2;


### List prereqs for pathway

       SELECT pl.lesson_id as p_lid, pl.position as p_pos,
              substring( pln.name from 1 for 40 ) as p_lesson_name,
              pgl.name as p_grade,
              substring( rln.name from 1 for 40 ) as req_lesson_name,
              rgl.name as req_grade
         FROM lesson_requisites reqs
              JOIN pathway_lessons pl ON pl.lesson_id = reqs.descendant_id
              JOIN lessons pln ON pln.id = pl.lesson_id
              JOIN lessons rln ON rln.id = reqs.ancestor_id
              JOIN grade_levels pgl ON pgl.id = pln.grade_level_id
              JOIN grade_levels rgl ON rgl.id = rln.grade_level_id
        WHERE pl.pathway_id = 89206
              AND rln.active = true
     ORDER BY pl.position, rgl.sort_order;

### List prereqs for pathway and grade number

       SELECT pl.lesson_id as p_lid,
              substring(pln.name from 1 for 40) as p_lesson_name,
              pgl.name as p_grade,
              rln.id as req_lid,
              substring(rln.name from 1 for 40) as req_lesson_name,
              rgl.name as req_grade
         FROM lesson_requisites reqs
              JOIN pathway_lessons pl ON pl.lesson_id = reqs.descendant_id
              JOIN lessons pln ON pln.id = pl.lesson_id
              JOIN lessons rln ON rln.id = reqs.ancestor_id
              JOIN grade_levels pgl ON pgl.id = pln.grade_level_id
              JOIN grade_levels rgl ON rgl.id = rln.grade_level_id
        WHERE pl.pathway_id = 5
              AND rgl.number = 3
              AND pln.active = true
              AND rln.active = true
     ORDER BY req_lid

### List prereqs for lesson (given lesson = descendant, or "what are my prereqs?" )

       SELECT pln.id as lsn_id,
              pln.active as lsn_active,
              substring( pln.name from 1 for 12 ) as lsn_name,
              pgl.name as lsn_grade,
              rln.id as req_lid,
              rln.active as req_active,
              substring( rln.name from 1 for 30 ) as req_name,
              rgl.name as req_grade
         FROM lessons pln
              JOIN lesson_requisites reqs ON pln.id = reqs.descendant_id
              JOIN lessons rln ON rln.id = reqs.ancestor_id
              JOIN grade_levels pgl ON pgl.id = pln.grade_level_id
              JOIN grade_levels rgl ON rgl.id = rln.grade_level_id
        WHERE pln.id = 17
     ORDER BY rgl.sort_order;

### List prereqs for lesson (given lesson = ancestor, or "what am I a prereq for?")

       SELECT rln.id as lsn_id,
              substring( rln.name from 1 for 30 ) as lsn_name,
              rgl.name as lsn_grade,
              pln.id as req_lid,
              substring( pln.name from 1 for 12 ) as req_name,
              pgl.name as req_grade
         FROM lessons pln
              JOIN lesson_requisites reqs ON pln.id = reqs.ancestor_id
              JOIN lessons rln ON rln.id = reqs.descendant_id
              JOIN grade_levels pgl ON pgl.id = pln.grade_level_id
              JOIN grade_levels rgl ON rgl.id = rln.grade_level_id
        WHERE pln.id = 72
     ORDER BY rgl.sort_order;

### Find prereqs that have higher grade level than lesson (given lesson = descendant, or "what are my prereqs?" )

       SELECT pln.id as lsn_id,
              pln.active as lsn_active,
              pln.name as lsn_name,
              pgl.name || '/' || pgl.number as lsn_grade,
              rln.id as req_lid,
              rln.active as req_active,
              rln.name as req_name,
              rgl.name || '/' || rgl.number as req_grade
         FROM lessons pln
              JOIN lesson_requisites reqs ON pln.id = reqs.descendant_id
              JOIN lessons rln ON rln.id = reqs.ancestor_id
              JOIN grade_levels pgl ON pgl.id = pln.grade_level_id
              JOIN grade_levels rgl ON rgl.id = rln.grade_level_id
        WHERE pgl.number < rgl.number
     ORDER BY rgl.sort_order;


### Activities and items for lesson

       SELECT lessons.id as lsn_id,
              substring( lessons.name from 1 for 10) as lsn_name,
              activities.position as apos,
              activities.id as a_id,
              substring( activities.name from 1 for 8 ) as aname,
              activities.activity_type as atype,
              activity_items.position as ipos,
              items.id as i_id,
              items.item_number as inum,
              item_steps.id as s_id,
              substring( item_steps.stem from 1 for 18 ) as stem,
              responses.id as r_id,
              CASE responses.is_correct WHEN true THEN 'YES' ELSE 'NO' END as rcorrect,
              responses.position as rpos,
              substring( responses.response_text from 1 for 15 ) as rtext
         FROM lessons
              JOIN activities ON activities.lesson_id = lessons.id
              JOIN activity_items ON activity_items.activity_id = activities.id
              JOIN items ON items.id = activity_items.item_id
              JOIN item_steps ON item_steps.item_id = items.id
              JOIN responses ON responses.item_step_id = item_steps.id
        WHERE lessons.id = 352
     ORDER BY apos, ipos, rpos;





## Items, item steps, item step groups

### Last step for all last groups in all items

    WITH max_group_position(item_id, group_position) AS (
            SELECT item_id, MAX(position)
              FROM item_step_groups
          GROUP BY item_id
    ),
    max_step_position(group_id, step_position) AS (
        SELECT step.item_step_group_id as group_id,
               MAX(step.position) as step_position
          FROM item_steps step
      GROUP BY step.item_step_group_id
    )
    SELECT items.id as item_id,
           grp.id as group_id,
           grp.position as group_position,
           step.id as step_id,
           step.position as step_position,
           (SELECT COUNT(*)
             FROM max_step_position msp
            WHERE grp.id = msp.group_id
                  AND step.position = msp.step_position) as last_in_group
      FROM items
           JOIN item_step_groups grp
                ON grp.item_id = items.id
           JOIN item_steps step
                ON step.item_step_group_id = grp.id
     ORDER BY item_id, group_position, step_position;


           LEFT OUTER JOIN max_step_position msp
                ON msp.group_id = step.item_step_group_id
                   AND msp.step_position = step.position
           LEFT OUTER JOIN max_group_position mgp
                ON mgp.item_id = grp.item_id
                   AND mgp.group_position = grp.position

### Items and item steps and responses in activity

       SELECT activities.id as a_id,
              substring( activities.name from 1 for 8 ) as aname,
              activities.activity_type as atype,
              activity_items.position as ipos,
              items.id as i_id,
              items.item_number as inum,
              item_steps.id as s_id,
              substring( item_steps.stem from 1 for 18 ) as stem,
              responses.id as r_id,
              CASE responses.is_correct WHEN true THEN 'YES' ELSE 'NO' END as rcorrect,
              responses.position as rpos,
              substring( responses.response_text from 1 for 15 ) as rtext
         FROM activities
              JOIN activity_items ON activity_items.activity_id = activities.id
              JOIN items ON items.id = activity_items.item_id
              JOIN item_steps ON item_steps.item_id = items.id
              JOIN responses ON responses.item_step_id = item_steps.id
        WHERE activities.id = 1900
     ORDER BY ipos, rpos;


### Item, item_steps and responses for an Item

       SELECT items.id as i_id,
              items.item_number as inum,
              item_steps.id as s_id,
              substring( item_steps.stem from 1 for 18 ) as stem,
              responses.id as r_id,
              CASE responses.is_correct WHEN true THEN 'YES' ELSE 'NO' END as rcorrect,
              responses.position as rpos,
              substring( responses.response_text from 1 for 15 ) as rtext
         FROM items
              JOIN item_steps ON item_steps.item_id = items.id
              JOIN responses ON responses.item_step_id = item_steps.id
        WHERE items.item_number = '6391'
     ORDER BY rpos;


### Item tree suitable for splicing among shards

       SELECT
         items.id as i_id,
         items.item_number as inum,
         item_steps.id as s_id,
         item_steps.position as spos,
         substring(item_steps.stem from 1 for 50) as stem,
         responses.id as r_id,
         responses.position as rpos,
         CASE responses.is_correct WHEN true THEN 'YES' ELSE 'NO' END as rcorrect,
         substring(responses.response_text from 1 for 100) as rtext
       FROM
         items
         JOIN item_steps ON item_steps.item_id = items.id
         JOIN responses ON responses.item_step_id = item_steps.id
        WHERE
          items.id = 13907
        ORDER BY
          spos, rpos;

### List item step groups + item steps for an item

      SELECT isg.id as group_id, isg.position as group_pos,
             isg.name as group_name,
             ist.id as step_id, ist.position as step_pos,
             substring(ist.stem from 1 for 30) as step_stem
        FROM item_step_groups isg
             JOIN item_steps ist ON ist.item_step_group_id = isg.id
       WHERE isg.item_id = 6116
    ORDER BY isg.position, ist.position;


### List item steps for items in a particular grade

      SELECT
        array_agg(DISTINCT lessons.name) as lesson_names,
        items.id as item_id,
        CASE items.active WHEN true THEN 'YES' ELSE 'NO' END as active,
        CASE items.use_for_placement WHEN true THEN 'PLACEMENT' ELSE 'NOT PLACEMENT' END as in_placement,
        items.state as item_state,
        grade_levels.name as grade,
        item_steps.id as step_id,
        substring(item_steps.stem from 1 for 60) as step_stem
      FROM
        items
        JOIN item_steps ON item_steps.item_id = items.id
        JOIN grade_levels ON grade_levels.id = items.grade_level_id
        JOIN activity_items ON activity_items.item_id = items.id
        JOIN activities ON activities.id = activity_items.activity_id
        JOIN lessons ON lessons.id = activities.lesson_id
      WHERE
        items.grade_level_id IN (5, 9)
      GROUP BY
        items.id, items.active, items.use_for_placement, items.state, grade_levels.name, item_steps.id, item_steps.stem
      ORDER BY
        item_steps.stem;

### Update items, item_steps, responses to ensure that published ones do not have version_of_*_id

Remember to run this on all shards!

    BEGIN WORK;

    UPDATE
      items
    SET
      version_of_item_id = null
    WHERE
      state = 'Published'
      AND version_of_item_id is not null;

    UPDATE
      item_steps
    SET
      version_of_item_step_id = null
    FROM
      items
    WHERE
      item_steps.version_of_item_step_id is not null
      AND items.state = 'Published'
      AND item_steps.item_id = items.id;

    UPDATE
      responses
    SET
      version_of_response_id = null
    FROM
      items, item_steps
    WHERE
      responses.version_of_response_id is not null
      AND items.state = 'Published'
      AND item_steps.item_id = items.id
      AND responses.item_step_id = item_steps.id;

    UPDATE
      hints
    SET
      version_of_hint_id = null
    FROM
      items, item_steps
    WHERE
      hints.version_of_hint_id is not null
      AND items.state = 'Published'
      AND item_steps.item_id = items.id
      AND hints.item_step_id = item_steps.id;

    --- check that it looks okay, then:

    COMMIT;

### FITB items

      SELECT items.id, items.item_number, items.state, items.active,
             isg.id as group_id, isg.position as group_pos,
             isg.name as group_name,
             ist.id as step_id, ist.position as step_pos,
             substring(ist.stem from 1 for 30) as step_stem
        FROM item_steps ist
             JOIN item_step_groups isg ON isg.id = ist.item_step_group_id
             JOIN items ON items.id = isg.item_id
       WHERE ist.responses_display_class = 'fill_in_the_blank'
    ORDER BY items.item_number, isg.position, ist.position;




## Responses

### Responses for item (assuming non-PSP)

by ID:

      SELECT items.id as item_id, items.item_number, items.state,
             items.active, items.grade_level_id as gl,
             CASE items.use_for_placement WHEN true THEN 'YES' ELSE 'NO' END as place,
             item_steps.id as step_id,
             responses.id as resp_id,
             CASE responses.is_correct WHEN true THEN 'YES' ELSE 'NO' END as correct,
             responses.position,
             substring( responses.response_text from 1 for 15 ) as text
        FROM items
             JOIN item_steps ON item_steps.item_id = items.id
             JOIN responses ON responses.item_step_id = item_steps.id
       WHERE items.id = 12115
    ORDER BY responses.position;

by item number:

      SELECT items.id as item_id, items.item_number, items.state,
             items.active, items.grade_level_id as gl,
             CASE items.use_for_placement WHEN true THEN 'YES' ELSE 'NO' END as place,
             item_steps.id as step_id,
             responses.id as resp_id,
             CASE responses.is_correct WHEN true THEN 'YES' ELSE 'NO' END as correct,
             responses.position,
             substring( responses.response_text from 1 for 15 ) as text
        FROM items
             JOIN item_steps ON item_steps.item_id = items.id
             JOIN responses ON responses.item_step_id = item_steps.id
       WHERE items.item_number = '88592'
    ORDER BY responses.position;

### Responses for item (with groups, by item number)

      SELECT items.id as item_id, items.item_number, items.state,
             items.active, items.grade_level_id as gl,
             CASE items.use_for_placement WHEN true THEN 'YES' ELSE 'NO' END as place,
             isg.id as group_id,
             isg.position as group_pos,
             isg.name as group_name,
             item_steps.id as step_id,
             item_steps.position as step_pos,
             responses.id as resp_id,
             CASE responses.is_correct WHEN true THEN 'YES' ELSE 'NO' END as correct,
             responses.position as resp_pos,
             substring( responses.response_text from 1 for 15 ) as text
        FROM items
             JOIN item_step_groups isg ON isg.item_id = items.id
             JOIN item_steps ON item_steps.item_step_group_id = isg.id
             JOIN responses ON responses.item_step_id = item_steps.id
       WHERE items.item_number = '30'
    ORDER BY items.id, isg.position, item_steps.position, responses.position;


### Find feedback + text by response

       SELECT fb.id AS feedback_id,
              fb.response_id,
              fbs.id AS feedback_step_id,
              fbs.text,
              fbs.position
         FROM feedback_steps fbs
              JOIN feedbacks fb ON fb.id = fbs.feedback_id
        WHERE fb.response_id = 99524
     ORDER BY fbs.position;

### Find responses by placement test

       SELECT r.id, r.is_correct, r.position, r.item_step_id,
              r.version_of_response_id, r.response_text
         FROM responses r
              JOIN placement_attempts pa ON pa.response_id = r.id
        WHERE pa.placement_test_id = 10430
     ORDER BY r.id;



## Students

### Student + school count by customer

    SELECT
      customers.id   as customer_id,
      customers.name as customer_name,
      states.code    as state,
      (SELECT count(*) FROM schools WHERE schools.customer_id = customers.id) as school_count,
      (SELECT count(students)
         FROM students
              JOIN classroom_students on students.id = classroom_students.student_id
              JOIN classrooms on classroom_students.classroom_id = classrooms.id
              JOIN schools on classrooms.school_id = schools.id
        WHERE schools.customer_id = customers.id) as student_count
    FROM
      customers
      JOIN states ON states.id = customers.state_id
    ORDER BY
      5 desc;


### Student + school count by district, with percent started

    WITH counts(district_id, school_count, all_students, active_students, started_students) AS (
      SELECT
        students.district_id,
        COUNT(DISTINCT school_id) as school_count,
        COUNT(students.*) as all_student_count,
        SUM(CASE WHEN students.is_active THEN 1 ELSE 0 END) as active_student_count,
        SUM(CASE WHEN students.started THEN 1 ELSE 0 END) as started_student_count
      FROM
        students
      GROUP BY
        students.district_id
    )
    SELECT
      districts.id as district_id,
      districts.name as district_name,
      counts.school_count,
      counts.all_students,
      counts.active_students,
      counts.started_students,
      CASE counts.all_students
        WHEN 0 THEN 0
        ELSE counts.started_students / counts.all_students::float
      END as percent_started
    FROM
      districts
      JOIN counts ON counts.district_id = districts.id
    ORDER BY
      7 DESC;

### Student count by school, with percent started

    WITH counts(school_id, all_students, active_students, started_students) AS (
      SELECT
        students.school_id,
        COUNT(students.*) as all_student_count,
        SUM(CASE WHEN students.is_active THEN 1 ELSE 0 END) as active_student_count,
        SUM(CASE WHEN students.started THEN 1 ELSE 0 END) as started_student_count
      FROM
        students
      GROUP BY
        students.school_id
    )
    SELECT
      schools.id as school_id,
      schools.name as school_name,
      counts.all_students,
      counts.active_students,
      counts.started_students,
      CASE counts.all_students
        WHEN 0 THEN 0
        ELSE counts.started_students / counts.all_students::float
      END as percent_started
    FROM
      schools
      JOIN counts ON counts.school_id = schools.id
    ORDER BY
      6 DESC;

### Schools and classroom count

    WITH school_students(school_id, student_count) AS (
      SELECT
        students.school_id, count(*)
      FROM
        students
      GROUP BY
        students.school_id
    ),
    school_classrooms(school_id, classroom_count) AS (
      SELECT
        classrooms.school_id, count(*)
      FROM
        classrooms
      GROUP BY
        classrooms.school_id
    )
    SELECT
      schools.id,
      schools.name,
      schools.shard_name,
      school_classrooms.classroom_count,
      school_students.student_count
    FROM
      schools
      JOIN school_students ON school_students.school_id = schools.id
      JOIN school_classrooms ON school_classrooms.school_id = schools.id
    WHERE
      schools.name not ilike '%demo%'
    ORDER BY 4 DESC;

### Classroom + student count within a school

    SELECT
      classrooms.id, classrooms.name,
      count(classroom_students.*) as student_count
    FROM
      classrooms
      join classroom_students on classroom_students.classroom_id = classrooms.id
    WHERE
      classrooms.school_id = 6347
    GROUP BY
      classrooms.id, classrooms.name;

### Students and grades within a district

    SELECT DISTINCT
      schools.name as school,
      students.last_name, students.first_name,
      grade_levels.name as grade
    FROM
      schools
      JOIN classrooms ON classrooms.school_id = schools.id
      JOIN classroom_students ON classroom_students.classroom_id = classrooms.id
      JOIN students ON students.id = classroom_students.student_id
      JOIN grade_levels ON grade_levels.id = students.grade_level_id
    WHERE
      schools.district_id = 1273
    ORDER BY
      schools.name, students.last_name, students.first_name;


### Students currently in remediation

    SELECT s.first_name, s.last_name, s.id as student_id,
           e.id as enrollment_id
      FROM lesson_enrollments le
           JOIN enrollments e ON e.id = le.enrollment_id
           JOIN students s ON s.id = e.student_id
     WHERE le.reason = 'remediation'
           AND le.passed IS NULL
           AND le.tested_out IS NULL
     LIMIT 50;

### Students starting an activity

Three versions here, I forget which worked or if they all work in
different ways. Sorry!

    SELECT DISTINCT s.first_name, s.last_name, s.id as student_id,
           e.id as enrollment_id,
           le.id as lesson_enrollment_id,
           activities.name as activity,
           activities.activity_type
      FROM enrollment_activities ea
           JOIN enrollments e ON e.id = le.enrollment_id
           JOIN students s ON s.id = e.student_id
           JOIN enrollment_activities ea ON ea.lesson_enrollment_id = le.id
           JOIN activities ON ea.activity_id = activities.id
     WHERE ea.completed = 'f'
           AND le.passed IS NULL
           AND le.tested_out IS NULL
           AND 0 = ( SELECT COUNT(*) FROM attempts a
                      WHERE a.lesson_enrollment_id = le.id
                            AND a.activity_id = ea.activity_id )
     LIMIT 50;

    SELECT DISTINCT s.first_name, s.last_name, s.id as student_id,
           e.id as enrollment_id,
           le.id as lesson_enrollment_id,
           ea.id as enrollment_activity_id,
           ea.lesson_enrollment_id as lesson_enrollment_id,
           activities.name as activity,
           activities.activity_type
      FROM enrollment_activities ea
           JOIN activities ON ea.activity_id = activities.id
           JOIN lesson_enrollments le on le.id = ea.lesson_enrollment_id
           JOIN enrollments e ON e.id = le.enrollment_id
           JOIN students s ON s.id = e.student_id
     WHERE ea.completed = 'f'
           and le.tested_out is null
           and le.passed is null
           AND 0 = ( SELECT COUNT(*) FROM attempts a
                      WHERE a.lesson_enrollment_id = ea.lesson_enrollment_id
                            AND a.activity_id = ea.activity_id )
     LIMIT 50;

    SELECT DISTINCT s.first_name, s.last_name, s.id as student_id,
           e.id as enrollment_id,
           le.id as lesson_enrollment_id
      FROM lesson_enrollments le
           JOIN enrollments e ON e.id = le.enrollment_id
           JOIN students s ON s.id = e.student_id
     WHERE le.tested_out is null
           and le.passed is null
           AND le.id IN (
                SELECT ea.lesson_enrollment_id as lesson_enrollment_id
                  FROM enrollment_activities ea
                       JOIN activities ON ea.activity_id = activities.id
                 WHERE ea.completed = 'f'
                       AND 0 = ( SELECT COUNT(*) FROM attempts a
                                  WHERE a.lesson_enrollment_id = ea.lesson_enrollment_id
                                        AND a.activity_id = ea.activity_id )
                 LIMIT 500
           )
    LIMIT 50;


                SELECT ea.lesson_enrollment_id as lesson_enrollment_id
                  FROM enrollment_activities ea
                       JOIN activities ON ea.activity_id = activities.id
                 WHERE ea.completed = 'f'
                       AND 0 = ( SELECT COUNT(*) FROM attempts a
                                  WHERE a.lesson_enrollment_id = ea.lesson_enrollment_id
                                        AND a.activity_id = ea.activity_id )
                 LIMIT 500

### Students currently within an activity

    SELECT s.first_name, s.last_name, s.id as student_id,
           e.id as enrollment_id,
           le.id as lesson_enrollment_id
      FROM lesson_enrollments le
           JOIN enrollments e ON e.id = le.enrollment_id
           JOIN students s ON s.id = e.student_id
           JOIN enrollment_activities ea ON ea.lesson_enrollment_id = le.id
     WHERE le.passed IS NULL
           AND le.tested_out IS NULL
           AND ea.completed = 'f'
           AND 0 < ( SELECT COUNT(*)
                       FROM enrollment_activity_attemptables att
                      WHERE att.enrollment_activity_id = ea.id
                            AND att.completed = 't' )
     LIMIT 50;

### Finding students + enrollments with five step word problems

    SELECT distinct stu.id as student_id
           ,stu.username
           ,le.enrollment_id as enr_id
           ,le.id as lesson_enr_id
           ,le.passed as passed
           ,le.tested_out as tested_out
      FROM lesson_enrollments le
           JOIN enrollments enr ON enr.id = le.enrollment_id
           JOIN students stu ON stu.id = enr.student_id
     WHERE le.lesson_id IN (
            SELECT DISTINCT( l.id )
              FROM lessons l
                   JOIN activities act ON act.lesson_id = l.id
                   JOIN activity_items ai ON ai.activity_id = act.id
                   JOIN items it ON it.id = ai.item_id
                   JOIN item_steps itst ON itst.item_id = it.id
             WHERE itst.responses_display_class = 'five_step_word_problem'
           )

### Students who are active with no active classrooms and some inactive classrooms

    SELECT
      COUNT(*)
    FROM
      students
    WHERE
      is_active = true
      AND is_demo = false
      AND 0 = (
        SELECT
          COUNT(*)
        FROM
          classroom_students cs
          JOIN classrooms ON cs.classroom_id = classrooms.id
        WHERE
          cs.student_id = students.id
          AND classrooms.active = true
     )
     AND 0 < (
        SELECT
          COUNT(*)
        FROM
          classroom_students cs
          JOIN classrooms ON cs.classroom_id = classrooms.id
        WHERE
          cs.student_id = students.id
          AND classrooms.active = false
     );

## Processes

### Dump and restore a database on Scalr

SSH to the postgres box on the farm. Change to the postgres user and
create the database -- here we use the name 'enrollments', be sure to
change it as needed:

    # su postgres
    $ cd /mnt/pgstorage
    $ createdb --owner=u1rtbpnu2rctb6 --encoding=UTF8 --lc-collate=en_US.UTF-8 --lc-ctype=en_US.UTF-8 enrollments

Assign permissions:

    $ psql enrollments
    psql (9.3.1, server 9.2.5)
    Type "help" for help.

    enrollments=# grant all on database enrollments to u1rtbpnu2rctb6;
    GRANT
    enrollments=# grant all on schema public to u1rtbpnu2rctb6;
    GRANT
    enrollments=# grant all on all sequences in schema public to u1rtbpnu2rctb6;
    GRANT
    enrollments=# \q

Dump the database:

    $ pg_dump --clean -Fc -f ./file.dump Postgres_URI
    (wait a long time...)
    $ pg_restore --clean -U postgres -d enrollments -Fc file.dump

### Copy a table from one database to another

Assume we want to copy the contents of the table 'charities' from production to our local system:

    cwinters@abita:~/Projects/TTM/wiki$ ttmscalr psql master -f production
    Executing `psql postgres://u1rtbpnu2rctb6:p92m80no0889sifpk4oj171lcf5@ext.master.postgresql.d929354648e0a5.scalr-dns.net:5432/apangea`
    psql (9.3.1, server 9.2.5)
    SSL connection (cipher: DHE-RSA-AES256-SHA, bits: 256)
    Type "help" for help.

    apangea=> \copy charities to 'charities.txt'
    Time: 9.903 ms
    apangea=> \q
    cwinters@abita:~/Projects/TTM/wiki$ psql apangea_development
    psql (9.3.1, server 9.2.5)
    Type "help" for help.

    apangea_development=# truncate table charities;
    TRUNCATE TABLE
    Time: 653.125 ms
    apangea_development=# \copy charities from 'charities.txt'
    Time: 110.256 ms

One danger with doing this from the command line is that you __MUST BE
SURE__ you're not connected to production when you issue commands like
`TRUNCATE TABLE`, especially because that cannot be put into a
transaction.

Note that you can dump a subset of data this way as well:

    apangea=> \copy (select * from charities where name like '%Red Cross%') to 'charities.txt'


### Finding the most recent attempt time by school

Given the file `max_attempt_by_classroom.sql`:

    SELECT
      enrollments.classroom_id, max(attempts.created_at), count(attempts.*)
    FROM
      attempts
      JOIN enrollments ON enrollments.id = attempts.enrollment_id
    GROUP BY
      enrollments.classroom_id

Run from the `apangea` directory:

    $ cd db
    $ ruby dump_all_shards.rb ttm-production max_attempt_by_classroom.sql > max_attempts_by_classroom.txt

Once that's done (it takes a while), open up a console to the master database:

    $ heroku pg:psql -a ttm-production
      OR
    $ psql `heroku config:get DATABASE_URL -a ttm-production`

and run each of these in turn:

    -- create holding table
    CREATE TABLE max_attempt_by_school (
      customer_name varchar(255),
      school_name   varchar(255),
      school_id     int,
      classroom_id  int not null,
      attempt_time  timestamp not null,
      attempt_count int not null
    );

    -- copy in the data we just dumped
    \COPY max_attempt_by_school(classroom_id, attempt_time, attempt_count) FROM 'classroom_and_max_attempt.txt';

    -- decorate our data with the customer and school names
    UPDATE max_attempt_by_school
       SET school_id = schools.id,
           school_name = schools.name,
           customer_name = customers.name
      FROM classrooms, schools, customers
     WHERE classrooms.id = classroom_id
           AND schools.id = classrooms.school_id
           AND customers.id = schools.customer_id;

    -- finally, dump out the data for use in a spreadsheet
    \COPY (SELECT customer_name, school_name, school_id, MAX(attempt_time), SUM(attempt_count) FROM max_attempt_by_school GROUP BY customer_name, school_name, school_id ORDER BY 4) TO 'max_attempt_by_school.txt';

## Reviewing feed item interactions

If you are interested in seeing messages that teachers are sending to students through the activity feed, here's an SQL snippet to run:

<pre>
SELECT message_text FROM feed_item_interactions
INNER JOIN students ON feed_item_interactions.student_id = students.id
WHERE interaction_type = 'message'
AND students.is_demo = false
AND students.is_active = true
</pre>
