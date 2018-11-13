* [Current Year Ad Hoc Reporting Tool](http://warehouse.thinkthroughmath.com/adhoc/new)
* [SY2014-15 Ad Hoc Reporting Tool](http://warehouse-2014-2015.thinkthroughmath.com/adhoc/new)
* [SY2013-14 Ad Hoc Reporting Tool](http://warehouse-2013-2014.thinkthroughmath.com/adhoc/new)

You can also [access the 2012-2013 Warehouse](/operations/Accessing-The-2012-2013-Warehouse.md) if you really really really need to.


Note that the schemas and reports available between years will change, so a report that's runnable in the current year isn't necessarily backported to previous versions of the warehouse and adhoc tool.

## Achievement Report

### Running the Achievement Report
- Select "Achievement Report" from the Report Type dropdown.
- The report will be filtered by the dates provided *or* if either field is left empty, it will filter by the start and end dates of the current school year (08/01 - 07/30 range that contains the date the report is being run). 
- You can filter by Classroom, School, District, Customer, or State. Choose which level you would like to use for your filter, and start typing - the filter should auto complete, and when you press enter or select an option, the filter is added. Filters can be removed by clicking the associated 'Remove' button.
- Once you've set up the report filters as you would like them, click 'Run Query'.
- When the report is ready, a green 'Download Results' button will appear. Click on this button to get a csv file of the Achievement Report.

### The Achievement Report Described

The Achievement Report has a row for each student with these columns:

- customer_name
- district_name
- school_name
- classroom_name
- student_id
- student_information_number
- student_name
- grade_level
- performance_grade_level
- columns for each month in the specified date range 
  - each of these columns shows the percentage of lessons that are on the student's *default* pathway that have been passed up to that month (but only counts passed lessons that are within the specified date range of the report - so lessons passed prior to the date range of the report will not be included).


For example, suppose that Sam has a default pathway with 10 lessons. As those lessons are passed, the percentages will go up (regardless of how those default pathway lessons are passed - it doesn't matter if they are on the pathway that Sam is currently on or if they are passed as remediation lessons or any other way). Here's a simplified version of the report, supposing that Sam passed 2 lessons in January and 3 in February:

| Student | 12-2012 | 01-2013 | 02-2013 | 03-2013 |
|:-------:|--------:|--------:|--------:|--------:|
| Sam     | 0%      | 20%     | 50%     | 50%     |
