### Customer Account Setup in LMS

1. Customer
    * Used to designate a 'higher education institution'
         * Each higher ed institution will be a separate customer in the LMS
              * i.e. University of Pittsburgh, University of Phoenix, Community College of Allegheny County
         * Each customer should be marked "Higher Ed" via the checkbox on the customer creation/edit page
         * State should indicate the state in which the institution operates
              * If a higher ed institution operates in multiple states, select the state in which the institution considers its primary campus, or where its headquarters or main administrative office is located
    * In Customer Settings, each customer should be marked "Require student records to have student information number" via the check box
         * NOTE: The following settings are optional and not recommended to be set unless the Sales, Customer Support or the customer feels strongly that they should be enforced
              * Minimum length of student information number
              * Restrict student information number to numeric values
       * Bear in mind that if there is a Customer Admin role established for the customer, the Customer Admin will also be able to access and manage these settings self-service via "Customer Settings" on their dashboard
1. District
    * Will not be used at this time
1. School
    * Used to designate 'campus'
        * If an institution has only one campus, it will only have one school in the LMS
             * This school can be named 'main campus' or named after the town in which the main campus resides
        * If an institution has more than one campus, each school in the LMS should be a separate campus
             * i.e. if 'Community College of Allegheny County' were a customer, it would have the following schools established 'underneath' it: Allegheny Campus, North Campus, Boyce Campus, South Campus
1. Classroom
    * Used to designate 'course'
    * Classroom name should reflect the naming conventions of the institution
        * Should include (minimally) course ID, section, instructor, a unique identifier (if possible), time frame or semester of the course

### Higher Ed Pathway

At this time there is exactly one TTM-created higher ed pathway. All higher ed
customers should use this pathway. On production it’s ID is: **332319**.

A higher ed pathway differs from a regular pathway in a couple ways. Higher Ed
pathways have a final lesson and do not create deferments for lessons or
precursors (the result of “remediation”). The final lesson should always be
placed at the end of the pathway and is pass/fail--a user cannot take it more
than once. When the final lesson has been completed, the pathway is also
considered completed.

Note: Higher Ed customers are not blocked from creating a pathway, but this is
not a feature we are actively supporting at this time.

### Student Enrollment

Use the `ttm:higher_ed:create` rake task to create students for a higher ed
institution and course. Operations are wrapped in a transaction, so if
something goes wrong, the system will not be left in an inconsistent state.
Refer to the console output to verify the request been successfully completed.

* *PATHWAY_IDS (optional)* - if no pathway is given, no enrollments will be created
* *FIRST_NAME (optional)* - Without this option, the set First Name defaults to classroom.name
* *LAST_NAME (optional)* - Without this option, the set Last Name defaults to classroom.customer.name
* *COUNT (optional)* - Defaults to one student

#### Execution

SSH into production and run the rake task with the params that fit your case

For example, to populate three specific classrooms (1, 2 and 3) with five
students who are enrolled in a specific pathway (332319), run the following
commands:

```bash
$ ttmscalr ssh debug.1 -f prod
$ cd /var/www
$ CLASS_IDS=1,2,3 PATHWAY_IDS=332319 COUNT=5 rake ttm:higher_ed:create
```

For example, to populate one specific classroom (4) with one student without
any enrollments, run the following commands:

```bash
$ ttmscalr ssh debug.1 -f prod
$ cd /var/www
$ CLASS_IDS=4 rake ttm:higher_ed:create
```

### Student Deactivation

Student accounts will be deactivated 60 days from the day they were created by
another rake task, `ttm:higher_ed:deactivate_student_accounts`, which is
automatically run every night. As a result, a user should never need to run
this script manually. **Please don’t run this script manually**. If there is a
problem, please see *Tim* or *Andre* to escalate the concern.
