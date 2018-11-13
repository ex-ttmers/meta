## Live teaching Deployments

The live teaching app is currently hosted on heroku without need a of a database, so it is pretty simple to deploy. Here are the steps you should take.

**Note: Deploying while live-teachers are logged on will cause there client to disconnect from ther server, they will need to refresh there browser.  This is why we only deploy when live teachers are offline**

## Mathbot way

1. Make sure live teachers aren't online

    mathbot teacher count

2. Promote the rc branch to master

    mathbot promote lt

3. Deploy code to servers and restart application

   mathbot deploy lt production master

## Manual way

You must have access to heroku and have keys setup. If you need them, see a lead.  

To setup your production and staging remotes, from your live teaching directory run...

    heroku git:remote -a ttm-live-teaching -r production
    heroku git:remote -a ttm-live-teaching-staging -r staging

1. Make sure that you are not going to interrupt live teachers.  There should be no live teachers online during the deployment, to check...

https://live-teaching.thinkthroughmath.com/data/stats/teachers

or

    mathbot teacher count

2. pull rc onto master

         git checkout master
         git pull
         git pull --no-ff --no-edit origin rc
         git push origin master

3. push master to the heroku repo

         git push production master:master

4. verify deployment by logging in as a student and a live teaching and connecting through the chat app
