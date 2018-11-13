# Data Model Limitations

This document describes some limitations in TTM's data model that might
complicate integrations with other system.

## Globally unique student usernames

Student usernames in TTM must be globally unique. So, if TTM must accept student usernames
from a 3rd party system, they may conflict with a user in a completely different district.

We currently work around this in one of two ways:

  - Adding random numbers as a suffix to the student's "raw" username.

  - Adding a unique site-code prefix to the student username. All clever customers in TTM require a username prefix.

Neither of these solutions is ideal because it means the student will have a different username in TTM than other
applications within their district. In addition, it makes (non-Clever) SSO significantly more complicated because
we have no way to match an inbound SSO request with a specific student population.

We have plans to address this by scoping username uniqueness to within a district,
but development work towards that has not yet begun. As of November 2016, this work as not begun.

## One email per user role/school

A single user (non-student) can only perform a single role within TTM. If a single person administers multiple
schools or districts within TTM, they need a different unique email address for each of those roles. Many users
use a "fake" email address for their various roles which means they are unable to receive pushed reports or
notifications via email for those entities.

We have plans to address this by either presenting the user an option when they log in which of their roles/schools
they'd like to administer, or by aggregating all of their reports and data to the appropriate entities that they have
permission to administer. As of November 2016, this work as not begun.

## Students can only be in one school

A single TTM student account can only belong to a single school. Trying to accept 3rd party data that associates a
student with more than one school will result in an error being generated.

When integrating with 3rd parties, we may need to have per-integration or per-site rules that decide which of a
student's many schools TTM will accept.

## Student work is tied to a classroom

All work done by a student is associated with a classroom. This is used in reports to correctly aggregate student work
at the classroom level. In other words, if a student is in two classrooms and has done work in each classroom, the work
done in each classroom will be split appropriately between the classrooms when viewing reports at the classroom level.

In addition, work being tied to a classroom has allowed TTM to set per-classroom restrictions on our Classroom Goal
motivation program.

When accepting data from a 3rd party, a student will be automatically enrolled in the first classroom specified by the 
integration. This results in work done by that student being reflected in that classroom even if it's not the "correct"
classroom to house that work. For this reason, during BTS 2016, we asked districts to try and include students only in a
single classroom for their initial sync so that work would appear in the correct classroom. This caused significant pain
on our end-users who provide us with their student roster data.

We have plans to decouple student work from classrooms because it has caused confusion as to why certain student work is
being attributed to a specific classroom. As of November 2016, this work as not begun.
