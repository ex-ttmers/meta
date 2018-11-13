For each __student__, the __individual student standards report__ shows:

- number of lessons in each standard that the student is 
	- failing (an average of less than 70% in the lessons associated to the standard)
	- passing (an average of 70% or higher in the lessons associated to the standard)
- number of lessons associated with each standard that the student is assigned to

The __individual student standards report__ can be filtered by:

- standard

__Draft SQL__

```
select
  student_source_id,
  standard_category_source_id,
  standard_source_id,
  sum(passing) as lessons_passing,
  sum(failing) as lessons_failing,
  sum(assigned) as lessons_assigned
from
(  select
    s.standard_category_source_id,
    s.standard_source_id,
    st.source_id as student_source_id,
    avg(r.lesson_score) as average_score_for_standard_category,
    case when avg(r.lesson_score) >= 0.7 then 1 else 0 end as passing,
    case when avg(r.lesson_score) < 0.7 then 1 else 0 end as failing,
    1 as assigned
  from
    (
      (
        select student_id, math_lesson_id,
          count(id) as attempts,
          max(id) as latest_lesson_completed
        from lesson_completed_facts
        group by student_id, math_lesson_id
      ) union (
        select student_id, m.id as math_lesson_id,
          0 as attempts,
          null as latest_lesson_completed
        from projected_lessons_by_student_by_pathways p
          inner join math_lesson_dimensions m
            on p.lesson_source_id = m.lesson_source_id
      )
    ) l
    left join
    (
      select lesson_completed_facts.id as lesson_completed_id,
        case when lesson_result_dimensions.tested_out = true
           then lesson_completed_facts.pre_quiz_score
           else lesson_completed_facts.post_quiz_score
        end as lesson_score
      from lesson_completed_facts
        inner join lesson_result_dimensions
          on lesson_completed_facts.lesson_result_id = lesson_result_dimensions.id
    ) r
      on l.latest_lesson_completed = r.lesson_completed_id
    inner join (select id, lesson_source_id, unnest(standard_source_ids) as standard_source_id
          from math_lesson_dimensions) ls
      on l.math_lesson_id = ls.id
    inner join standard_dimensions s
      on ls.standard_source_id = s.standard_source_id
    inner join student_dimensions st
      on l.student_id = st.id
    JOIN (
            SELECT
              id as student_for_classroom_id
            FROM
              student_dimensions
            WHERE
              source_id in
              (
                SELECT
                  source_id
                FROM
                  student_dimensions
                WHERE
                  is_latest
              )
          ) student_classroom ON student_classroom.student_for_classroom_id = l.student_id
  group by
    s.standard_category_source_id,
    s.standard_source_id,
    st.source_id
) as student_status_per_standard
where student_source_id = 7
group by
  student_source_id,
  standard_category_source_id,
  standard_source_id
order by
  student_source_id,
  standard_category_source_id,
  standard_source_id
;
```


### Accordian

When expanded, the accordian shows the names of each of the lessons associated with the standard that is assigned to the student, the student's latest score, and how many attempts the student has made on that particular lesson.

__SQL__
Still working on excluding projected lessons that have been completed at least once.

```
select
    s.standard_category_source_id,
    s.standard_source_id,
    st.source_id as student_source_id,
    ls.lesson_source_id,
    l.attempts,
    r.lesson_score,
    case when r.lesson_score >= 0.7 then 1 else 0 end as passing,
    case when r.lesson_score < 0.7 then 1 else 0 end as failing,
    1 as assigned
  from
    (
      (
        select student_id, math_lesson_id,
          count(id) as attempts,
          max(id) as latest_lesson_completed
        from lesson_completed_facts
        group by student_id, math_lesson_id
      ) union (
        select p.student_id, m.id as math_lesson_id, 0 as attempts, null as latest_lesson_completed
        from projected_lessons_by_student_by_pathways p
                  inner join math_lesson_dimensions m
                    on p.lesson_source_id = m.lesson_source_id
                  left join (
                    select s.source_id as student_source_id, m.lesson_source_id
                    from
                    lesson_completed_facts lc
                      inner join student_dimensions s on lc.student_id = s.id
                      inner join math_lesson_dimensions m on lc.math_lesson_id = m.id
                  ) lessons_already_completed
                    on p.student_source_id = lessons_already_completed.student_source_id
                    and p.lesson_source_id = lessons_already_completed.lesson_source_id
        where
          p.student_source_id = 3731 and
          lessons_already_completed.student_source_id is null
      )
    ) l
    left join
    (
      select lesson_completed_facts.id as lesson_completed_id,
        case when lesson_result_dimensions.tested_out = true
           then lesson_completed_facts.pre_quiz_score
           else lesson_completed_facts.post_quiz_score
        end as lesson_score
      from lesson_completed_facts
        inner join lesson_result_dimensions
          on lesson_completed_facts.lesson_result_id = lesson_result_dimensions.id
    ) r
      on l.latest_lesson_completed = r.lesson_completed_id
    inner join (select id, lesson_source_id, unnest(standard_source_ids) as standard_source_id
          from math_lesson_dimensions) ls
      on l.math_lesson_id = ls.id
    inner join standard_dimensions s
      on ls.standard_source_id = s.standard_source_id
    inner join student_dimensions st
      on l.student_id = st.id
    JOIN (
            SELECT
              id as student_for_classroom_id
            FROM
              student_dimensions
            WHERE
              source_id in
              (
                SELECT
                  source_id
                FROM
                  student_dimensions
                WHERE
                  is_latest
              )
          ) student_classroom ON student_classroom.student_for_classroom_id = l.student_id
  where st.source_id = 7
  order by st.source_id,
    s.standard_category_source_id,
    s.standard_source_id,
    lesson_source_id
;
```
