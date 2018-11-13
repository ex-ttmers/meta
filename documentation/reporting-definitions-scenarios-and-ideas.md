This [Reporting Scenarios](https://docs.google.com/spreadsheet/ccc?key=0Au4IbUR065QMdEliWGx0ck1qVnR0aW9WQmpoQUZSZkE&usp=drive_web#gid=0) spreadsheet outlines a number of scenarios regarding deactivation, moves, and date range queries to explicitly show what should happen to the number of lessons shown and numbers of started and enrolled students at various levels of aggregation.

This [Reporting Ecosystem](https://docs.google.com/a/thinkthroughmath.com/spreadsheets/d/1sm-vZYKS6pXhI6lto2gPOiw6orNc06oUdV9oDUmdF_o/edit?usp=sharing) spreadsheet lists each of the reports in our system, what fields they show, who can see it and where they come from. 

Here are some more general definitions of reporting fields based on discussions with various folks about what these fields should mean and our conversations afterwards as we lined those up with the scenarios we had defined:

# Started / Enrolled Students

- For all purposes in TTM, a student is a person, and therefore exists once in roll-up reporting.
- A student can only _enroll_ or _start_ a single time each school year. It is a one-way switch.
- Enrolled is a YES/NO binary. It happens once a school year and does not reverse. Once you were enrolled, you were enrolled.
- Started is a YES/NO binary. It happens once a school year and does not reverse. Once you were started, you were started.

# "Started"

- A student becomes **started** as soon as they take a "math action" in the system.
  - Start a placement test
  - Start a lesson (if placement test is skipped)
  - Logging in is NOT significant enough to trigger "started"
  - Completing a lesson is not required for "started" 
- We can define started as any student that has made an attempt on either a placement test or lesson enrollment.
- "Started" counts are at the student level and not at the level of work done. Therefore, this count makes sense to show at the school level or higher, but not the classroom level (at the classroom level, we will instead show work done in that classroom).

# Live Helps

When students request live help they may or may not be actually picked up by a teacher. In both cases the sessions are added to the  `live_help_facts` dimension. Sessions that are not picked up have no associated teacher_id, and the `engaged_date`, `engaged_time`, `completed_date` and `completed_time` will be the same as the `started_date` and `started_time` for the rows. It's important to note that aggregate tables in the warehouse only track sessions that are picked up, not requested. See [this part](https://github.com/thinkthroughmath/reporting/blob/master/app/models/student_daily_aggregate.rb#L263) of the code to verify.

