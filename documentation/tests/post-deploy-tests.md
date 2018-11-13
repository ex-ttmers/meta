## Post Deployment Tests

#### What are they for?

In a world where deployments do not require human scrutiny, there is a need to know if a deployment broke any critical parts of our production environment.

A good example of this is interactions between the application we deployed, and other applications in our environment.  So lets say you deploy apangea, and with the deployment, comes a new kind of authentication with the reporting server.  We want to make sure that after we deploy, that a user will still be able to see their reports, and that we humans didn't forget to set a new enivornment variable with an authentication token AND restart the servers.

There are countless more ways that a deployment could break the production environment, so these tests are defining what connections we feel are critical to know aren't broken by the change.

#### What makes the so different from integration tests?

So some might say that these tests should just be integration tests, however the major difference here is that we want these tests to run against the actually deployed environment.  We aren't so much focusing on features and code here as we are focusing on the interactions and configurations of the actual production environment.

#### What kind of challenges does that bring?

Ok, so this brings on a challenge that we don't have with integration tests.  We are now running tests on 1 server, against a **REMOTE** host (ie. http://lms.thinkthroughmath.com).  This means that we don't have access to simply create or find data from the database, we are restricted to what we have in the web ui.  Some of you might asking, "why can't you connect the database?", well we probably could, but I see a lot of risk there that I don't want to undertake for these.

#### Solutions!

So we've added a rake task to create the basic set of users that would be run in the environment you want to create them in. see `create_pd_users` in `lib/tasks/test.rake`

#### Running Post deployment tests locally

1. create the basic set of users and classrooms if you have not yet already. You can set the user passwords to whatever you want locally, and ask Jeff if you want to know the passwords used on the farms.

        export TEACHER_PASSWORD=test
        export CONTENT_ADMIN_PASSWORD=test
        export STUDENT_PASSWORD=test
        rake create_pd_users

2. execute the tests

        rake test_post_deploy

#### Running Post deployment tests against a FARM

use http://jenkins.thinkthroughmath.com:8080/job/Post-deploy-tests/

If you want to run them locally against a farm, you'll need to get the passwords from Jeff, then you'll do the same steps above, but to execute the tests run `rake test_post_deploy[rc]` where rc can be substituted for any of the farm names.

#### JEFF WAS HIT BY A BUS! WHAT ARE THE PASSWORDS?

Just change the passwords for the test accounts that were created on each farm and update them in the jenkins job [HERE](http://jenkins.thinkthroughmath.com:8080/job/Post-deploy-tests/). NOTE that the teacher and live teacher account share `TEACHER_PASSWORD`
