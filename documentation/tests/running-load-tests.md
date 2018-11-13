## Load Tests

We have a suite of load tests set up to run on jenkins.

The source for the load tests is maintained in the [jmeter-scripts repo](https://github.com/thinkthroughmath/jmeter-scripts). there's information in the repo there about how to set up jmeter locally and how to create a new jmeter script if you need to.

As with any load test, the goal is to identify changes from baseline, not necessarily to stress test the system. Given that, the best way to use these is to run them against a known base, then add your change, run them again, and compare.

## Apangea Load Test

This test runs automagically against the RC farm every night at midnight Eastern, and posts the results to the Hipchat Notifications room. 

In case you want to run the apangea (main platform) load test by hand, use the [Jenkins job](http://jenkins.thinkthroughmath.com:8080/view/All/job/performance-test/). You'll need to set parameters using the link in the sidebar. Make sure you are testing against the correct farm, and that you have that farm reserved first. You may want to set other variables depending on what you are trying to test.

You can access results while it's running or afterward in [New Relic](https://rpm.newrelic.com/accounts/106461/applications/3305878)

## WWW Load Test
To run the WWW load test use the [Jenkins job] (http://jenkins.thinkthroughmath.com:8080/view/All/job/www_redesign_performance_test/). Click `build with parameters` in the sidebar. Make sure it's pointing against the dev server, not live.

You can access results while it's running or afterward in [New Relic](https://rpm.newrelic.com/), but you'll need to select the farm in the upper left dropdown that matches your load test after logging in.
