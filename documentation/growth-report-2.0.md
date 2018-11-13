## Background

As a company, we are in business to help students move from their current math understanding back to grade level. We are asked frequently by our customers for information about how their students are growing in math proficiency. In order to provide a standard percentage baseline to report off of, a need exists to revise the original proof of concept.

## Model

Currently the system allows for teachers to dynamically assign students to pathways that TTM has not created, or are outside the bounds of the students current grade level. Directly correlated to this is the issue that once a student finishes a pathway they can be enrolled in another one, hereby erasing their growth metric.

Since TTM can only report growth off what their standard pathways, we are going to use those lessons as our base line denominator.

Taking the students performance grade level into consideration, we know the exact amount of TTM lessons that the placement exam will assign to a student base off their grade level deviation.

These numbers combined will be the final denominator.

<pre>
baseline = (TTM Pathway Lessons For Student Grade level and state ) + (TTM Placement Test Lessons For Student Deviation based on the default pathway)
</pre>

We will create a table that will store the required lessons for a student to achieve based off the default grade level pathway and their performance deviation. Each lesson that a student passes that is in this list, will count towards the students growth.

This will allow a teacher to have a solidified list of lessons to that TTM believes will encourage and help achieve math standards for the students grade.

Let's get an example

<pre>
Zack is a 7th grade student, performing at a 5th grade level after he took the placement test.

The TTM 7th grade standard pathway has a total of 30 lessons on it.
The placement test will add 10 lessons to the pathway.
Zack's total baseline lessons to complete is now 40.

30 + 10 = 40 = baseline

After passing the first 2 placement lessons, and failing the next 5.
His Teacher assigns him to a pathway that will allow him to focus on fractions and percents.
Assume this pathway has 10 lessons, 3 of which overlap with the 10 lessons required from the placement test.
Zack finishes this pathway.

He will now have 5 total lessons passed that apply to his growth.

2 from original pathway  +  3 from teacher created = 5 total lessons

(Completed Lessons in List) / (Baseline Denominator)  = 5/40 = 12% growth

At this point the student is re enrolled in their orignal pathway, and continues as planned.
Every lesson the student passes that has not previously been passed will continue to add to their growth total.
</pre>

## Next Steps
Include Standards into the report

## How to run a report
A report can be run for a school or a classroom. Start up rails console for what ever environment you want to run it on and execute the following commands.

<pre>
school = School.find([school_id])
ShardHelper.shard_context_for(school)
params = {school: school.id, from_date: Date.new(2012,8,1), to_date: Date.today }
report = GrowthReport.new(params)
x = report.export
File.open("/tmp/#{school.name}.csv", 'w') {|file| file.write x}
</pre>



