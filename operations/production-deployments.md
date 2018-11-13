# Production Deployments

The following is the "long way" and is somewhat out of date. These days, mathbot and jenkins do most of it for us-- go see the [Mathbot Jenkins Deployments](mathbot-jenkins-deployments.md#steps-to-do-a-full-system-deployment-through-mathbot) page.

Keep this around for use just in case those aren't working/available.

### To deploy to production without downtime:

1. Code passes tests on Jenkins
2. Dependencies listed in the [release notes](http://jenkins.thinkthroughmath.com:8080/job/release-notes-dashboard/) have been met
3. Tim approves of the changes
4. Newrelic reports throughput is under 15k rpm

### To deploy to production with downtime

0. The above steps for deployments without downtime are met
1. _For deployments that require migrations or down time we have to wait until live teaching hours are over (see below) and ensure there is minimal impact to **students** and **users**._
2. Downtime communications have been sent and expected downtime has been vetted _(for deployments requiring downtime; 0 downtime deployments do not require advance communication and vetting)._
See also: [What will require downtime](../documentation/what-requires-downtime.md).

### Downtime communications and vetting:

**At least 72 hours in advance of scheduled downtime**, Tim or Jim will send an email to the field (cc tier2@imaginelearning.com), providing the following details. Based on field activities, Tim or Jim will advise on  proceeding as planned or rescheduling to reduce impact to end users (i.e. students, teachers, admins, internal TTM groups such as ICs and Sales).
   * Anticipated start of deployment/downtime (in ET)
   * Anticipated length of deployment/downtime (in minutes or hours)
   * Nature of deployment/downtime (high-level description, such as bug fixes, feature releases, infrastructure upgrades, etc.)
   * Who will be affected during deployment/downtime (students, users, all users, live teachers, etc.)
   * What will users experience during deployment/downtime (maintenance page, application unavailable message, etc.)

### Live-teaching deployments will be handled separately. See [Live Teaching Deployments](./live-teaching-deployments.md).
### Game deployments will be handled separately. See [Game Deployments](./game-deployments.md).

### Continue with the below instructions, or use mathbot
[Mathbot Jenkins Deployments](mathbot-jenkins-deployments.md#steps-to-do-a-full-system-deployment-through-mathbot)

### 2. Maintenance Mode ON

Only needed if the deploy requires downtime. See also [What will require downtime](../documentation/what-requires-downtime.md).

    $ ttmscalr maintenance on -f [target]

### 3. Merge rc to master on all repos.  There are different ways to do this, this is how I do it

    git checkout master
    git pull
    git pull --no-ff --no-edit origin rc
    git push origin master

notes:
`--no-ff` forces a merge commit to be entered.  This allows us to revert a merge if need be.
`--no-edit` saves me from having to edit the merge commit, this will just accept the default merge message

### 4. Push the code to the repo

Now would be a good time to note down productions SHA before you push new code. This will be useful in the rare case that you might have to rollback to it.

But first make sure that your local master is updated:

    $ git pull origin master
    $ git push aws-[target] master:master

The `aws-[target]` above is a remote reference to one of the `aws-[target]` repositories in the `thinkthroughmath` GitHub organization. To add e.g. `aws-production`, before running the above commands, you would need to run:

    $ git remote add aws-production git@github.com:thinkthroughmath/aws-production.git

### 5. Run migrations
If there are migrations, run them now.  Note that the deploy:migrate command will first pull down the code from `aws-production/master`
run migrations:

    $ ttmscalr deploy:migrate [target]

### 6. Deploy the code to the farm

In the off chance that the `ttmscalr` gem has updated, you may want to update your gems:

    $ bundle install
    $ ttmscalr deploy [target]

### 7. Clear and prime the lesson player JSON cache

This should take ~5 min.

    $ ttmscalr ssh debug.1 -f production
    $ cd /var/www
    $ bundle exec rake lesson_player:clear_lesson_cache
    $ bundle exec rake lesson_player:prime_lesson_cache

### Sometimes there are extra manual steps to take

These are listed and managed here [[Manual tasks for the next deployment]]

### 8. Maintenance Mode OFF

If the app was put in maintenance mode

    $ ttmscalr maintenance off -f [target]

### 9. Do some math to verify the system is working!

## SOMETING WENT WRONG!
**Stay calm!** If you can figure out and fix what went wrong, do it. If you know something is wrong, and can't fix it, then you might just want to rollback.  You saved the previous SHA in earlier steps, so you just have to force push back to that SHA. I'll walk you through that now.

1. Checkout the leading SHA from before this deployment

    git checkout <SHA>

2. Create a branch from it, and switch to it

    git checkout -b rollback

3.  Force push our branch to the production repo

    git push -f aws-[target] rollback:master

4. Run the deployment command to have the farm pick up the code change

    ttmscalr deploy [target]

5. Check things out to see if everything is working.  If you are still having problems and have not called someone yet, now is a good time.
