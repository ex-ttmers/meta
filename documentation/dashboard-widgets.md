An astonishing percentage of our reports are implemented using the dashboard widget framework. Even things like the overview report, which don't display on the dashboard, use this framework. There's a [Markdown document](https://github.com/thinkthroughmath/apangea/blob/rc/app/assets/javascripts/reporting/DASHBOARD_WIDGETS.md) in git that describes the workflow since it can be complex for someone new to the app.


### Snapshot of the file (This might be out of date but no reason to click around unless you have to)

# Dashboard Widgets

## Ruby/Rails

Since rails mediates all requests to the DW for end-user reports the initializing and building of the reports is directed via post from the client side to rails on page load complete

#### lib/reporting/report.rb

Concerns here are with the initializing of builders and decorators for the apangea side of the widget process. A dashboard widget looks like this:

      def initialize(id, name = nil, decorator = nil, cache = nil)

This initializer creates an instance for every active report. The instance sits between the browser (which requests the widget) and the DW (which fills the widget request).

    ReportContainer.instance.add(
    'overview_report'          => Report.new('overview_reports', 'overview_reports',
      ->(r) { OverviewReportEntryDecorator.new(r).decorate }),

    'started_students'         => Report.new('started_students'),
    'think_30_scoreboard'      => Report.new('think_30_scoreboard'),
    'overall_performance'      => Report.new('overall_performance', 'overall_performance',
      ->(r) { LeaderboardEntryDecorator.new(r).decorate }),

    'lesson_leaderboard'       => Report.new('lesson_leaders', 'lesson_leaders',
      ->(r) { LeaderboardEntryDecorator.new(r).decorate }),

    'point_leaderboard'        => Report::PointLeaderboardReport.new,
    'lesson_leaders_by_class'  => Report.new('lesson_leaders_by_class', 'lesson_leaders'),
    'lesson_leaders_by_school' => Report.new('lesson_leaders_by_school', 'lesson_leaders'),
    'lesson_leaders_by_state'  => Report.new('lesson_leaders_by_state', 'lesson_leaders'),
    'charities'                => Report.new('charities'),
    'classroom_goals'          => Report.new('classroom_giving', 'classroom_giving',
      ->(r) { ClassroomGoalsDecorator.new(r).decorate }),
    'classroom_aggregate'     => Report.new('classroom_aggregate', 'classroom_aggregate',
      ->(r) { ClassroomAggDecorator.new(r).decorate })
    )

#### app/controllers/widgets_controller.rb

The controller takes a POST request for widgets to initialize and passes it on to the ReportResolver

#### lib/reporting/report_resolver.rb

This pulls the instance of the report from Report and builds a report object to return to the controller

#### lib/reporting/report_decorators.rb

Here's where you configure a report as well as define the columns that are valid to export. It gives you a hook into the reporting route so you can create a new report at /reports/<your_report> and setup and rails side configs for pre clientside configuration.

#### lib/reporting/reporting_gateway.rb

This is just the class that makes the real request to generate the report. If for some reason you need to see how that happens there it is.

## Coffee
### Assuming these are all under app/assets/javascripts

A JS namespace has been created for the dashboard widgets to be mixed into.

    window.ttm.dashboard

A sample of what a client side call to the DW looks like

     window.ttm.dashboard.Report.build('overview_report', 'district');

This particular call skips the resolution process and returns a report object for the single named report. From here a fetcher can be extracted and fetch can be run manually to test the call the DW is valid.

    a =  window.ttm.dashboard.Report.build('overview_report', 'district', true);
    b = a.buildFetcher();
    b.fetch();

If you checkout the network tab in chrome you will now see the requests routed through apangea and in the end the data returned from the DW query.

##### dashboard/report.coffee

This is the client side parallel to dashboard_widget.rb on the rails side. It contains an array of report instances ready to be initialized. New widgets/reports must be added to the widgets object. The key will be how the resolver looks up your report. The key of the hash represented by this object will be the name of the report used when making a DW request via apangea. It is a good practice for this to match the name property of your widget but it is not required. There is a caveat in regards to your widgets ability to identify when its widget data is returned although.

    WIDGETS = {
      classroom_aggregate: window.ttm.dashboard.ClassroomAggReportWidget,
      overview_report: window.ttm.dashboard.OverviewReportWidget,
      lessons: window.ttm.dashboard.LessonLeaderboardWidget,
      points: window.ttm.dashboard.PointLeaderboardWidget,
      think_30_scoreboard: window.ttm.dashboard.Think30ScoreboardReportWidget,
      overall_performance: window.ttm.dashboard.OverallPerformanceReportWidget,
      charities: window.ttm.dashboard.CharitiesReportWidget,
      classroom_goals: window.ttm.dashboard.ClassroomGoalsReportWidget,
    }

#### dashboard/widget.coffee

A widget is a series of callbacks that maintain state for requesting decorating rendering and interacting with DW report data. While this is not the parent to an actual report widget it does interact by setting up a specific widget and allowing it to inject its own activities during this workflow.

#### dashboard/hashchange.coffee

There are a lot of callbacks that are associated with changing the parameter hash in the url. We use a jQuery plugin called [bbq](http://benalman.com/projects/jquery-bbq-plugin/) which helps our interaction with the back button and query strings. You will notice that the url does not contain a standard ?(query) parameter list but begins with an anchor hash with a parameter list. The plugin lets us manage the state of this hash string and makes requests on our behalf. When a widget is loaded it registers its hashchange setup which in itself triggers the hashchange event on loading. __This is how the fetcher is initialized by default for widgets__.

#### <some>_report_widget.coffee

Lower down the rendering chanin the widget.coffee assumes the functionality of you specific report and provides the fetchers data so it can be decorated on the resolution of the DW call.

## Making the report

The workflow here is a little hard but here is a good way to look at it from the overview report.

Report specific callbacks should be defined in the initializer for your specific report like so:

    @onComplete    = window.ttm.dashboard.spinUpTable
    @decorator     = window.ttm.decorators.OverviewReportDecorator
    @styling       = window.ttm.dashboard.Styling.build()
    @linkFormatter = window.ttm.dashboard.DrillLinkFormatter.build @group
    @teacherFormatter = window.ttm.dashboard.TeacherFormatter
    @optionalFilters = window.ttm.dashboard.OptionalFilters.build('dashboard/overview_report_header')
    @hashchange    = widgetParams.hashchange

With those callbacks in place you can thread them into the __defaultParams__ and __formatResults__.

Default params can look a little something like this:

    @defaultParams = {
      name: 'overview_report'
      container: '#overview-report'
      template:  template
      formatResults: @formatResults
      afterRender: (results) =>
        @styling.decorateTables()
        @optionalFilters.build(results)
        #@hashchange.wire_column_sort_after()
        @onComplete() if @onComplete
    }

Format Results is where the widget rows is converted to a handlebars consumable object.

Some sample data from the DW
    {"widgets":[
      {"widget":"classroom_overview_reports",
       "results":{"columns":[
          {"name":"id"},
          {"name":"name"},
          {"name":"started_students"},
          {"name":"average_lessons_tested_out"},
          {"name":"prequiz_average"},
          {"name":"postquiz_average"},
          {"name":"teachers"},
          {"name":"district_id"},
          {"name":"district_name"},
          {"name":"school_id"},
          {"name":"school_name"}],
      "rows":[{
        "id":140208,
        "name":"magic class",
        "started_students":0,
        "average_lessons_tested_out":"--",
        "prequiz_average":"--",
        "postquiz_average":"--",
        "teachers":"Teacher Not Assigned",
        "district_id":"1271",
        "district_name":"Beacon Learning Centers",
        "school_id":"2404",
        "school_name":"Beacon - Central Middle School"}],
      "paging":{},"order_by":{}}}]}

A sample formatter iteration pattern

The results param passed to format results

    rows = _.map results, (r) =>
      r.report_id       = @defaultParams.name
      r.type            = @group
      r.type_name       = @group
      r.linkData        = @linkFormatter.toLinkData r
      r.show_link       = true
      r.teacher_column  = @teacher_column()
      values = _.map cols, (c) =>
        r[c]

      r.values = values
      r

A sample results set from the overview report

    rs =
      {
       has_data: rows.length > 0
       cols: cols
       rows: rows
       paging: paging
       type: rows[0].type if rows.length > 0
       sort: @sortCriteria
       type_name: @group[0].toUpperCase() + @group[1..-1].toLowerCase()
       teacher_column: @teacher_column()
       order_by: order_by.order_by if order_by
       sort: order_by.sort if order_by
      }

Template configuration is provided by the template defaultParam and should represent a slimbars (slim handlebars) template in the dashboard/ directory.


## What happens when we load the report
1. Report.coffee finds your specific widget and starts loading its default params.
2. Next comes the wireup and fetching process.
   1. First hashchange is wired up
   2. Hashchange registers its hashchange event
   3. The hashchange event is triggered with the side effect of beginning the widget fetch process
3. Render is called which polls until the fetcher is completed and passes the results to mustache
4. The template is rendered


## What happens when the fetcher starts
1. A request is posted to apangea which starts a sidekiq worker which posts a request with the __id__ from the WIDGET configured in report.coffee to a request with this pattern.
    From __reporting_gateway.rb__

        HTTParty.get("#{REPORTING_URL}/#{report}", query: params)

    This request is started manually with the render method of a report widget.
    1. render starts the hashchange registration process which invokes refresh
    2. refresh calls down to the widget to refresh and then rebuilds itself
    3. refresh builds the widgets fetcher and invokes the fetch

2. This is where things get hairy.
   1. The widget fetcher builds a widget collection around your widget no matter the number of them.
   2. fetcher begins polling
   3. Each round displayandfetch is called which once again continues to call displayandfetch until all widgets have data
   4. Once that is the case WidgetCollection display is called and render from widget.coff is called for each widget.
3. Now that data is returned and widget.coffee render is called formatResults for your specific widget is invoked.
4. Then afterRender is called.
5. At this point the element on the page is replaced with the template with the formatted data.
