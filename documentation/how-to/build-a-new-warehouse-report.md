> I realized while writing this that I always build them kind of backwards, from the controller down. But all the dependencies go the other way, so it's probably better to start at the query level and build up from there. For example, when building the 'Pass Rate by Grade' report, [my first commit](https://github.com/thinkthroughmath/reporting/commit/46853992487cce9d70b31e6cfedc388bc9ee2d5c) included the Endpoint, Report Thing, and an empty query. After that commit, you could browse to `localhost:3000/pass_rate_by_grade` and get an empty version of the report. But it means that the first commit has to have a stubbed out blank query to work.

### Building a New Warehouse Report

#### Query

Most of the complexity of writing a warehouse report is in the query. This is where all of the work is done to take the parameters passed in by the user and build a sql statement that will return the correct data.

> ##### More query things to write about:
>
>  - Main Query / Query Builder
>  - [Level](https://github.com/thinkthroughmath/reporting/blob/rc/app/reports/report_levels.rb) vs [Group](https://github.com/thinkthroughmath/reporting/blob/rc/app/queries/groups.rb)
>    - Groups group the subqueries used in the report. Should group on ids only.
>    - Level is the grouping of the top level of the report query.

#### Report Thing

##### Report

The report is responsible for cleaning up parameters (using the ParamParser), and basically managing the pieces that the ReportExecutor puts together for the final product (the query, the column set, any paging or order_by info). These are generally pretty simple and we could probably DRY these up a bit if we wanted to (like the controllers).

Each of these defines an `empty?` method, which basically lists any required parameters. If this method returns true, the ReportExecutor returns a report with no data in it.

```ruby
# E.G. - reports/pass_rate_by_grade_report.rb

class PassRateByGradeReport
  attr_reader :query, :column_set

  def initialize(params)
    params[:group] = Groups::None.new
    @params     = ParamParser.new(params).parse
    @column_set = PassRateByGradeColumnSet.new
    @query      = PassRateByGradeQuery.new(@params)
  end

  def empty?
    @params[:student].nil?
  end
end
```

##### Column Set

Each report has a column set, which is used by the report executor to map the data to the columns that should display in the final result (using the `columns` method). Generally, we do our number formatting and any cleanup of data that should happen between the database and the JSON that is returned by the warehouse here. In this example, we use ReportHelper methods to format our ints and mean_percents. We generally calculate means at this point so that we have consistent rounding and formatting, and when we write our supporting queries we only calculate the sum and count values used for the mean calculation.

```ruby
# E.G. - reports/pass_rate_by_grade_column_set.rb

class PassRateByGradeColumnSet
  include ReportHelper

  def build_row(row)
    {
        grade_level:      int(row[:adjusted_grade_level]),
        target_pass_rate: mean_percent(row[:passed_count], row[:lesson_count])
    }
  end

  def columns
    build_row({}).keys
     .map { |k| { name: k.to_s } }
     .to_a
  end
end
```

##### Specs

So far, we've mostly been writing integration tests at the report level -- setting up some fake data, passing in parameters and checking the result, like [this](https://github.com/thinkthroughmath/reporting/blob/rc/spec/reports/standards_report_spec.rb).
[Here's](https://github.com/thinkthroughmath/reporting/blob/rc/spec/reports/standards_report_bigger_data_spec.rb) an example with some generated data.

#### Endpoint

Would hope at some point to change this so we have just one controller for all of the reports. The only thing that changes right now is the __report__.

Each controller instantiates a report with the passed in params, runs it with the `ReportExecutor`, and returns the results.

##### Controller

```ruby
# E.G. - controllers/pass_rate_by_grade_controller.rb

class PassRateByGradeController < ApplicationController
  def index
    report  = PassRateByGradeReport.new(params)
    results = ReportExecutor.new(report).run
    render json: results
  end
end

```

##### Add a route

```ruby
# E.G. - in routes.rb

resources :pass_rate_by_grade, only: [:index]
```

### Shared Components Used for Building Warehouse Reports

#### About the ParamParser

https://github.com/thinkthroughmath/reporting/blob/rc/app/reports/param_parser.rb

Any parameter used in a warehouse query should have a line in the `parse` method of the ParamParser. This just sanitizes our inputs and makes sure we are getting the parameters we expect.

#### About the QueryBuilder

https://github.com/thinkthroughmath/reporting/blob/rc/app/queries/query_builder.rb

This is a general class responsible for building the top level query used for report from the warehouse. It gets the information needed to build a query (including parameters, ordering info, paging info) and puts together a paged query using the driver query defined by the report level. This class basically puts all that stuff together and gives you back a big sql statement for the report.

The `Report` gives the QueryBuilder the following:
  - params
  - level
  - sortables
  - limit
  - subqueries
  - order_by
  - sort
  - common_table_expressions
  - min_lessons

And the QueryBuilder also does some manipulation of the query using values from the `Level`:
  - additional_queries
  - where_clause
  - order_by

This is terribly confusing and we can probably simplify this area a little bit.

#### About the ReportExecutor

https://github.com/thinkthroughmath/reporting/blob/rc/app/reports/report_executor.rb

1. Runs the query and gets the results back as an array
2. Uses the `ColumnSet` to map the raw data to the columns to display (including formatting)
3. Runs any transforms defined on the report to update the formatted data
4. Uses a pager to display total pages and current page information if the report is paged
5. Uses an order_by to display ordering information if the report is ordered
5. Returns JSON defining the columns, rows, paging info, and order by info
