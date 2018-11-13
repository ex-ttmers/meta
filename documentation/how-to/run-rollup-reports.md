# Run Rollup Reports

**You will need to have Apangea and Reporting running**

Locally:
* In the CMS impersonate a school official.(Teacher/SchoolAdmin)
* Go to the Reporting tab
* Under the Weekly Reports block click Change Settings
* On the right hand side there will be a Report Opt-in slider
* Now in Rails Console run:  UserOptedInRollupReport.update_all(next_run_date: 7.days.ago)
* Out of rails
* rake ttm:enqueue_rollup_reports_for_generation['now']
* rake ttm:compile_and_email_rollup_reports
* Refresh the page
* For Teachers you should see Student Info Report and Class Level Summary Report
* For School Admins you will get an extra report: Grade Level Summary
* Rollup reports don't get sent to users with email like @example.com. If you are running locally and have a user like this, you need to remove `.where("users.email !~*'@example.com'")` from UserOptedInRollupReport#reports_scheduled_for(date_

Just the commands Mailcatcher optional:
* $`gem install mailcatcher` Optional
* $`open http://127.0.0.1:1080` Optional
* rails c `Teacher.find([ID]).opt_in_to_rollup_reports` make sure their email is not @example.com because we filter on that
* rails c `UserOptedInRollupReport.update_all(next_run_date: 7.days.ago)`
* apangea$`foreman s -f Procfile.development` you need redis so just run apangea
* reporting$`foreman s -f Procfile.Development` you need reporting to generate the classroom overview
* $`rake ttm:enqueue_rollup_reports_for_generation['now']`
* $`rake ttm:compile_and_email_rollup_reports`
