Just recording this in meta so someone can find it when they need it like I did:

Math time is stored in the warehouse as whole seconds. In apangea reports (such as student progress) we show it as Hours and Minutes (HH:MM), so we use [this method](https://github.com/thinkthroughmath/reporting/blob/rc/app/reports/report_helper.rb#L101) to reformat it before sending.
