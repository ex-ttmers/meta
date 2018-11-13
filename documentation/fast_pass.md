### Fast Pass

#### What the heck is fast pass?

Fast pass is a feature that we added that allows sales demos to connect to live teachers without waiting in line.  The way it is currently implemented, we have a hard coded a list of customer id's in which sales has student accounts for demoing.

Fast pass students are **NOT** students with `is_demo == true` in the DB or customers with `is_demo == true` in the DB. These are completely spararate designations.

#### Where is it configured?

https://github.com/thinkthroughmath/apangea/blob/rc/lib/live_teaching_enrollment_item.rb#L20

### How to test it?

__For the Live Teacher__

1.  Log in as any live teacher

2.  Launch PZ

__For the Student/Salesperson__

1.  Find a customer id's listed in the `SALES_CUSTOMERS` array, currently defined in [lib/live_teaching_enrollment_item](https://github.com/thinkthroughmath/apangea/blob/rc/lib/live_teaching_enrollment_item.rb).

2.  Login as or impersonate a user from within that customer.  Teachers and admins should work fine.

3.  Navigate to Content/Preview an Activity

4.  Pick a Guided Learning activity

5.  In a different window, login as or impersonate a student not in one of the fastpass customers.  Find a guided learning, get the item wrong, and request live help.

6.  Launch the Guided Learning for the fastpass student.

7. Now,
  - if the teacher is "Demo Ready", and they click that 'Add Student' button, they will get the fastpass student first even though the non-fastpass student has been waiting longer.
  - if the teacher is **NOT** "Demo Ready", and they click that 'Add Student' button, they will get the non-fastpass student first.
  - if the teacher is **NOT** "Demo Ready", and they click that 'Add Student' button, **AND** the time limit (currently 10 seconds, set here: https://github.com/thinkthroughmath/live_teaching/blob/rc/config/default.js#L31) has surpassed, they will get the fastpass student first.
