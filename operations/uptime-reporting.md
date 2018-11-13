From time to time we get asked about historical uptime. Most typically this is for feel-good presentations and RFPs. 
New Relic's paid plans can generate an SLA report, but now that we're on the free tier we have to get a little creative. 

Conveniently, every week New Relic emails us with the previous week's uptime. [This spreadsheet](https://docs.google.com/spreadsheets/d/1eWPZlVneeg13qzQ2M5repCTgJb9LQiI_YsFHDMQv1wA/edit#gid=0) is used to track the values reported from New Relic. To automate this process, there is also a Zapier connection between email and the Google sheet to add a row automatically based on the emails we get. The email parser is set up at [Parser.zapier.com](http://parser.zapier.com) - credentials are in LastPass and the mailbox is `5toqytwi@robot.zapier.com`. There is a fake user in our New Relic Account named Zapp Brannigan with this email - password is in LastPass but its only reason to exist is to get the weekly emails from New Relic.

If you ever need to modify it, the [Zapier Zap](https://zapier.com) is set up under the `technology-managers@thinkthroughmath.com` account, and it's called `Throughout Metrics`.
