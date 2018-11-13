### What is the Standards Report?

The Standards Report is being developed as a way for teachers to see at a glance how well their students are progressing on the standards that they are going to be tested on.

For planned user experience, as well as explicit documentation for how the report should behave under specific scenarios, please see the [reqs and specs](https://drive.google.com/a/thinkthroughmath.com/?urp=https://drive.google.com/?authuser%3D1#folders/0B6bKks88JmyiMDNEN1ZOOHJPd0E).


#### Some Relevant Entities and Definitions 

* **Standard** _::ToDo::_
* **Standard Category** _::ToDo::_
* **Pathway**

	A pathway is a list of lessons that a student is assigned (via an _enrollment_). They are grade level specific. This doesn't necessarily match up to the lessons that a student actually takes, because it doesn't take into account remediation lessons (or retakes). We can calculate which remedial lessons a student will encounter based on their placement test (this depends on the student's performance grade level); but this does not include any remedial lessons added because a student failed a lesson.
	
	These options are set per pathway (these are not all of them, just relevant):
	* use placement test - Students take a placement test which determines their performance grade level before any other lessons are presented to them.
	* skip problem solving process
	* allow test out
	* require pre quiz
	* use remediation - If this option is true, when a student fails a lesson all lesson prerequisites will be added to the student's lesson enrollments. Otherwise, failed lessons are repeated once and then if they are failed a 2nd time, they are deferred (placed at the end of the pathway). Students only have one chance to pass a deferred lesson - they won't see it again (so they can see a lesson a maximum of 3 times). 
		
* **Grade Level / Performance Grade Level**

	Students are assigned a grade level when they are created, which will generally match the grade level of the pathways they are enrolled in. However, they can be placed at a different grade level based on the results of a placement test. This is the student's _performance grade level_. The performance grade level determines if remedial lessons (prerequisites starting from their placement grade level up to the lessons that are on their enrolled pathway) are presented to them before they get to the pathway lessons that are at the grade level the pathway is assigned.
		
* **Lesson**

	Each lesson can have:
	* standards
	* prerequisites (sometimes called precursors)
	* activities

	* **Projected** 
	
		The `EnrollmentLessonCalculator` calculates the student's next lesson when they complete their previous one. This means that although the Pathway generally describes the Lessons that the Student will encounter, it's not exact. The Pathway cannot know if a student will fail a lesson and require additional remediation lessons to be added. Also, the Pathway may change - Lessons can be added. Since the next Lesson presented to the student is calculated just in time, if a new Lesson is inserted to the Pathway prior to where the student is currently on that Pathway, the student will never see it (of course, if the portion of a Pathway that the student has not yet completed is modified, the student _will_ see those changes to the pathway).

		So we are doing the best we can - we will use the lessons that the Pathway contains for the student's performance grade level as "projected" lessons. In addition, any lessons added for remediation will be included as a "projected" lesson. The method `EnrollmentLessonCalculator.project` will give us all of these, plus lessons that have been added as retakes. We will persist this information any time a pathway changes or a student completes a lesson and this needs to be recalculated.
		
	* **Completed _as related to a standard or standard category_** _::ToDo::_
	* **Passing _as related to a standard or standard category_** _::ToDo::_
	* **Struggling _as related to a standard or standard category_** _::ToDo::_
	* **Failing _as related to a standard or standard category_** _::ToDo::_

#### Technical Design

Data for the Standards Report will come from the Data Warehouse.

##### Standards

A `StandardsDimension` will need to be added to the Data Warehouse.

```
                                           Table "public.standard_dimensions"
           Column            |          Type          |                            Modifiers
-----------------------------+------------------------+------------------------------------------------------------------
 id                          | integer                | not null default nextval('standard_dimensions_id_seq'::regclass)
 standard_source_id          | integer                |
 standard_category_source_id | integer                |
 standard_category           | character varying(600) |
 standard_code               | character varying(255) |
 standard_details            | text                   |
 state_code                  | character varying(255) |
 state_name                  | character varying(255) |
 math_lesson_source_ids      | integer[]              |
 effective_start             | date                   | not null default '2012-01-01'::date
 effective_end               | date                   | not null default '2100-01-01'::date
 is_latest                   | boolean                | not null default true
Indexes:
    "standard_dimensions_pkey" PRIMARY KEY, btree (id)
    "index_standard_dimensions_on_math_lesson_source_ids" gin (math_lesson_source_ids)
    "index_standard_dimensions_on_standard_category_source_id" btree (standard_category_source_id)
    "index_standard_dimensions_on_standard_source_id" btree (standard_source_id)
    "index_standard_dimensions_on_state_code" btree (state_code)
```

We will add Standard source ids to the `MathLessonDimension` table:

```
                                       Table "public.math_lesson_dimensions"
       Column        |          Type          |                              Modifiers
---------------------+------------------------+---------------------------------------------------------------------
 standard_source_ids | integer[]              |
Indexes:
    "index_math_lesson_dimensions_on_lesson_source_id" btree (lesson_source_id)
```
 _::ToDo::_
 - ETL
 - Report Queries
 - UI
 
##### Report Views
- [[Standards Report Domain Summary]]
- [[Standards Report Standards Summary]]
- [[Standards Report Individual Student Standards]]
 
<img src="standards-report-query.png" height=800px/>
 

#### MVP and Other Planned Functionality

Initially, this report only needs to be available to teachers - so Classroom is the highest filter level we need to worry about. Eventually this report should be available for teachers and admins (and parents?) so the filtering will become more complex.


### Old Standards Report

<img src="old-standards-report.png" height=200px />

#### Definitions
* **Passed All** _::ToDo::_
* **Passed Some** _::ToDo::_
* **Needs More Work** _::ToDo::_
* **Not Attempted** _::ToDo::_

#### Technical Design

The old Standards Report did not use any data from the Data Warehouse.
 _::ToDo::_
