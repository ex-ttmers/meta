For each __standard__ in a __standard category/domain__, the __standard summary__ shows:

- number of students _failing_: average score of lessons completed within the standard is _39% or lower_
- number of students _struggling_: average score of lessons completed within the standard is _between 40% and 69%_
- number of students _passing_: average score of lessons completed within the standard is _70% or above_
- number of students _assigned_


The __standard summary__ can be filtered by:

- standard category
- classroom
- student's grade level

__Draft SQL__

```
select
  standard_category_source_id,
  standard_source_id,
  sum(passing) as passing_students,
  sum(struggling) as struggling_students,
  sum(failing) as failing_students,
  sum(assigned) as assigned_students
from (
  select
    s.standard_category_source_id,
    s.standard_source_id,
    st.source_id as student_source_id,
    avg(r.lesson_score) as average_score_for_standard_category,
    case when avg(r.lesson_score) >= 0.7 then 1 else 0 end as passing,
    case when avg(r.lesson_score) >= 0.4 and avg(r.lesson_score) < 0.7 then 1 else 0 end as struggling,
    case when avg(r.lesson_score) < 0.4 then 1 else 0 end as failing,
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
group by
  standard_category_source_id,
  standard_source_id;
```

### Accordian

When expanded, the accordian shows the names of each of the students that are passing, failing, or struggling with the standard.

Basically the inner query of the above will work for this, filtered the same way plus by standard.

