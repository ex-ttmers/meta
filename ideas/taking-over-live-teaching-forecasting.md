Our needs for staffing for live teaching are no different than a call center, a hospital, or a restaurant. In each case the actors are the same; there are customers (students, patrons, patients, etc), agents (servers, nurses/doctors, teachers, etc), there is a finite number of service connections (slots, beds, tables, etc). Forecasting requires knowing the duration of each interaction as well. Armed with estimation for the demand and the average interaction, there are a number of well-established models that can be used for forecasting.

The most commonly used in call centers is called [Erlang C](https://en.wikipedia.org/wiki/Erlang_distribution) - it is based on early research in the telecommunications industry. A number of software packages are available that allow you to use custom historical data along with this model to predict staffing. There's a [basic model in Excel](https://www.lokad.com/GetFile.aspx?File=Support%2fCallCenterTutorial%2ferlang-by-lokad.xls), just to play around with the concept.

[Agenses Forecast](http://www.agenses.com/358/Weekly_forecast_in_30_minutes/) seems like a particularly promising tool for forecasting demand. It can use historical call data to predict staffing needs, down to the 15 minute increment. It can import data in the [Call Detail Record (CDR) format](https://en.wikipedia.org/wiki/Call_detail_record) from most common Call Center phone systems.

Currently, TTM's live teaching data recording doesn't match any specific format, but the CDR format for [Asterisk](http://www.asterisk.org/), an open source Phone Bridge Exchange system, should be easy to map to the data we do track. The Asterisk CSV format is here](http://www.voip-info.org/wiki/view/Asterisk+cdr+csv). A mapping might look like this, mapped against the available warehouse data:

* accountcode: What account number to use: Asterisk billing account, (string, 20 characters) - Map to TTM `customer_id`
* src: Caller*ID number (string, 80 characters) - Map to TTM `student_id`
* dst: Destination extension (string, 80 characters) - Map to TTM live teacher `user_id`
* dcontext: Destination context (string, 80 characters) - OPTIONAL map to TTM live teacher `email` (for ease of use)
* clid: Caller*ID with text (80 characters) - Map to TTM `live_help_facts` record id
* channel: Channel used (80 characters) - OPTIONAL Map to TTM student user agent
* dstchannel: Destination channel if appropriate (80 characters) - OPTIONAL Map to TTM live teacher user agent
* lastapp: Last application if appropriate (80 characters)
* lastdata: Last application data (arguments) (80 characters)
* start: Start of call (date/time) - Map to TTM `utc_started` datestamp (in whatever date/time format we need)
* answer: Answer of call (date/time) - Map to TTM `utc_engaged` datestamp (in whatever date/time format we need)
* end: End of call (date/time) - Map to TTM `utc_completed` datestamp (in whatever date/time format we need)
* duration: Total time in system, in seconds (integer) - map to `seconds_of_help` (NOTE: Or maybe this should include queue time?)
* billsec: Total time call is up, in seconds (integer) - map to `seconds_of_help`
* disposition: What happened to the call: ANSWERED, NO ANSWER, BUSY, FAILED - map to TTM reason codes
* amaflags: What flags to use: see amaflags::DOCUMENTATION, BILL, IGNORE etc, specified on a per channel basis like accountcode.

With an extract of historical data in this format, plugged in to Agenses Forecast, we should be able to get a forecasting model to compare against and decide if we need all this custom stuff.
